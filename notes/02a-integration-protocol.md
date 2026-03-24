# Deep Dive: Integration Protocol (Channel Adapters)

> **Parent:** [Architecture](02-architecture.md) | **User Story:** US2.1

## Channel Adapter Interface

Each channel adapter implements the **ChannelPlugin** interface with these sub-adapters:

| Sub-Adapter | Purpose |
|-------------|---------|
| **ChannelConfigAdapter** | Account discovery, config validation |
| **ChannelStatusAdapter** | Health monitoring (`probeAccount`, `auditAccount`, `collectStatusIssues`) |
| **ChannelOutboundAdapter** | Message delivery (`sendText`, `sendMedia`, `sendPoll`, `sendPayload`) |
| **Security Adapter** | Access control enforcement |
| **Lifecycle Hooks** | `onboarding`, `setup`, `reload` |

### Outbound Adapter Details

```
sendText(ChannelOutboundContext) -> Promise<OutboundDeliveryResult>
sendMedia(ChannelOutboundContext) -> Promise<OutboundDeliveryResult>
sendPoll(ChannelPollContext) -> Promise<ChannelPollResult>
sendPayload(ChannelOutboundPayloadContext) -> Promise<OutboundDeliveryResult>
```

The adapter also declares:
- `deliveryMode`: `"direct"` (HTTP per message), `"gateway"` (persistent connection), or `"hybrid"`
- `chunker` / `chunkerMode` / `textChunkLimit` -- message splitting config
- `resolveTarget(config, target, allowFrom, accountId, mode)` -- maps abstract targets to platform addresses

## Unified Internal Message Format (MsgContext)

All platform payloads normalize into **MsgContext**:

### Core Fields

| Field | Type | Description |
|-------|------|-------------|
| `Body` | string | Raw message text |
| `BodyForAgent` | string | Agent-formatted body (may differ from raw) |
| `Channel` | string | Channel identifier (e.g., "telegram", "slack") |
| `From` | string | Sender identifier |
| `To` | string | Recipient identifier |
| `ChatType` | enum | `"direct"` \| `"group"` \| `"channel"` |
| `MessageId` | string | Platform-native message ID |
| `ReplyToId` | string | Thread/reply reference |
| `MediaUrl` | string | Media attachment URL |
| `CommandAuthorized` | boolean | Controls directive processing |

### Extended Fields

| Field | Description |
|-------|-------------|
| `BodyStripped` | Body with mentions removed |
| `SessionId` | Current session identifier |
| `IsNewSession` | `"true"` or `"false"` |

Before entering the pipeline, MsgContext is finalized to **FinalizedMsgContext** -- ensures `CommandAuthorized` is always set (default-deny: missing becomes `false`).

## WebSocket Connection Management

OpenClaw runs as a **single Node.js process** (`127.0.0.1:18789`). The Gateway is a WebSocket server:

- **Auth is fail-closed** -- no token = connections refused
- Clients authenticate with `connect.auth.token` matching `gateway.auth.token` (or `OPENCLAW_GATEWAY_TOKEN` env var)
- One long-running process handles all platform connections simultaneously

### Platform-Specific Strategies

| Platform | Strategy |
|----------|----------|
| **Telegram** | grammY library with sequentialization + throttling. Long polling (no webhooks needed) |
| **Discord** | Persistent gateway connection via Discord.js |
| **Slack** | Bolt framework with Socket Mode or HTTP events |
| **WhatsApp** | Baileys library (WhatsApp Web protocol) |

## End-to-End Message Flow (Telegram Example)

