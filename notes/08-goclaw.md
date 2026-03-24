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

## User Management (Per-User, Not All-in-One)

GoClaw is **definitively per-user**, unlike OpenClaw's single-operator model.

### How Users Work

- **No signup page** — users identified by `user_id` string from messaging channel (Telegram ID, Discord ID, etc.) or `X-GoClaw-User-Id` HTTP header
- **First interaction auto-creates** a user profile (`user_agent_profiles` table)
- **Admins add users to tenants** via API: `POST /v1/tenants/{id}/users`
- **Web dashboard** has full tenant admin section for user management

### 5 User Roles

| Role | Access |
|------|--------|
| `owner` | Full control, cross-tenant access (set via `GOCLAW_OWNER_IDS` env) |
| `admin` | Full access within tenant (agents, config, API keys) |
| `operator` | Read + write (chat, sessions, agent management) |
| `member` | Standard user access |
| `viewer` | Read-only |

### Authentication (Cascading Priority)

1. **Gateway Token** — shared bearer token, grants admin role (dev/single-admin)
2. **API Keys** — format `goclaw_` + 32 hex chars, SHA-256 hashed at rest, tenant-scoped
3. **Browser Pairing** — `X-GoClaw-Sender-Id` header, grants operator role
4. **No auth** — falls back to operator (dev mode only)
5. **None** → 401 Unauthorized

**No OAuth for end users.** Authentication is stateless (bearer token or API key per request).

### Per-User Isolation

- Every DB query includes `WHERE tenant_id = $N`
- Each user gets their own: memory, knowledge graph, context files (USER.md), session history
- API keys bound to specific owner (`owner_id` overrides any header)
- Agent sharing is explicit via `agent_shares` table with per-share roles

## Memory Management (PostgreSQL-Based)

### Per-User by Default

- Memory queries filter by `agent_id + user_id + tenant_id`
- Opt-in shared mode: queries filter by `agent_id + tenant_id` only (org-wide memory)
- **Users cannot see each other's memory**

### Schema (vs OpenClaw's Markdown Files)

**OpenClaw:** `MEMORY.md` + `memory/YYYY-MM-DD.md` files on disk

**GoClaw:** Two PostgreSQL tables:

```sql
-- Documents (equivalent of memory files)
memory_documents:
  agent_id UUID, user_id VARCHAR(255), tenant_id UUID
  path TEXT, content TEXT, hash VARCHAR(64)
  UNIQUE(agent_id, COALESCE(user_id, ''), path)

-- Chunks (for semantic search)
memory_chunks:
  document_id (FK), agent_id, user_id, tenant_id
  text content, start_line, end_line
  embedding vector(1536)    -- pgvector
  tsv tsvector               -- full-text search
```

### Hybrid Search (3-Phase)

```
Query → [FTS Phase] + [Vector Phase] → Hybrid Merge → Results
         (30% weight)   (70% weight)    (deduplicate + boost personal 1.2x)
```

1. **FTS Phase**: PostgreSQL `plainto_tsquery` + `ts_rank()` scoring
2. **Vector Phase**: pgvector `<=>` cosine distance, embedding via configured provider
3. **Hybrid Merge**: Dedup by `(Path, StartLine)`, weighted merge, **personal chunks get 1.2x boost**

**Fallback**: If FTS returns nothing (non-English), falls back to `ILIKE` keyword search.

**Defaults**: max chunk 1000 bytes, max 6 results per search.

### Daily Logs (Memory Flush)

Same concept as OpenClaw but database-backed:

- Writes to paths like `memory/2026-03-24.md` in `memory_documents` table
- **Trigger**: Before session compaction when token count exceeds threshold
- **Process**: Last 10 messages → LLM at temperature 0.3 → writes via `memStore.PutDocument()`
- **Fallback**: If LLM produces no tool calls, regex extraction on last 20 messages → writes to `memory/YYYY-MM-DD-auto-extract.md`
- Documents are appended (not overwritten) and immediately indexed

### Knowledge Graph (Extra Layer)

GoClaw adds a knowledge graph OpenClaw doesn't have:

```sql
kg_entities: name, type, description, properties JSONB, confidence, embedding
kg_relations: source_id → target_id, relation_type, properties, confidence
```

- LLM-driven extraction builds the graph from conversations
- Hybrid search: ILIKE (0.3) + vector (0.7)
- Per-user scoped with shared mode available

## Multi-Tenant Architecture

### Column-Based Isolation (Not RLS, Not Schema-per-Tenant)

- **Every table** has `tenant_id UUID NOT NULL` column (30+ tables)
- **Every SQL query** includes `WHERE tenant_id = $N` — fail-closed
- Tenant resolved **from key binding**, not client headers (prevents impersonation)
- Unique constraints are tenant-scoped: `UNIQUE(tenant_id, agent_key)`

### Tenant Resolution

| Source | Resolution |
|--------|-----------|
| API keys | `tenant_id` field on the key |
| Gateway tokens | Owner membership or master tenant |
| Chat channels | Baked into `channel_instances` config |
| System keys | `X-GoClaw-Tenant-Id` header |

### Session Isolation

- WebSocket sessions: tenant scope persists for session lifetime
- Event filtering: three-mode server-side filter prevents cross-tenant leakage
- Access revocation: removing user triggers `EventTenantAccessRevoked` → immediate disconnect

### Security Summary

| Layer | Protection |
|-------|-----------|
| API keys | SHA-256 hashed at rest, 5-min TTL cache with pubsub invalidation |
| LLM keys | AES-256-GCM encrypted (format: `aes-gcm:` + base64) |
| File URLs | HMAC-signed tokens (`?ft=` parameter) |
| Cross-tenant | Only `GOCLAW_OWNER_IDS` can access other tenants |

