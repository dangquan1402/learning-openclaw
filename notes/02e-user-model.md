# Deep Dive: Single-User vs Multi-User Architecture

> **Parent:** [Architecture](02-architecture.md) | **User Story:** US2.5

## TL;DR

**OpenClaw is fundamentally a personal AI assistant.** Its tagline is "Your own personal AI assistant." The community has stretched it into multi-user territory, but its security model assumes **one trusted operator** per gateway.

> *"OpenClaw is not tuned for the multi-user use case. It's a personal agent. Folks are making it be multi-user, but it has pinch points."* — Community

## Design Philosophy

- **Workspace-first**: the agent's workspace directory is the single source of truth
- One Gateway → one operator → one or more agents → one or more channels
- Designed for a single person to run their own AI assistant across all their messaging apps
- Multi-user support exists but is **not the primary design goal**

## How Multiple Users Can Interact

A single Gateway can serve multiple channels simultaneously:

```
                    ┌─ Telegram (personal)
                    ├─ Discord (community server)
User/Owner ──── Gateway ──── Agent
                    ├─ Slack (work team)
                    └─ WhatsApp (family group)
```

In group settings (Telegram groups, Discord servers, Slack workspaces):
- Bot participates as a member
- Typically gated by **mention** (only responds when @mentioned)
- Can be restricted to specific channels via allowlists

## Access Control: The 4 DM Policies

| Policy | How It Works | Risk Level |
|--------|-------------|------------|
| **Pairing** (default) | Unknown users get 6-digit code (expires 1hr). Owner must run `openclaw pairing approve` | Low |
| **Allowlist** | Only pre-approved user IDs can DM | Low |
| **Open** | Anyone can DM the bot | **High** — dangerous for tool-enabled agents |
| **Disabled** | DMs entirely blocked | None |

**Important:** Group access does NOT grant DM access (and vice versa). A user chatting in a public Discord channel still needs separate pairing for private DMs.

### Device Pairing (Apps)

macOS/iOS/Android apps connecting to the Gateway use a separate challenge-response mechanism with per-device tokens and cryptographic signatures.

## Memory Isolation: Group vs DM

This is the key architectural boundary:

### Session Keys

| Context | Session Key | Memory Access |
|---------|------------|---------------|
| **DM (main)** | `agent:<agentId>:<mainKey>` | Full: MEMORY.md, USER.md, personal notes, all tools |
| **Group** | `agent:<agentId>:<channel>:group:<id>` | Restricted: no personal memory files loaded |

### Critical Rules

- **MEMORY.md is ONLY loaded in private/main sessions** — never in group chats
- This prevents personal details from leaking to strangers in group contexts
- Each group maintains its own isolated conversation history
- Groups are isolated from other groups and from DMs

### Known Limitation

If multiple users DM the same agent, their DMs may share the same "main" session by default (depending on `dmScope` settings). For true per-user isolation in DMs, you need **one agent per person**.

## Main Session vs Group Sessions

| Aspect | Main Session (DMs) | Group Sessions |
|--------|-------------------|----------------|
| **Purpose** | Personal "home base" | Shared context |
| **Memory** | Full access (MEMORY.md, USER.md, notes) | No personal memory |
| **Tools** | Full tool access | Potentially restricted |
| **Persistence** | Single continuous session | Isolated per group |
| **Visibility** | Only the owner | All group members |

## Multi-Agent Routing for Multiple Users

For true multi-user setups, run **multiple agents on one Gateway**:

```
Gateway
  ├── Agent A (workspace-a) ←── Slack messages from Team A
  ├── Agent B (workspace-b) ←── Telegram messages from User B
  └── Agent C (workspace-c) ←── Discord server for Community
```

Each agent has:
- Its own workspace (SOUL.md, AGENTS.md, persona rules)
- Its own state directory (`~/.openclaw/agents/<agentId>/`)
- Its own session store
- **No cross-talk** unless explicitly enabled

**Routing via bindings:** Map `(channel, accountId, peer)` → `agentId`. Can route different senders to different agents.

> **Rule:** Never reuse agent directories across agents — causes auth/session collisions.

## Log Reading: Who Can See What

