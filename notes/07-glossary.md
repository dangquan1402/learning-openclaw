# Glossary

| Term | Definition |
|------|-----------|
| **Gateway** | The always-on WebSocket server that routes messages between channels and agents. Default port: 18789 |
| **Brain** | The LLM-powered reasoning engine that processes messages using the ReAct loop |
| **ReAct** | Reason + Act. The agent loop pattern: receive message -> reason -> call tool -> observe result -> repeat |
| **Memory** | Markdown-based persistent context. Daily logs + long-term MEMORY.md |
| **Skill** | A markdown-based capability package (SKILL.md) that teaches the agent how to perform tasks |
| **Plugin** | A TypeScript/JS code module that extends the Gateway runtime at infrastructure level |
| **Heartbeat** | Proactive scheduler. Agent wakes up every N minutes to check for tasks |
| **Cron** | Precise time-based scheduling with cron expressions. Runs in isolated sessions |
| **ClawHub** | The public skill marketplace/registry for OpenClaw (13,700+ skills) |
| **Channel Adapter** | Platform-specific connector that normalizes messages between a platform and OpenClaw |
| **Session** | A conversation context bound to a user/channel/agent combination |
| **Workspace** | Directory containing agent files: skills, memory, persona, config |
| **SKILL.md** | The file defining a skill. Contains YAML frontmatter + Markdown instructions |
| **AGENTS.md** | Bootstrap file defining the agent's persona and behavior rules |
| **SOUL.md** | File defining the agent's tone, personality, and communication style |
| **MEMORY.md** | Long-term durable memory file at workspace root |
| **HEARTBEAT.md** | File defining what the agent should check during each heartbeat |
| **Control UI** | Web dashboard for managing the Gateway, served at port 18789 |
| **WebChat** | Browser-based chat interface served by the Gateway |
| **Ollama** | Local LLM runner. Free, private alternative to cloud API providers |
| **jiti** | Just-in-time TypeScript compiler used to load plugins without a build step |
| **Baileys** | Library used for WhatsApp Web protocol integration |
| **grammY** | Library used for Telegram Bot API integration |
| **mcporter** | Tool that converts MCP connectors to OpenClaw tools (MCP compatibility layer) |
| **Sandbox** | Docker-based isolation for tool execution. Off by default |
| **Memory Flush** | Silent agentic turn before context compaction to persist important info |
| **Pairing** | Security flow where new users must approve a code before messaging the agent |
