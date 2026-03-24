# Deep Dive: Browser Control & Social Media Automation

> **Parent:** [Advanced Topics](05-advanced.md) | **User Story:** US5.1

## Browser Automation Architecture

OpenClaw's browser system has three layers:

| Layer | Role |
|-------|------|
| **Browser Layer** | Isolated Chromium instance (separate from personal browser) |
| **Control Layer** | Gateway HTTP API — unified control interface |
| **Agent Layer** | AI model calls browser operations via CLI/tools |

Built on **Chrome DevTools Protocol (CDP)** — bidirectional WebSocket to a Chromium browser. Uses **Playwright** (not raw Puppeteer) for advanced actions.

## Three Browser Control Modes

| Mode | Port | Use Case |
|------|------|----------|
| **Extension Relay** | 18792 | Controls existing Chrome tabs. Preserves login state and cookies |
| **OpenClaw Managed** | 18800-18899 | Isolated Chromium instances for safe automation |
| **Remote CDP** | Custom | Connects to cloud-hosted browser infrastructure |

## The Snapshot System (How AI "Sees" Pages)

Instead of CSS selectors, OpenClaw uses a **Snapshot system**:

1. Agent calls `browser_snapshot` → gets a role-based element tree with refs like `[ref=e12]`
2. AI interprets the page structure and decides actions
3. Actions use refs: `browser_click e12`, `browser_type e12 "hello"`
4. Refs resolved internally via `getByRole()`

> **Important:** Refs expire on navigation — must re-snapshot after any page change.

## Page Interaction Tools

| Tool | Purpose |
|------|---------|
| `browser_navigate` | Go to a URL |
| `browser_click` | Click an element (via ref) |
| `browser_type` | Type into form fields |
| `browser_snapshot` | Get page structure as role-based tree |
| `browser_screenshot` | Capture full page, element, or PDF |
| `browser_select_option` | Select from dropdowns |
| `browser_evaluate` | Execute arbitrary JavaScript |
| `browser_get_text` | Extract text content |
| `browser_press` | Press keyboard keys |
| `browser_choose_file` | Upload files |
| `browser_close` | Close browser |

## Typical Browser Automation Flow

```
1. Start gateway with browser support
   ↓
2. Choose mode (Extension Relay for logged-in, Managed for isolation)
   ↓
3. Send natural language instruction
   e.g., "Go to example.com and fill out the contact form"
   ↓
4. Agent: browser_navigate → load page
   ↓
5. Agent: browser_snapshot → get element tree with refs
   ↓
6. Agent: browser_click/browser_type with refs
   ↓
7. After navigation: re-snapshot for fresh refs
   ↓
8. Agent: browser_screenshot for verification
   ↓
9. Repeat until task complete
```

## Social Media Automation

### Cross-Platform Posting Services

| Service | Platforms | Notes |
|---------|-----------|-------|
| **Post Bridge** | Instagram, TikTok, YouTube, X, LinkedIn, Facebook, Threads, Bluesky, Pinterest | Connect accounts → generate API key → add to OpenClaw |
| **PostFast** | TikTok, Instagram, Facebook, X, YouTube, LinkedIn, Threads, Bluesky, Pinterest, Telegram | Natural language scheduling |
| **Mixpost** | Multiple | Self-hosted social media scheduler |
| **Genviral** | 6 platforms | Automated content generation + posting |

### Twitter/X Skills

| Skill | Notes |
|-------|-------|
| **opentweet/x-poster** | Most popular (8,000+ users). OAuth flow — X credentials never touch OpenClaw. $5.99/mo vs $100/mo Twitter API |
| **tweet-cli** | Alternative CLI-based approach |
| **twitter-openclaw** | Direct API credential approach |
| **x-skill** (yongkangc) | Uses "bird" CLI |

### YouTube

- YouTube toolkit for transcripts, metadata, channel research, playlist analysis
- Video-to-clip repurposing tools
- Short-form publishing asset generation

### Skill Installation

```bash
clawhub install publisher/skill-name    # Install
clawhub list                            # List installed
clawhub uninstall publisher/skill-name  # Remove
```

