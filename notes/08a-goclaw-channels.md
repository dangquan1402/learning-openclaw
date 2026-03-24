# GoClaw: 7 Messaging Channels Setup Guide

> **Parent:** [GoClaw](08-goclaw.md)

## Configuration Methods

Two ways to set up channels:

| Method | How | Best For |
|--------|-----|----------|
| **Config file** (`config.json`) | JSON5 under `channels.*`, secrets in `.env` | Single instance per channel, dev/simple setups |
| **Database instances** (preferred) | `channel_instances` table, managed via web dashboard or API | Multiple instances, hot-reload, multi-tenant, encrypted credentials |

When DB instances exist, config-based channels are skipped.

## 1. Telegram

**Library:** `telego` (Go Telegram Bot API)
**Connection:** Long polling (no webhook server needed)

### Setup

1. Create bot via @BotFather → get bot token
2. Configure:
   ```json
   { "channels": { "telegram": { "enabled": true, "token": "123:ABC..." } } }
   ```
   Or env: `GOCLAW_TELEGRAM_TOKEN`

### Key Config

| Field | Default | Purpose |
|-------|---------|---------|
| `dm_policy` | `pairing` | How new DM users are authorized |
| `group_policy` | `pairing` | How groups are authorized |
| `require_mention` | `true` | Require @bot mention in groups |
| `dm_stream` | — | Edit-in-place streaming for DMs |
| `draft_transport` | — | Stealth streaming (no notifications) |
| `proxy` | — | HTTP proxy for Telegram API |
| `api_server` | — | Custom self-hosted Bot API server |
| `voice_agent_id` | — | Route voice messages to separate agent |

### Unique Features

- **Forum/topic support** with layered config (global → wildcard group → specific group → topic)
- 10+ bot commands
- Voice routing to separate agent via `voice_agent_id`
- Per-topic tool allow lists
- STT proxy integration
- **Text limit:** 4,096 chars (chunked at 4,000)

---

## 2. Discord

**Library:** `discordgo`
**Connection:** Discord Gateway WebSocket

### Setup

1. Create app at https://discord.com/developers
2. Create Bot, enable **Message Content Intent**
3. Copy bot token
4. Invite bot with permissions (Send Messages, Read Message History)
5. Configure:
   ```json
   { "channels": { "discord": { "enabled": true, "token": "MTIz..." } } }
   ```

### Required Gateway Intents

`GuildMessages`, `DirectMessages`, `MessageContent`

### Key Config

| Field | Default | Purpose |
|-------|---------|---------|
| `dm_policy` | `open` | DM access control |
| `group_policy` | `pairing` (DB) | Group access control |
| `require_mention` | `true` | Require @bot in channels |
| `history_limit` | 50 | Pending group messages for context |
| `media_max_bytes` | 25MB | Max media upload size |

### Unique Features

- "Thinking..." placeholder editing
- Typing indicator with 9-second keepalive
- Auto-detection and ignoring of own messages
- **Text limit:** 2,000 chars

---

## 3. Slack

**Library:** `slack-go/slack`
**Connection:** Socket Mode (WebSocket, no public URL needed)

### Setup

1. Create app at https://api.slack.com/apps
2. Enable **Socket Mode** → get `xapp-` App-Level Token
3. Add bot scopes: `chat:write`, `app_mentions:read`, `im:history`, `channels:history`
4. Install to workspace → get `xoxb-` Bot Token
5. Optional: `xoxp-` User Token for custom identity
6. Configure:
   ```json
   {
     "bot_token": "xoxb-...",
     "app_token": "xapp-...",
     "user_token": "xoxp-..."
   }
   ```

**Three token types (prefix validated at startup):**
- `xoxb-` — Bot Token (required)
- `xapp-` — App-Level Token (required for Socket Mode)
- `xoxp-` — User Token (optional, custom identity)

### Key Config

