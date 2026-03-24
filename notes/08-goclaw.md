# GoClaw — OpenClaw Rebuilt in Go

> **User Story:** US6 | **Repo:** [nextlevelbuilder/goclaw](https://github.com/nextlevelbuilder/goclaw) | **Site:** [goclaw.sh](https://goclaw.sh/en/)

## What is GoClaw?

A **ground-up rewrite of OpenClaw in Go** — multi-tenant AI agent platform that deploys as a single static binary (~25 MB).

> *"OpenClaw rebuilt in Go — with multi-tenant isolation, 5-layer security, and native concurrency."*

- **License:** MIT
- **Created:** 2026-02-22 (~1 month old)
- **Stars:** 1,085 | **Forks:** 312 | **Contributors:** 30
- **Latest:** v2.8.1 (active daily development)
- **Core team:** Vietnamese developer community (NextLevelBuilder)

## OpenClaw vs GoClaw at a Glance

| Metric | OpenClaw (TypeScript) | GoClaw (Go) |
|--------|-----------------------|-------------|
| **Binary size** | 28 MB + Node.js runtime | ~25 MB standalone |
| **RAM (idle)** | >1 GB | ~35 MB |
| **Startup** | >5 seconds | <1 second |
| **Target hardware** | $599+ Mac Mini | $5 VPS |
| **Runtime deps** | Node.js, npm | Zero (static binary) |
| **Multi-tenant** | Not native (community hacks) | Built-in PostgreSQL |
| **Concurrency** | Node.js event loop + Lane Queue | Native goroutines + lane scheduler |
| **Storage** | Markdown files + SQLite | PostgreSQL 18 + pgvector |
| **Security layers** | Basic (pairing, sandbox) | 5-layer (gateway → global → agent → channel → owner) |

## 5-Layer Architecture

```
1. CHANNELS        Telegram, Discord, Slack, Feishu, Zalo, WhatsApp, WebSocket
       │
2. API GATEWAY     WebSocket RPC (v3 protocol) + HTTP REST (OpenAI-compatible)
       │
3. SECURITY        Rate limiting, prompt injection detection, SSRF protection,
                   shell deny patterns, AES-256-GCM encryption
       │
4. AGENT ENGINE    Agent loop (think → act → observe), teams, delegation, MCP bridge
       │
5. DATA LAYER      PostgreSQL 18 + pgvector, optional Redis, audit logs
```

## Key Components

| Package | Purpose |
|---------|---------|
| `internal/agent/` | Agent loop, router, resolver, input guard |
| `internal/channels/` | Telegram, Feishu, Zalo, Discord, WhatsApp adapters |
| `internal/providers/` | 20+ LLM providers (Anthropic, OpenAI, DashScope, Claude CLI, etc.) |
| `internal/tools/` | 40+ built-in tools (filesystem, exec, web, memory, subagent, MCP) |
| `internal/gateway/` | WebSocket + HTTP server, RPC router |
| `internal/store/` | Store interfaces + PostgreSQL implementations (raw SQL, no ORM) |
| `internal/mcp/` | Model Context Protocol bridge (stdio/SSE/streamable-http) |
| `internal/knowledgegraph/` | LLM-extracted knowledge graph with traversal |
| `internal/skills/` | BM25 + pgvector hybrid search for skill discovery |
| `internal/cron/` | Scheduling with `at`, `every`, and cron expressions |
| `internal/sandbox/` | Docker-based isolated code execution |
| `internal/crypto/` | AES-256-GCM encryption for API keys |
| `internal/permissions/` | RBAC (admin/operator/viewer) |
| `internal/tracing/` | LLM call tracing + optional OpenTelemetry OTLP export |
| `internal/tts/` | Text-to-Speech (OpenAI, ElevenLabs, Edge, MiniMax) |
| `pkg/browser/` | Browser automation via go-rod + CDP |
| `ui/web/` | React 19 SPA (Vite 6, Tailwind CSS 4, Radix UI, Zustand) |

## Unique Features (Not in OpenClaw)

1. **Multi-tenant PostgreSQL** — per-user workspaces, isolated sessions, encrypted API keys
2. **5-layer permission system** — gateway → global tool policy → per-agent → per-channel → owner-only
3. **Agent teams with task boards** — create, claim, complete tasks with `blocked_by` dependencies; team mailbox for peer/broadcast messaging
4. **Sync and async delegation** — Agent A waits for Agent B (sync) or fire-and-forget (async)
5. **Lane-based scheduler** — separate lanes for main/subagent/team/cron (cron can't starve interactive)
6. **Knowledge graph** — LLM extraction + traversal, stored in PostgreSQL
7. **Built-in observability** — LLM call tracing with spans, prompt cache metrics, optional Jaeger export
8. **Extended thinking modes** — per-provider config (Anthropic budget tokens, OpenAI reasoning effort)
9. **20+ LLM providers** — includes DashScope (Alibaba), Bailian, MiniMax, Cohere, Perplexity
10. **7 messaging channels** — includes Zalo OA, Zalo Personal, Feishu/Lark (not in OpenClaw)
11. **Docker sandbox** — isolated untrusted code execution
12. **Web dashboard** — React 19 SPA, mobile-optimized

## Why Go?

### Advantages

- **Native concurrency**: goroutines handle thousands of simultaneous agents (no event loop bottleneck)
- **Single binary**: no `npm install`, no `node_modules`, no runtime deps
- **30x less RAM**: ~35 MB vs >1 GB idle
- **5x faster startup**: <1s vs >5s
- **Type safety**: Go's type system makes per-tenant data separation more straightforward
- **Multi-tenant by design**: PostgreSQL solves the Markdown file concurrency problem

### Trade-offs

- Smaller AI/ML ecosystem than TypeScript/Python
- Less flexible for rapid prototyping
- Web dashboard still requires TypeScript/React
- OpenClaw's larger community = more plugins/extensions
- Fewer third-party skills carry over directly

## How It Solves the 10-User Problem

GoClaw's architecture **directly addresses OpenClaw's multi-user weakness**:

| OpenClaw Problem | GoClaw Solution |
|------------------|-----------------|
| Markdown file race conditions | PostgreSQL with proper transactions |
| No native multi-tenant | Built-in per-user workspace isolation |
| Lane Queue per-session only | Lane-based scheduler with global concurrency control |
| Shared MEMORY.md conflicts | Per-user memory stored in PostgreSQL rows |
| Single operator security model | 5-layer RBAC (admin/operator/viewer) |

## Installation

### From Source

```bash
git clone https://github.com/nextlevelbuilder/goclaw.git && cd goclaw
make build
./goclaw onboard    # Interactive setup
source .env.local && ./goclaw
```

### Docker (Recommended)

```bash
git clone https://github.com/nextlevelbuilder/goclaw.git && cd goclaw
chmod +x prepare-env.sh && ./prepare-env.sh
make up
# Dashboard: http://localhost:3000
# Health: curl http://localhost:18790/health
```

### Optional Services

| Flag | Feature |
|------|---------|
| `WITH_BROWSER=1` | Headless Chrome for web scraping |
| `WITH_OTEL=1` | Jaeger for OpenTelemetry tracing |
| `WITH_SANDBOX=1` | Docker sandbox for code execution |
| `WITH_TAILSCALE=1` | Private network exposure |
| `WITH_REDIS=1` | Caching layer |

**Prerequisites:** Go 1.26+, PostgreSQL 18 + pgvector, Docker (optional)

## Design Patterns

- **Interface-based stores** (`store.SessionStore`, `store.AgentStore`) — PostgreSQL implementations with raw SQL, no ORM
- **Context propagation** — Go's `context.Context` carries user ID, agent ID, agent type, locale
- **Build-tag gated features** — OpenTelemetry is opt-in via build tags (keeps base binary small)
- **Agent types:** "open" (per-user context, 7 files) vs "predefined" (shared context + per-user USER.md)

## The Broader "Claw Ecosystem"

| Project | Language | Binary | RAM | Focus |
|---------|----------|--------|-----|-------|
| **OpenClaw** | TypeScript | 28 MB + Node | >1 GB | Reference implementation, largest community |
| **GoClaw** | Go | ~25 MB | ~35 MB | Multi-tenant, PostgreSQL, performance |
| **ZeroClaw** | Rust | 3.4 MB | <5 MB | Minimal footprint, edge devices |
| **PicoClaw** | Go | ~8 MB | <10 MB | Ultra-lightweight, edge-targeted |
| **IronClaw** | Rust | — | — | Security-focused, WASM sandboxed tools |

## Limitations

1. **Very young** — 1 month old, v2.8.1 already (expect breaking changes)
2. **Smaller community** — 1K stars vs OpenClaw's 250K+
3. **PostgreSQL required** — more operational complexity than OpenClaw's file-based approach
4. **Limited public discussion** — no Reddit/HN presence yet
5. **i18n limited** — English, Vietnamese, Chinese only
6. **Rapid version churn** — v0 to v2.8.1 in one month suggests API instability
7. **Vietnamese-centric core team** — may affect English docs/support
8. **No native mobile apps** — web dashboard only (mobile-optimized)

## Key Docs & Sources

- [GoClaw GitHub](https://github.com/nextlevelbuilder/goclaw)
- [GoClaw Docs](https://docs.goclaw.sh)
- [GoClaw Official Site](https://goclaw.sh/en/)
- [GoClaw API Reference](https://github.com/nextlevelbuilder/goclaw/blob/main/api-reference.md)
- [OpenClaw Alternatives Comparison 2026](https://www.elegantsoftwaresolutions.com/blog/openclaw-alternatives-comparison-guide-2026)