```
Telegram Update
    │
    ▼
1. CHANNEL INGRESS
   grammY handler receives raw platform event
    │
    ▼
2. NORMALIZATION
   Extract text, sender, media, thread info → construct MsgContext
    │
    ▼
3. DEDUPLICATION
   Gateway checks for duplicate message IDs
    │
    ▼
4. ACCESS CONTROL
   DM/group policies enforced → CommandAuthorized evaluated
   MsgContext → FinalizedMsgContext
    │
    ▼
5. SESSION RESOLUTION
   DM key:    agent:{agentId}:telegram:{chatId}
   Group key: agent:{agentId}:telegram:group:{groupId}:topic:{threadId}
   Load session from ~/.openclaw/agents/<agentId>/sessions/
    │
    ▼
6. PROCESSING
   Fast path: recognized command → handle immediately (no LLM)
   Agent path: assemble context + memory → LLM → tool calls → persist
    │
    ▼
7. REPLY DELIVERY
   Chunk response per channel limits
   DM: sendMessageDraft API for live preview
   Group: send preview → edit in place as text accumulates
   Final: sendText/sendMedia via outbound adapter
    │
    ▼
Telegram renders message to user
```

## Outbound Rendering by Platform

OpenClaw uses a **channel_hints** system to render platform-native rich messages:

| Platform | Rich Format | Notes |
|----------|------------|-------|
| **Slack** | Block Kit structures | Click/selection interactions route back via Slack events |
| **Telegram** | `inline_keyboard` with callback data | Interactive buttons |
| **Discord** | Rich embeds + slash commands | Native Discord formatting |
| **WhatsApp** | Template-based messages | Structured templates |
| **Fallback** | Plain text | When no channel hint provided |

When `streaming=off`, the outbound dispatcher **buffers all content blocks** from one model response and concatenates into a single payload before sending.

## Building a Custom Channel Adapter

To build a custom channel plugin:

1. **Create a plugin package** with `openclaw.plugin.json` manifest
2. **Implement ChannelPlugin interface** (config, status, outbound, security sub-adapters)
3. **Register channel ID** in manifest's `channels` array
4. **Implement inbound normalization** -- convert platform events to MsgContext
5. **Implement outbound delivery** -- `sendText`, `sendMedia`, `sendPoll`, `sendPayload`
6. **Implement health checks** -- `probeAccount` and `auditAccount`

### Plugin Manifest (`openclaw.plugin.json`)

```json
{
  "id": "my-channel",
  "channels": ["my-platform"],
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

**Fields:**
- `id` (required) -- canonical plugin ID
- `configSchema` (required) -- JSON Schema for plugin configuration
- `channels` (optional) -- channel IDs registered
- `providers` (optional) -- provider IDs
- `skills` (optional) -- skill directories to load
- `kind` (optional) -- plugin kind (e.g., `"memory"`)

**Community example:** [openclaw-channel-dingtalk](https://github.com/soimy/openclaw-channel-dingtalk)

## Multi-Agent Routing

**Bindings** map `(channel, accountId, peer)` tuples to an `agentId`.

### Priority Hierarchy (most-specific wins)

| Priority | Match Level | Field |
|----------|------------|-------|
| 1 (highest) | Exact peer | `peer.kind` + `peer.id` |
| 2 | Guild | `guildId` (Discord) |
| 3 | Team | `teamId` (Slack) |
| 4 | Account | `accountId` on channel |
| 5 | Channel | Any account on that channel |
| 6 (lowest) | Default | First agent or one marked `default` |

### Rules

- Multiple match fields in one binding use **AND** semantics (all must match)
- Same-tier conflicts resolved by **config order** (first listed wins)
- Put most specific bindings first
- Agents are fully isolated (separate auth, sessions, workspace) with no cross-talk

## Session Key Format

```
DM:    agent:{agentId}:{provider}:{chatId}
Group: agent:{agentId}:{provider}:group:{groupId}:topic:{threadId}
```

Session stores: `~/.openclaw/agents/<agentId>/sessions/`

## Key Docs & Sources

- [Chat Channels - OpenClaw](https://docs.openclaw.ai/channels)
- [Messages - OpenClaw](https://docs.openclaw.ai/concepts/messages)
- [Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)
- [Plugins - OpenClaw](https://docs.openclaw.ai/tools/plugin)
- [Channel Architecture - DeepWiki](https://deepwiki.com/openclaw/openclaw/4.1-channel-architecture)
- [openclaw-channel-dingtalk (example)](https://github.com/soimy/openclaw-channel-dingtalk)