| Field | Default | Purpose |
|-------|---------|---------|
| `debounce_delay` | 300ms | Delay for rapid messages |
| `thread_ttl` | 24h | Hours before thread participation expires |
| `reaction_level` | — | Emoji reactions (thinking_face, hammer, checkmark) |

### Unique Features

- **Thread participation cache** — once bot replies in a thread, subsequent messages auto-trigger without @mention for 24h
- Message dedup via `channel+ts` key
- SSRF protection on file downloads (hostname allowlist for `*.slack.com`)
- Markdown → mrkdwn conversion
- **Text limit:** 4,000 chars

---

## 4. Zalo OA (Official Account)

**Protocol:** Official Zalo Bot API
**Connection:** Long polling against `https://bot-api.zaloplatforms.com` (30s poll timeout)

### Setup

1. Create Zalo Official Account at Zalo Business
2. Create bot via Zalo Bot API platform
3. Get bot token
4. Configure:
   ```json
   { "channels": { "zalo": { "enabled": true, "token": "..." } } }
   ```

### Limitations

- **DM only** — no group support at all
- Text limit: 2,000 chars
- Media: images only (5 MB)
- No streaming, no reactions
- Plain text only (markdown stripped)

---

## 5. Zalo Personal

**Protocol:** Reverse-engineered, unofficial (ported from `zcago`, MIT)
**Connection:** Custom WebSocket with own crypto layer

> **Warning:** Security warning logged on every startup: `security.unofficial_api`. Account may be locked/banned.

### Setup

1. **QR Login** via web dashboard — scan with personal Zalo account
2. Generates credentials: `{"imei":"...","cookie":{...},"userAgent":"...","language":"vi"}`
3. Stored encrypted in DB
4. Alternative: pre-load from JSON file via `credentials_path`

### Key Differences from Zalo OA

| Aspect | Zalo OA | Zalo Personal |
|--------|---------|---------------|
| Protocol | Official Bot API | **Reverse-engineered** |
| Groups | No | **Yes** |
| DM policy default | `pairing` | `allowlist` (more restrictive) |
| Auth | API token | QR scan or pre-loaded creds |
| Risk | None | **Account ban possible** |

### Resilience

- Max 10 restart attempts
- Exponential backoff up to 60s
- Special handling for error code 3000 (60s initial delay)
- **Text limit:** 2,000 chars, plain text only

---

## 6. Feishu/Lark

**Library:** Custom implementation
**Connection:** WebSocket (default) or Webhook

### Setup

