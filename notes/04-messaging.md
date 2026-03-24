# Messaging Integration

## Supported Platforms

### Tier 1 (Mature, Full Feature Support)

| Platform | Library | Notes |
|----------|---------|-------|
| **Telegram** | grammY | Most popular, easiest setup. Full images, files, inline keyboards |
| **Discord** | Native | Slash commands, rich embeds. Great for community bots |
| **Slack** | Slack App API | Requires creating app at api.slack.com/apps |
| **WhatsApp** | Baileys | Uses WhatsApp Web protocol. Needs internet-accessible instance |

### Tier 2 (Supported)

- **Signal** (via signal-cli, privacy-focused)
- **iMessage** (BlueBubbles recommended for new setups)
- **Microsoft Teams** (via Bot Framework plugin)
- **Google Chat**, **Matrix**, **IRC**, **Mattermost**

### Tier 3 (Community/Regional)

LINE, Feishu, Zalo, Nostr, Synology Chat, Tlon, Twitch, Nextcloud Talk, WebChat

## Gateway Message Routing

Hub-and-spoke architecture:

```
User Message -> Platform -> Channel Adapter -> Gateway -> Session -> Agent
                                                                      |
Agent Response <- Platform <- Channel Adapter <- Gateway <------------|
```

1. Message arrives on any channel
2. **Channel adapter** normalizes platform-specific payload to OpenClaw format
3. **Gateway** examines source platform, user ID, channel/group
4. Routes to correct **session** bound to an agent
5. Agent loads context + skills, calls LLM, executes tools
6. Response streams back through same path
7. Channel adapter renders to platform format (Slack blocks, Telegram keyboards, etc.)

**Multi-Agent Routing:** Gateway can route different platform/user combos to independent agents (e.g., Slack -> "work" agent, Telegram -> "personal" agent).

## Telegram Setup (Step-by-Step)

### 1. Create a Bot with BotFather

1. Open Telegram, search for `@BotFather`
2. Send `/newbot`
3. Choose a display name, then a username (must end in `bot`)
4. BotFather responds with your **Bot Token** -- save it securely

### 2. Configure OpenClaw

Edit `~/.openclaw/openclaw.json` and add Telegram channel config with your `botToken`.

Set `dmPolicy`:
- `pairing` (default) -- new users must complete a pairing flow with 1-hour code
- `allowlist` -- only pre-approved user IDs can message
- `open` -- anyone can message (not recommended for production)

### 3. Start and Test

```bash
openclaw start   # or restart if already running
```

Message your bot on Telegram. If using pairing mode, approve the code via CLI.

**Notes:**
- Telegram uses long polling (no webhook setup needed)
- Does NOT use `openclaw channels login`
- Get a fresh token anytime: `@BotFather` -> `/mybots` -> select bot -> API Token

## Discord Setup (Step-by-Step)

### 1. Create a Discord Application

1. Go to [discord.com/developers](https://discord.com/developers)
2. Click "New Application", name it, create it

### 2. Add a Bot User

1. Navigate to Bot section, click "Add Bot"
2. Copy the **Bot Token**

### 3. Enable Required Intents

Under Privileged Gateway Intents, enable:
- `Message Content Intent` (required)
- `Server Members Intent` (recommended)

### 4. Configure Permissions

Bot needs: Send Messages, Read Message History, Use Slash Commands

### 5. Generate Invite Link & Add to Server

1. Use OAuth2 URL Generator, select `bot` scope + permissions
2. Open generated URL, select your server, authorize

### 6. Configure OpenClaw

Add Discord channel config to `~/.openclaw/openclaw.json` with bot token. Restart Gateway.

**Security Tips:**
- Create a dedicated `#openclaw` channel
- Keep mention-gating on in public channels
- If enabling DMs, keep pairing enabled

## Heartbeat for Scheduled Tasks

Once a channel is connected, configure the Heartbeat for proactive tasks:

1. Create/edit `HEARTBEAT.md` in your workspace
2. Define what the agent should check on each heartbeat
3. Default interval: 30 minutes (configurable)

Example `HEARTBEAT.md`:
```markdown
## On each heartbeat, check:
1. My email inbox for urgent messages
2. Calendar for upcoming meetings in the next hour
3. GitHub notifications for mentions
```

The agent will either:
- Send `HEARTBEAT_OK` (silent, nothing to report)
- Message you if something needs attention
