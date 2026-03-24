# Deep Dive: Concurrency & Multi-User Scaling

> **Parent:** [Architecture](02-architecture.md) | **User Story:** US2.6

## The Core Problem

OpenClaw stores everything as **plain Markdown files**. When 10 users hit the same instance simultaneously, what prevents file write conflicts and data loss?

**Short answer:** OpenClaw is a **single-writer-per-session** system. It serializes writes within each session but does NOT fully protect shared workspace files across sessions. For 10+ concurrent users, the recommended pattern is **one instance per user**.

## The Lane Queue (Concurrency Model)

OpenClaw uses a **two-level lane-aware FIFO queue** (`src/process/command-queue.ts`):

```
Incoming Messages
       │
       ▼
┌─────────────────────────────────┐
│         GLOBAL LANE             │
│  maxConcurrent: 4 (main)       │
│  maxConcurrent: 8 (subagent)   │
│                                 │
│  ┌──────────┐ ┌──────────┐     │
│  │Session A │ │Session B │ ... │
│  │(serial)  │ │(serial)  │     │
│  └──────────┘ └──────────┘     │
└─────────────────────────────────┘
```

### Two Levels

| Level | Key | Behavior |
|-------|-----|----------|
| **Session Lane** | `session:<key>` | Only **one active run per session** at a time. Prevents race conditions on session files |
| **Global Lane** | `main` | Caps total parallelism across all sessions (default: 4 concurrent) |

**Key insight:** It does NOT serialize ALL requests globally. Different sessions run in parallel up to the global cap. Only same-session requests are serialized.

### Queue Modes

| Mode | Behavior |
|------|----------|
| `steer` | New input replaces queued input |
| `followup` | New input queued after current run |
| `collect` | New input batched into current run |
| `steer-backlog` | Replaces oldest queued item |

## The Race Condition Problem

### What's Protected

- **Session history** (`.jsonl` files) — protected by `updateSessionStore` promise-chain mutex
- **Per-session memory writes** — protected by lane queue serialization

### What's NOT Protected

