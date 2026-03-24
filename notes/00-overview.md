# What is OpenClaw?

## Summary

OpenClaw is a **free, open-source autonomous AI agent framework** that you self-host. It connects to 20+ messaging platforms (WhatsApp, Telegram, Slack, Discord, Signal, iMessage, Teams, etc.) and acts autonomously -- monitoring inboxes, following up with contacts, and executing multi-step tasks.

- **Language:** TypeScript (500K+ lines)
- **License:** MIT
- **GitHub:** [openclaw/openclaw](https://github.com/openclaw/openclaw)
- **Docs:** [docs.openclaw.ai](https://docs.openclaw.ai)
- **Stars:** 250K+ (one of the most-starred repos on GitHub)

## History

| Date | Event |
|------|-------|
| Nov 2025 | First published as "Clawdbot" by Peter Steinberger (Austrian developer) |
| Jan 27, 2026 | Renamed to "Moltbot" after Anthropic trademark complaints |
| Jan 30, 2026 | Renamed again to "OpenClaw" |
| Feb 14, 2026 | Founder joined OpenAI; project moved to open-source foundation |

## Core Concept

OpenClaw is a **hub-and-spoke architecture** centered on a single always-on process called the **Gateway**. The Gateway:

1. Connects to all your messaging platforms simultaneously
2. Routes incoming messages to an AI agent
3. The agent reasons, calls tools, and responds
4. Can act proactively on a schedule (Heartbeat)

Think of it as: **a personal AI assistant that lives across all your chat apps.**

## Five Core Components

| Component | Role |
|-----------|------|
| **Gateway** | Always-on control plane. Routes messages, manages sessions, serves Control UI |
| **Brain** | Agent loop using ReAct reasoning. Orchestrates LLM calls and tool execution |
| **Memory** | Markdown-based persistent context stored locally |
| **Skills** | Plugin capabilities defined in SKILL.md files. 13,700+ available on ClawHub |
| **Heartbeat** | Scheduler for proactive tasks (inbox monitoring, follow-ups, briefings) |

## Key Differentiators

- **Self-hosted** -- runs on your machine, your data stays local
- **Model-agnostic** -- works with Anthropic, OpenAI, Google, Ollama (local), and more
- **Multi-channel** -- one agent across all your messaging apps
- **Skills over MCP** -- uses Markdown-based skill system (simpler, more flexible)
- **Proactive** -- can act without user prompting via Heartbeat/Cron

## Prerequisites to Learn OpenClaw

- **Node.js** (v24 recommended, or v22.16+ LTS)
- **TypeScript/JavaScript** knowledge (for skills/plugins)
- **Basic LLM/prompt engineering** understanding
- **Command-line comfort**
- **An LLM API key** (or Ollama for free local models)

## Not to Be Confused With

- **OpenLaw** -- a separate project for blockchain-enabled legal smart contracts ([docs.openlaw.io](https://docs.openlaw.io))