1. Create app on Feishu Open Platform (https://open.feishu.cn) or Lark (https://open.larksuite.com)
2. Get `app_id` and `app_secret`
3. For webhook: also need `encrypt_key` and `verification_token`
4. Configure:
   ```json
   {
     "app_id": "cli_xxx",
     "app_secret": "...",
     "encrypt_key": "...",
     "verification_token": "..."
   }
   ```

### Connection Modes

| Mode | Config | Notes |
|------|--------|-------|
| **WebSocket** (default) | `connection_mode: "websocket"` | Persistent, auto-reconnect, no public URL |
| **Webhook** | `webhook_port: 0` (gateway port) or `> 0` (dedicated) | HTTP handler, path `/feishu/events` |

### Key Config

| Field | Default | Purpose |
|-------|---------|---------|
| `domain` | `lark` | `"lark"` (global), `"feishu"` (China), or custom URL |
| `render_mode` | `auto` | `"auto"` (detect code/tables), `"card"`, or text |
| `streaming` | `true` | Interactive card messages with real-time updates |
| `topic_session_mode` | `disabled` | Thread-per-topic isolation |
| `media_max_mb` | 30 MB | Max media upload |

### Unique Features

- **Streaming via interactive card messages** with sequence numbers, 100ms throttle, print animation
- Mention resolution (bot mentions stripped, user mentions → `@DisplayName`)
- Topic session mode for thread isolation
- Single-port operation (mounts on gateway HTTP mux)
- Media: images, files, audio, video, stickers
- **Text limit:** 4,000 chars

---

## 7. WhatsApp

**Protocol:** External WebSocket bridge (e.g., whatsapp-web.js, Baileys)
**Connection:** WebSocket to bridge server

> GoClaw does NOT implement WhatsApp protocol directly — it relays through an external bridge.

### Setup

1. Deploy a WhatsApp Web bridge server (whatsapp-web.js or Baileys-based)
2. Configure bridge URL:
   ```json
   { "channels": { "whatsapp": { "enabled": true, "bridge_url": "ws://localhost:3000" } } }
   ```

### Bridge Protocol (JSON over WebSocket)

```json
// Inbound
{"type":"message","from":"...","chat":"...","content":"...","id":"...","from_name":"..."}

// Outbound
{"type":"message","to":"...","content":"..."}
```

Group detection: chat ID ending in `@g.us`

### Limitations

- No streaming, no reactions, no rich formatting
- Plain text only
- No direct media upload (bridge handles media)
- Auto-reconnect with exponential backoff (1s → 30s max)
- **Simplest channel adapter** — essentially a JSON relay

---

## Channel Feature Comparison

| Feature | Telegram | Discord | Slack | Feishu | WhatsApp | Zalo OA | Zalo Personal |
|---------|---------|---------|-------|--------|----------|---------|---------------|
| **Streaming** | Edit-in-place | Edit-in-place | Edit (1s throttle) | Card messages | No | No | No |
| **Groups** | Yes | Yes | Yes | Yes | Yes | **No** | Yes |
| **Forum/Topics** | Yes | No | No | Yes | No | No | No |
| **Reactions** | Yes | No | Yes | Yes | No | No | No |
| **Voice/STT** | Yes | Yes | No | Yes | No | No | No |
| **Media upload** | Yes | 25MB | Yes | 30MB | Bridge | 5MB images | No |
| **Bot commands** | 10+ | No | No | No | No | No | No |
| **Per-topic tools** | Yes | No | No | No | No | No | No |
| **Protocol** | Official | Official | Official | Official | Bridge | Official | **Unofficial** |
| **Text limit** | 4,096 | 2,000 | 4,000 | 4,000 | — | 2,000 | 2,000 |

## DM Policy Defaults

| Channel | DM Policy | Group Policy |
|---------|-----------|-------------|
| Telegram (DB) | `pairing` | `pairing` |
| Discord (DB) | `open` | `pairing` |
| Slack (DB) | — | `pairing` |
| Zalo OA | `pairing` | N/A (no groups) |
| Zalo Personal | `allowlist` | `allowlist` |
| WhatsApp (DB) | `open` | `pairing` |
| Feishu (DB) | — | `pairing` |

**Pairing system:** 8-character code (no ambiguous chars), 60-minute TTL, max 3 pending per account.

## GoClaw vs OpenClaw Channel Architecture

| Aspect | OpenClaw | GoClaw |
|--------|----------|--------|
| Config method | Config file only | Config file OR database instances |
| Hot-reload | No (restart needed) | Yes (DB cache invalidation) |
| Multiple instances | One per channel type | Multiple per type (e.g., 2 Telegram bots) |
| Multi-tenant | No | Yes (`tenant_id` per instance) |
| Credential storage | Plain text in config | AES-256-GCM encrypted in DB |
| Zalo support | No | Yes (OA + Personal) |
| Feishu/Lark support | No | Yes |
| WhatsApp | Baileys (built-in) | External bridge (whatsapp-web.js) |

## Key Source Files

- `internal/channels/channel.go` — Channel interface, BaseChannel, policies
- `internal/channels/manager.go` — Lifecycle, outbound dispatch
- `internal/channels/instance_loader.go` — DB-based loading with hot-reload
- `internal/channels/{telegram,discord,slack,feishu,whatsapp}/factory.go` — Per-channel factories
- `internal/channels/zalo/zalo.go` — Zalo OA
- `internal/channels/zalo/personal/` — Zalo Personal (reverse-engineered)
- `cmd/gateway_channels_setup.go` — Channel registration
- `docs/05-channels-messaging.md` — Official docs