- **Shared workspace files** (like `MEMORY.md`) — cross-session writes are NOT serialized
- **Known bug (Issue #29947):** Concurrent read/modify/write against the same shared file can lose one writer's update

```
Session A reads MEMORY.md  →  modifies  →  writes back
Session B reads MEMORY.md  →  modifies  →  writes back (overwrites A's changes!)
```

> *"Serialization is necessary, not sufficient. It keeps your session history sane. It doesn't automatically make your side-effects safe."*

## Solutions for 10 Concurrent Users

### Pattern 1: One Instance Per User (Recommended)

```
User 1 ──── Container 1 ──── Agent 1 (own workspace, own memory)
User 2 ──── Container 2 ──── Agent 2 (own workspace, own memory)
...
User 10 ── Container 10 ── Agent 10 (own workspace, own memory)
```

- **Complete isolation**: separate config, secrets, storage, network
- No file conflicts possible
- Each user gets their own MEMORY.md, daily logs, session history
- Managed via Docker Compose or Kubernetes

**This is the community consensus:** *"If you want to run agents for ten people, you need ten instances."*

### Pattern 2: Shared Gateway, Multiple Agents

```
Shared Gateway
  ├── Agent 1 (workspace-1) ←── User 1 via Telegram
  ├── Agent 2 (workspace-2) ←── User 2 via Discord
  ...
  └── Agent 10 (workspace-10) ←── User 10 via Slack
```

- One Gateway, many agents with **separate workspaces**
- Route users to agents via channel bindings
- Each agent has own `agentDir`, session store, memory files
- Shared Gateway process but isolated file writes
- **Never reuse `agentDir` across agents**

### Pattern 3: Shared Agent with Session Isolation

```
Shared Gateway ──── Single Agent
  ├── User 1 DM → session: agent:1:telegram:dm:user1
  ├── User 2 DM → session: agent:1:telegram:dm:user2
  ...
  └── User 10 DM → session: agent:1:telegram:dm:user10
```

- Set `dmScope: "per-channel-peer"` — each user gets own session key
- Sessions serialized independently (no cross-session blocking)
- **Caveat:** All users share MEMORY.md → race condition risk on writes
- **Caveat:** All users share same tool permissions
- Only viable for low-risk, read-heavy workloads

## Session Isolation Settings

| Setting | Session Key | Use Case |
|---------|------------|----------|
| `dmScope: "main"` (default) | `agent:<id>:<mainKey>` | Personal use. All DMs share one session |
| `dmScope: "per-channel-peer"` | `agent:<id>:<channel>:dm:<peerId>` | Multi-user. Each user gets isolated session |

## Kubernetes / Docker Scaling

### Docker Compose (Single Host)

```yaml
# docker-compose.yml — one container per user
services:
  user1:
    image: openclaw:latest
    volumes:
      - ./users/user1:/root/.openclaw
    ports:
      - "18789:18789"
  user2:
    image: openclaw:latest
    volumes:
      - ./users/user2:/root/.openclaw
    ports:
      - "18790:18789"
```

### Kubernetes (Multi-Host)

| Concern | Solution |
|---------|----------|
| **Storage** | ReadWriteMany (RWX) volumes: NFS, CephFS, Longhorn |
| **Cron duplication** | Designate one replica as cron leader |
| **TLS** | Caddy with automatic per-subdomain TLS |
| **Scaling** | One pod per user, or use community K8s operator |

**Community operator:** [openclaw-rocks/k8s-operator](https://github.com/openclaw-rocks/k8s-operator)

### Multi-Tenant Docker (Caddy)

Each user gets their own container, Caddy handles automatic TLS per subdomain:

```
user1.openclaw.example.com → Container 1
user2.openclaw.example.com → Container 2
```

## Database-Backed Alternatives (Future)

File-based memory is the main scaling bottleneck. Alternatives being explored:

| Alternative | Status | Benefit |
|-------------|--------|---------|
| **PostgreSQL + pgvector** | Feature request (Issues #1568, #15093) | Multi-instance shared memory, no file conflicts |
| **Hindsight** | Available | External API mode, multiple instances share memory server |
| **QMD** | Available | Local search engine combining BM25 + vector + LLM re-ranking |
| **openclaw-mem** | Community project | Documents DB concurrency patterns for OpenClaw |

PostgreSQL + pgvector is the most promising path for true multi-user shared memory.

## Reverse Proxy Patterns

| Proxy | Best For |
|-------|----------|
| **Nginx** | TLS termination, load balancing, WebSocket support. Multiple instances behind one IP |
| **Caddy** | Multi-tenant with automatic per-subdomain TLS |
| **Pomerium / oauth2-proxy** | Identity-aware proxy, pass user identity via headers |
| **Traefik** | Kubernetes-native routing |

OpenClaw supports **Trusted Proxy Auth** — runs behind auth proxies with `allowUsers` access control.

**Common pitfall:** Pairing loops from trust-boundary mismatch between gateway, proxy, and browser origin.

## Known File Issues

| Issue | Description | Impact |
|-------|-------------|--------|
| **#29947** | Concurrent shared-workspace read/modify/write loses updates | Data loss |
| **#5430** | Terminated tool calls corrupt session `.jsonl` files | All subsequent API requests fail |
| **#31322** | Group chat sessions reset daily at 4 AM UTC after upgrade | History wiped |
| **#52786** | Stale-cache race in `sessions.patch modelOverride` | Config lost |
| **SQLite BUSY** | Multiple processes accessing memory index | Search failures |

## Decision Matrix

| Users | Complexity | Pattern | Notes |
|-------|------------|---------|-------|
| 1 | Low | Default setup | Personal assistant, single agent |
| 2-5 (trusted) | Medium | Multi-agent on shared Gateway | Route by channel binding, separate workspaces |
| 5-10 | Medium-High | One container per user | Docker Compose, shared host |
| 10-50 | High | Kubernetes with K8s operator | One pod per user, RWX storage |
| 50+ | Very High | Multiple hosts + load balancer | Caddy/Nginx, per-subdomain routing |

## Bottom Line

1. **OpenClaw is single-writer-per-session**, not single-writer-per-instance
2. Shared workspace files (MEMORY.md) have **known race conditions** across sessions
3. For 10 concurrent users: **one instance per user** via Docker/K8s
4. Database-backed memory (PostgreSQL + pgvector) will eventually solve the file-based scaling limit
5. Use `dmScope: "per-channel-peer"` if you must share one agent across users

## Key Docs & Sources

- [Command Queue](https://docs.openclaw.ai/concepts/queue)
- [Session Management](https://docs.openclaw.ai/concepts/session)
- [Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)
- [Trusted Proxy Auth](https://docs.openclaw.ai/gateway/trusted-proxy-auth)
- [Architecture Part 2: Concurrency](https://theagentstack.substack.com/p/openclaw-architecture-part-2-concurrency)
- [Race Condition Issue #29947](https://github.com/openclaw/openclaw/issues/29947)
- [High Availability Guide](https://lumadock.com/tutorials/openclaw-high-availability-clustering)
- [Multi-Tenant Docker Guide](https://clawtank.dev/blog/openclaw-multi-tenant-docker-guide)
- [K8s Operator](https://github.com/openclaw-rocks/k8s-operator)