| What | Where | Who Can Read |
|------|-------|-------------|
| Session transcripts | `~/.openclaw/agents/<agentId>/sessions/*.jsonl` | Anyone with filesystem access |
| Memory files | `~/.openclaw/workspace/` | Anyone with filesystem access |
| Gateway metadata | Control UI / API | Anyone with gateway token |

### Privacy Implications

- The **trust boundary is disk access** — lock down `~/.openclaw` permissions
- Operators with control-plane access can inspect session history by design
- Self-hosted + encrypted channels (Signal) = no third party reads messages
- But the **server operator always can** read stored logs
- Conversation logging is configurable; logs can be auto-deleted after N days

## Sandbox Model for Public-Facing Agents

| Setting | Behavior |
|---------|----------|
| `sandbox.mode: "off"` (default) | No sandboxing — agent has full tool access |
| `sandbox.mode: "non-main"` | Sandbox everything except owner's main session |
| `sandbox.mode: "all"` | Sandbox all sessions including owner's |

### For Public-Facing Agents

- `tools.allow` should be an explicit allowlist (no wildcard)
- Remove `exec` tool if not needed
- Docker containers run with `network: 'none'` by default
- **A shared agent for mutually untrusted users is NOT a supported security boundary**

### Security Philosophy

> "Identity first (who can talk), Scope next (where the bot can act), Model last (assume the model can be manipulated; limit blast radius)."

## Enterprise/Team Deployment Patterns

### Pattern 1: Multiple Independent Instances

```
Instance A (frontend team) ──── Gateway A ──── Agent A
Instance B (backend team)  ──── Gateway B ──── Agent B
Instance C (data team)     ──── Gateway C ──── Agent C
```

Fully isolated, no interference. Higher cost.

### Pattern 2: Single Multi-Tenant Instance

```
Shared Gateway
  ├── Agent: frontend-team
  ├── Agent: backend-team
  └── Agent: data-team
```

Saves costs but requires careful configuration. Not a security boundary for adversarial users.

### Enterprise Features in Progress

- Microsoft Teams integration with granular permissions (channel-specific, user-group-specific)
- RBAC (role-based access control) — [Feature request #8081](https://github.com/openclaw/openclaw/issues/8081)
- Kubernetes-based multi-user scaling guides

## Known Security Limitations

1. **Multiple users messaging one tool-enabled agent all share the same permission set** — no per-user tool restrictions
2. A [multi-user session isolation vulnerability](https://www.penligent.ai/hackinglabs/openclaw-multi-user-session-isolation-failure-authorization-bypass-and-privilege-escalation/) documented authorization bypass and privilege escalation
3. The security model assumes **one trusted operator per gateway**
4. For adversarial multi-user scenarios: use **separate gateways/hosts per trust boundary**

## Bottom Line

| Scenario | Recommended Approach |
|----------|---------------------|
| Personal use | Single agent, single Gateway. Default setup. |
| Family/friends | One agent, pairing for DM access. Groups for shared context. |
| Small team (trusted) | Multi-agent on one Gateway. Route by channel/user. |
| Public-facing bot | Sandbox mode "all", strict tool allowlist, no personal memory. |
| Enterprise (untrusted users) | Separate Gateways per trust boundary. Never share agent dirs. |

## Key Docs & Sources

- [Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)
- [Session Management](https://docs.openclaw.ai/concepts/session)
- [Security](https://docs.openclaw.ai/gateway/security)
- [DM Policy Guide](https://zenvanriel.com/ai-engineer-blog/openclaw-dm-policy-access-control-guide/)
- [Data Privacy Guide](https://www.clawctl.com/blog/openclaw-data-privacy-guide)
- [Running OpenClaw Safely - Microsoft](https://www.microsoft.com/en-us/security/blog/2026/02/19/running-openclaw-safely-identity-isolation-runtime-risk/)
- [Session Isolation Vulnerability](https://www.penligent.ai/hackinglabs/openclaw-multi-user-session-isolation-failure-authorization-bypass-and-privilege-escalation/)
- [RBAC Feature Request #8081](https://github.com/openclaw/openclaw/issues/8081)
