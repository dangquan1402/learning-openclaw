# Skills Development

## What Are Skills?

Skills are modular capability packages -- primarily Markdown files -- that teach the AI agent how to perform specific tasks. They package functionality (API calls, database queries, workflows) into reusable components.

Think of them as **apps on a smartphone** where OpenClaw is the OS.

## SKILL.md File Format

### YAML Frontmatter

```yaml
---
name: my-skill
description: "Short summary. Use when: [specific triggers]."
version: 1.0.0
user-invocable: true
disable-model-invocation: false
emoji: "🔧"
metadata:
  openclaw:
    requires:
      env:
        - MY_API_KEY
      bins:
        - curl
        - jq
      config:
        - some.config.key
    primaryEnv: MY_API_KEY
    homepage: https://example.com
    install: "npm install some-dependency"
---
```

### Required Fields

| Field | Purpose |
|-------|---------|
| `name` | Skill identifier |
| `description` | **Critical for triggering** -- how the agent decides when to use the skill |
| `version` | Semver string |

### Optional Fields

| Field | Default | Purpose |
|-------|---------|---------|
| `user-invocable` | `true` | Expose as slash command |
| `disable-model-invocation` | `false` | User-only invocation (exclude from model prompt) |
| `command-dispatch` | -- | Set to `tool` for direct tool routing without reasoning |
| `emoji` | -- | Icon displayed when skill activates |
| `requires.env` | -- | Required environment variables |
| `requires.bins` | -- | Required system binaries (ALL must exist) |
| `requires.anyBins` | -- | At least ONE must exist |
| `requires.config` | -- | Required config keys |
| `primaryEnv` | -- | Main credential env var |
| `install` | -- | Shell commands for first-run setup |

### Instruction Body

Below the frontmatter: step-by-step instructions in Markdown. The body is **only loaded after the skill triggers** -- the agent reads only `name` + `description` first.

> **YAML Gotcha:** Values containing `: ` (colon + space) are interpreted as nested YAML. Always wrap descriptions in quotes!

## Skill Types

### Markdown-Only (Most Common)

A `SKILL.md` file with YAML frontmatter and plain-English instructions. No programming required -- the agent follows written instructions to use existing tools (Bash, HTTP calls via curl, etc.).

Can include supporting scripts (Python, Bash, Node.js) alongside SKILL.md.

### TypeScript/JavaScript Function Skills

For complex integrations:

```typescript
export default {
  name: "my-skill",
  description: "...",
  parameters: { /* JSON Schema */ },
  async execute(params) { /* logic */ }
}
```

## Skill Eligibility

How OpenClaw decides which skills to inject:

1. **Binary presence** (`requires.bins`) -- all must exist on host
2. **Environment variables** (`requires.env`) -- all must be set
3. **Config keys** (`requires.config`) -- must be available
4. **Platform restrictions** -- optional `os` field (darwin, linux, win32)
5. **Enabled flag** -- can be explicitly disabled

Verify eligible skills: `openclaw skills list --eligible`

**Important:** Only skills relevant to the current turn are fully injected into the prompt. This prevents prompt bloat.

## ClawHub Marketplace

The public skill registry -- like npm for AI agents.

- **13,700+** community-built skills
- **5,400+** curated in awesome-openclaw-skills

### CLI Commands

```bash
# Search (supports natural language)
openclaw skill search "manage calendar events"

# Install
openclaw skill install <name>

# Update
openclaw skill update <name>

# Publish
openclaw skill publish <name>
```

### Security Warning

In Feb 2026, the **"ClawHavoc"** supply chain attack poisoned ClawHub with 1,184 malicious skills containing credential stealers and reverse shells. **Always review skills before installation.**

## Example Skills

### 1. Hello World (Minimal)

```yaml
---
name: hello-world
description: "Greet the user. Use when: someone says hello or asks for a greeting."
version: 1.0.0
---

When triggered, respond with a friendly greeting that includes the current date and time.
Use the Bash tool to get the current time: `date`
```

### 2. API Integration (Stock Price)

```yaml
---
name: stock-price
description: "Fetch current stock prices. Use when: user asks for stock price, ticker lookup, or market data."
version: 1.0.0
metadata:
  openclaw:
    requires:
      env:
        - ALPHAVANTAGE_API_KEY
      bins:
        - curl
        - jq
    primaryEnv: ALPHAVANTAGE_API_KEY
---

## Steps
1. Take the stock ticker symbol from the user's request
2. Call the AlphaVantage API:
   ```bash
   curl -s "https://www.alphavantage.co/query?function=GLOBAL_QUOTE&symbol=${TICKER}&apikey=${ALPHAVANTAGE_API_KEY}" | jq '.["Global Quote"]'
   ```
3. Parse and present the price, change, and volume in a readable format
```

### 3. Automation (Test Runner)

A skill that instructs the agent on how to run, parse, and report test results for a project. Found at `skills/cmanfre7/test-runner/SKILL.md` in the official repo.

## Best Practices

1. **Description is a trigger phrase, not documentation.** Write it to indicate *when* to use the skill. Include nouns users actually type ("log summary", "deploy checklist").

2. **Keep SKILL.md under 500 lines.** Point to reference files for detailed knowledge.

3. **Reference files over 100 lines should have a table of contents.**

4. **Test-driven development:** Write test cases -> watch agent fail without the skill (baseline) -> write the skill -> verify tests pass -> refactor.

5. **Never hardcode secrets.** Reference credentials by env var name only.

6. **Quote descriptions** containing `: ` to avoid YAML parse failures.

7. **Keep "when to use" info in frontmatter description**, not in the body (body loads only after triggering).

## Skills vs Plugins vs MCP

| Aspect | Skills | Plugins | MCP |
|--------|--------|---------|-----|
| **What** | Markdown instructions | Code inside OpenClaw | Protocol for external services |
| **Language** | Markdown + scripts | TypeScript/JS | Standardized protocol |
| **Scope** | Task-level | Platform-level | External integration |
| **Install** | `openclaw skill install` | `openclaw plugins install` | Via mcporter |
| **Registry** | ClawHub (13,700+) | Fewer | Separate ecosystem |

OpenClaw's creator chose skills over native MCP, with MCP available as fallback through `mcporter`.

## Key File Locations

```
~/.openclaw/workspace/skills/<name>/SKILL.md   # User skills
.openclaw/skills/<name>/SKILL.md               # Project-local skills
```
