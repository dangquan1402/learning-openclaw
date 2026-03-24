# Advanced Topics

## Plugin Development

Plugins are **TypeScript modules** that extend the Gateway runtime. Unlike skills (text-based), plugins are code.

### Plugin Loading (4 Phases)

1. **Discovery** -- scan candidate locations
2. **Manifest parsing** -- read `openclaw.plugin.json`
3. **Dynamic loading** -- jiti loads `.ts`/`.js` (no build step needed)
4. **Registration** -- call `register()` function

### Plugin Locations (scanned in order)

```
<workspace>/.openclaw/extensions/*.ts
<workspace>/.openclaw/extensions/*/index.ts
~/.openclaw/extensions/*.ts
~/.openclaw/extensions/*/index.ts
```

### What Plugins Can Register

- Gateway RPC methods
- Gateway HTTP handlers
- Agent tools
- CLI commands
- Background services
- Config validation
- Skills (plugins can bundle skills)
- Auto-reply commands (execute without invoking AI)

### Installation

```bash
openclaw plugins install <npm-spec>
```

Uses npm pack, extracts into `~/.openclaw/extensions/<id>/`, enables in config.

> **Warning:** Plugins run in-process with the Gateway -- treat them as trusted code.

## Multi-Workspace and Multi-Agent

Each agent has three isolated components:

| Component | Location | Purpose |
|-----------|----------|---------|
| **Workspace** | Configurable | Files, persona rules, local notes |
| **Agent Directory** | `~/.openclaw/agents/<agentId>/` | Auth profiles, model registry, per-agent config |
| **Session Store** | `~/.openclaw/agents/<agentId>/sessions` | Chat history + routing state |

### Critical Rules

- **Never reuse agentDir across agents** (causes auth/session collisions)
- Auth profiles are per-agent (each reads from its own `auth-profiles.json`)
- Skills can be per-agent (workspace `skills/`) or shared (`~/.openclaw/skills`)

### Routing Bindings

Map `(channel, accountId, peer)` + optional guild/team IDs to an `agentId`. One server can host multiple accounts without mixing sessions.

### Setup

```bash
openclaw agent create   # agent wizard scaffolds workspace + agent directory
```

## Deployment Options

| Option | Best For | Details |
|--------|----------|---------|
| **Local** | Development, personal use | Run directly on your machine |
| **Docker** | Isolation, reproducibility | `docker-setup.sh` with Docker Compose |
| **AWS Lightsail** | Easy cloud | One-click blueprint (March 2026). Pre-configured with Bedrock |
| **Cloudflare Workers** | Serverless | Via Moltworker. 1/2 vCPU, 4 GiB RAM, 8 GB disk |
| **DigitalOcean** | Managed cloud | App Platform with elastic scaling |
| **Self-Hosted VPS** | Full control | Gateway on VPS; connect via Control UI, Tailscale, or SSH |

## Security Best Practices

### API Key Management

- Never store keys in plain text config or commit to version control
- Use encrypted stores or secret managers
- Set restrictive file permissions: `chmod 600` on credential files, `chmod 700` on directories
- Environment variables do NOT inherit into sandbox

### Sandboxing

- **Off by default** -- must be explicitly enabled
- Three permission gates:
  1. `agents.list[].tools.allow/deny` -- agent-level tool permissions
  2. `tools.sandbox.tools.allow` -- sandbox-level tool filter
  3. `sandbox.docker.network` -- network access for container (disabled by default)
- **All layers must be configured** for proper security
- Recommended: `sandbox.mode: "non-main"` or `"all"` for any public-facing agent

### Permissions

- By default, OpenClaw can execute **any shell command** with no allowlist
- For production: set explicit `tools.allow` lists
- If an agent doesn't need `exec`, remove it
- For public-facing agents: sandboxing should always be on

## The OpenClaw Ecosystem

| Project | Description |
|---------|-------------|
| **OpenClaw** | Core project. Multi-channel AI agent gateway |
| **NemoClaw** (NVIDIA) | Runs OpenClaw inside NVIDIA OpenShell for security |
| **IronClaw** (NEAR AI) | Rust reimplementation. Tools in WASM containers, strict access |
| **Moltworker** (Cloudflare) | Runs OpenClaw on Cloudflare Workers. Centralized key management |
| **Gitclaw** | OpenClaw running entirely on GitHub Actions |
| **Nanobot** | OpenClaw concepts in ~4,000 lines of Python |
| **PicoClaw** | Ultra-lightweight Go agent, under 10 MB RAM |
| **MetaClaw** | Agent that learns and evolves through conversation |

## Building an End-to-End Project

Suggested project: **Personal Assistant Bot**

1. Set up OpenClaw locally with Ollama (free) or Anthropic
2. Connect to Telegram
3. Create custom skills for your workflow:
   - Calendar integration
   - Note-taking
   - Task management
   - Web search
4. Configure Heartbeat for daily briefings
5. Add memory for persistent context
6. Deploy to Docker or cloud for always-on availability