## OpenClaw vs GoClaw: User & Memory Comparison

| Aspect | OpenClaw | GoClaw |
|--------|----------|--------|
| **User model** | Single operator (personal assistant) | Multi-tenant, per-user isolation |
| **User roles** | None (owner only) | 5 roles (owner/admin/operator/member/viewer) |
| **Auth** | Gateway token + pairing | Cascading: token → API key → pairing |
| **Memory storage** | Markdown files on disk | PostgreSQL + pgvector |
| **Memory isolation** | MEMORY.md shared, group sessions restricted | Per-user by default, opt-in sharing |
| **Memory search** | BM25 + optional vector (SQLite) | Hybrid FTS + vector (PostgreSQL) |
| **Knowledge graph** | None | Built-in (entities + relations) |
| **Concurrent writes** | Race condition on shared files | PostgreSQL transactions |
| **Daily logs** | `memory/YYYY-MM-DD.md` files | Same paths, stored in DB |

## Browser Control (Two Paths)

GoClaw supports browser automation via two approaches:

| Path | Technology | Pros | Cons |
|------|-----------|------|------|
| **Native go-rod** (built-in) | Chrome DevTools Protocol, `pkg/browser/` | Zero overhead, same process, multi-tenant isolation, 11 actions | Chrome only |
| **Playwright MCP** (external) | Connect via `mcp_servers` config | Cross-browser (Chrome/Firefox/WebKit), richer selectors | Extra Node.js process, ~100MB overhead |

### Native Browser Actions

`start`, `stop`, `status`, `tabs`, `open`, `close`, `navigate`, `snapshot`, `screenshot`, `console`, `act` (click/type/press/hover/wait/evaluate)

### Playwright MCP Config

```json
"mcp_servers": {
  "playwright": {
    "transport": "stdio",
    "command": "npx",
    "args": ["@anthropic/mcp-playwright"],
    "tool_prefix": "pw"
  }
}
```

Tools auto-register as `mcp_pw__browser_navigate`, `mcp_pw__browser_click`, etc. No code changes needed.

## Deployment Purpose: Personal vs Enterprise

### Two Fundamentally Different Products

| | Personal (OpenClaw) | Enterprise (GoClaw) |
|---|---|---|
| **Who controls it** | You own & run it | Company owns & runs it |
| **Account access** | You give the bot YOUR credentials | Company pre-configures service accounts |
| **Users** | Just you | Employees assigned by admin |
| **Trust model** | You trust yourself | Company trusts platform, employees trust company |
| **Social media** | Bot posts AS YOU | Bot posts to COMPANY accounts |
| **Browser auth** | You log in, bot reuses your session | Service accounts managed by IT |
| **Credentials** | You paste API keys in config | IT admin manages, users never see keys |

### How Users "Give Access" to Accounts

**Personal (OpenClaw):**
- Paste API keys in config/env vars
- Log into Chrome, bot reuses cookies (Extension Relay mode)
- Authorize OAuth apps (like x-poster) that act on your behalf
- You are the only user — you trust the bot with everything

**Enterprise (GoClaw):**
- Users DON'T give personal credentials
- IT admin configures service accounts (company Twitter, Facebook)
- Users interact via chat → bot uses company credentials
- RBAC controls who can trigger what actions
- Per-user audit logs track all actions

### The Missing Piece: User-Facing OAuth Consent

Neither OpenClaw nor GoClaw natively handles the case where **end users grant access to their own accounts** (e.g., "Connect your Twitter"). To build this, you'd need:

```
1. User clicks "Connect Twitter" in web dashboard
       │
2. Redirected to Twitter OAuth → user authorizes your app
       │
3. Token stored (encrypted) per-user in database
       │
4. Bot uses that user's token when posting on their behalf
```

**GoClaw is much closer** to supporting this because it has:
- Per-user database storage (`tenant_id + user_id` scoping)
- AES-256-GCM encryption for credentials
- API key management infrastructure
- Web dashboard (React 19) to build the UI on

**What you'd need to build:**
- OAuth consumer endpoints (redirect URI handler, token exchange)
- Per-user credential storage table (or extend existing `user_agent_profiles`)
- UI flow in the web dashboard ("Connect your accounts" page)
- Token refresh logic (most OAuth tokens expire)
- Skill that reads per-user tokens and posts on their behalf

This is essentially building a **SaaS layer** on top of GoClaw — turning it from "company tool" into "user-facing product."

### Decision Framework

| Use Case | Choose | Why |
|----------|--------|-----|
| Personal assistant for yourself | OpenClaw | Simpler, larger ecosystem, your own keys |
| Internal team tool | GoClaw | Multi-tenant, RBAC, per-user isolation |
| SaaS product (users connect own accounts) | GoClaw + custom OAuth | Closest architecture, needs OAuth layer built |
| Public-facing bot (no user accounts) | Either + sandbox mode | Lock down tools, no personal data |

## Key Docs & Sources

- [GoClaw GitHub](https://github.com/nextlevelbuilder/goclaw)
- [GoClaw Docs](https://docs.goclaw.sh)
- [GoClaw Official Site](https://goclaw.sh/en/)
- [GoClaw API Reference](https://github.com/nextlevelbuilder/goclaw/blob/main/api-reference.md)
- [OpenClaw Alternatives Comparison 2026](https://www.elegantsoftwaresolutions.com/blog/openclaw-alternatives-comparison-guide-2026)
