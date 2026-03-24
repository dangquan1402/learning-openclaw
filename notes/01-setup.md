# Setup & Installation

## System Requirements

| Requirement | Details |
|-------------|---------|
| **Node.js** | v24 (recommended) or v22.16+ LTS |
| **RAM** | 2 GB minimum for Docker builds; lighter at runtime |
| **OS** | macOS, Linux, Windows (WSL2 recommended) |
| **Package Manager** | npm, pnpm, or bun (pnpm preferred for source builds) |

## Installation Methods

### Method A: Global npm Install (Simplest)

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

Or with pnpm:

```bash
pnpm add -g openclaw@latest
openclaw onboard --install-daemon
```

The `--install-daemon` flag sets up auto-start (LaunchAgent on macOS, systemd on Linux).

### Method B: Build from Source

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
pnpm openclaw onboard --install-daemon
```

For development with auto-reload: `pnpm gateway:watch`

### Method C: Docker

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
./scripts/docker/setup.sh
```

The setup script builds the image, runs onboarding, generates a gateway token, and starts via Docker Compose.

**Docker directories created:**
- `~/.openclaw` -- configuration (memory, config, API keys)
- `~/openclaw/workspace` -- workspace files available to the agent

Check logs: `docker compose logs -f`

## The Onboarding Wizard

Run `openclaw onboard` for a guided setup. Two modes:

### QuickStart Mode
Sensible defaults, minimal decisions. Sets port 18789, token auth, pick your model.

### Advanced Mode
Full control over every step:

1. **Model/Auth** -- Choose LLM provider and default model
2. **Workspace** -- Location for agent files (default: `~/.openclaw/workspace`)
3. **Gateway** -- Port (18789), bind address, auth mode, optional Tailscale
4. **Channels** -- Connect messaging platforms (WhatsApp, Telegram, Discord, etc.)
5. **Web Search** -- Pick search provider (Perplexity, Brave, Gemini, Grok, Kimi)
6. **Daemon** -- Auto-start service installation
7. **Health Check** -- Verify gateway is running
8. **Skills** -- Install recommended skills

## LLM Provider Options

| Provider | How to Configure |
|----------|-----------------|
| **Anthropic (Claude)** | `ANTHROPIC_API_KEY=sk-ant-...` (recommended) |
| **OpenAI (GPT)** | `OPENAI_API_KEY=sk-...` |
| **Google Gemini** | Via onboarding wizard |
| **Ollama (local)** | Free, private. Auto-detected at `http://127.0.0.1:11434` |
| **OpenRouter** | Access to many models via single API |
| **xAI (Grok)** | Via onboarding |
| **Groq / Cerebras** | Fast inference providers |
| **Mistral** | Via onboarding |
| **GitHub Copilot** | Via onboarding |
| **Custom** | Any OpenAI-compatible or Anthropic-compatible API |

### Using Ollama (Free Local Models)

- Install Ollama from [ollama.com](https://ollama.com)
- Set `OLLAMA_API_KEY` or select during onboarding
- Completely free and private -- no data leaves your machine

## Control UI (Port 18789)

The Gateway serves two web interfaces at `http://127.0.0.1:18789/`:

- **Control UI** -- Dashboard for managing gateway, viewing sessions, configuring settings
- **WebChat** -- Browser-based chat with your agent

Access: Run `openclaw dashboard` or visit the URL directly. Authenticate with your gateway token.

## Diagnostic Tool

```bash
openclaw doctor
```

Checks Node.js version, Docker status, API keys, port availability, and common issues.

## Common Issues & Fixes

| Problem | Solution |
|---------|----------|
| **Node.js version mismatch** | Use `node -v` to check. Need 22.16+. Use nvm/fnm to manage |
| **Sharp dependency error (macOS)** | Install Xcode CLI tools: `xcode-select --install` |
| **EACCES permission error** | Set npm prefix: `npm config set prefix ~/.npm-global` |
| **Windows path issues** | Install Node.js to path without spaces, or use WSL2 |
| **Terminal not finding openclaw** | Restart terminal after Node.js install |