Skills directory: `~/.openclaw/skills/`

There are **17+ free social media skills** and **5,400+ total skills** in the registry.

## API-Based vs Browser Automation

| Approach | Pros | Cons | Best For |
|----------|------|------|----------|
| **API-based skills** | Faster, more reliable, no browser overhead | Requires API keys, may cost money, rate limits | Routine posting, scheduling, data retrieval |
| **Browser automation** | Works with any website, uses existing login | Slower, fragile to UI changes, needs browser instance | Sites without APIs, complex UI interactions, one-off tasks |

**Recommendation:** Use API-based skills for routine social media posting. Use browser automation as a fallback for sites without APIs or for complex multi-step workflows.

## Authentication Handling

| Method | How It Works |
|--------|-------------|
| **Cookie persistence** | Read/write cookies; browser state (cookies + localStorage) persists between sessions |
| **Extension Relay** | Attach to already logged-in browser — keeps existing session |
| **Auto-login** | Agent fills username/password forms automatically |
| **Cookie export/import** | Log in manually → export cookies → import during automation |
| **OAuth flows** | Skills like x-poster handle OAuth — credentials never touch OpenClaw directly |

## Cloudflare Moltworker Browser

The [Moltworker](https://github.com/cloudflare/moltworker) project runs OpenClaw on Cloudflare Workers:

- Includes **CDP shim** for headless browser automation
- Uses **Cloudflare R2** for persistent storage
- Protected by **Cloudflare Access** (Zero Trust)
- **Status: proof of concept** — not officially supported, may break

Related: **Cloud Claw** integrates Cloudflare Browser Rendering for headless CDP.

## MCP Browser Integrations

| Integration | Tools Provided |
|-------------|---------------|
| **Playwright MCP** | navigate, click, type, select, get_text, evaluate, snapshot, choose_file, press, close |
| **Chrome DevTools MCP** | Full browser control via CDP |
| **Composio Browser Tool** | Browser Tool MCP + Cloudflare Browser Rendering MCP |

## Security Considerations

- Browser automation = **same caution as remote desktop access**
- Keep gateway and node hosts on a **private network**
- Use a **dedicated Chrome profile** for automation — never your personal profile
- Only include **cookies/credentials needed** for the specific task
- **Never store credentials in config** — use env vars or secrets manager
- OpenClaw Managed mode provides **isolated Chromium** separate from personal browsing
- Cloudflare deployments use **Zero Trust** for Admin UI protection

## Scheduled Posting with Heartbeat

Combine browser automation or API skills with Heartbeat for scheduled social media:

1. Create a skill for posting (API-based or browser-based)
2. Configure `HEARTBEAT.md` with posting schedule
3. Agent checks on each heartbeat if it's time to post
4. Alternatively, use **Cron** for precise scheduling (e.g., "Post at 9am Monday")

Example `HEARTBEAT.md` entry:
```markdown
## Social Media
- Check if there are scheduled posts pending in projects.md
- If a post is due, use the x-poster skill to publish it
- Log the result to memory/YYYY-MM-DD.md
```

## Key Docs & Sources

- [Browser (OpenClaw-managed) - Official Docs](https://docs.openclaw.ai/tools/browser)
- [Browser Automation Guide - Apiyi.com](https://help.apiyi.com/en/openclaw-browser-automation-guide-en.html)
- [Browser Automation Complete Guide 2026](https://levelup.gitconnected.com/openclaw-browser-automation-the-complete-guide-for-2026-58826734fc98)
- [Post Bridge + OpenClaw](https://www.post-bridge.com/openclaw)
- [PostFast + OpenClaw](https://postfa.st/blog/how-to-schedule-social-media-posts-with-ai-using-postfast-and-openclaw)
- [OpenTweet X/Twitter Skills](https://opentweet.io/blog/openclaw-twitter-skills-directory)
- [Cloudflare Moltworker](https://github.com/cloudflare/moltworker)
- [Social Media Skills Directory](https://openclawdesktop.com/skills/topic/social-media.html)
- [awesome-openclaw-skills](https://github.com/VoltAgent/awesome-openclaw-skills)
