# MCPorter Investigation

> **Repo:** [steipete/mcporter](https://github.com/steipete/mcporter)  
> **Tagline:** "Call MCPs via TypeScript, masquerading as simple TypeScript API. Or package them as cli."

## What It Is

MCPorter is a **TypeScript runtime, CLI, and code-generation toolkit for MCP**.

It is not an MCP server itself. It sits **on the client side** and makes existing MCP servers easier to:

- discover
- call from the terminal
- call from TypeScript
- wrap as single-purpose CLIs

Think of it as a **developer ergonomics layer over the MCP SDK**.

## What Problem It Solves

Raw MCP usage in TypeScript usually has several annoyances:

- each editor/tool stores MCP config in a different place
- tool schemas are verbose to inspect manually
- calling tools directly from scripts requires connection/setup boilerplate
- sharing a single useful MCP tool as a CLI takes extra wrapper work

MCPorter tries to remove that friction.

## Core Idea

MCPorter gives you one surface for several workflows:

1. **Discover MCP servers you already configured**
2. **Inspect tools and signatures**
3. **Call tools from CLI**
4. **Generate typed TypeScript clients**
5. **Generate dedicated CLIs for a server**

That makes it useful both for:

- human operators in the terminal
- TypeScript agents/scripts
- internal tooling teams that want reusable wrappers around MCP servers

## Main Capabilities

### 1. Auto-discovery of existing MCP configs

MCPorter can import server definitions from multiple tools instead of forcing you to duplicate config by hand.

It supports imports from:

- Cursor
- Claude Code
- Claude Desktop
- Codex
- Windsurf
- OpenCode
- VS Code

This is one of the most practically useful features because MCP config is often fragmented across tools.

### 2. Direct tool calling from CLI

Examples:

```bash
npx mcporter list
npx mcporter list context7 --schema
npx mcporter call linear.create_comment issueId:ENG-123 body:'Looks good!'
npx mcporter call 'linear.create_comment(issueId: "ENG-123", body: "Looks good!")'
```

This turns MCP servers into something much closer to a normal command-line tool.

### 3. Typed TypeScript clients

`mcporter emit-ts` can generate:

- `.d.ts` interfaces
- client wrappers that call MCP tools through a proxy

Example:

```bash
npx mcporter emit-ts linear --out types/linear-tools.d.ts
npx mcporter emit-ts linear --mode client --out clients/linear.ts
```

This is useful when you want MCP tools to feel like ordinary typed TypeScript functions.

### 4. Generate standalone CLIs

`mcporter generate-cli` can turn one MCP server into a dedicated CLI.

Example:

```bash
npx mcporter generate-cli linear --bundle dist/linear.js
```

That is helpful when you want to package a narrow "one weird tool" experience for yourself or teammates without exposing the whole MCP stack.

### 5. Ad-hoc HTTP and stdio targets

You can point MCPorter at:

- HTTP MCP endpoints
- stdio servers
- ad-hoc commands

without necessarily storing permanent config first.

That makes it good for experimentation and quick testing.

## How It Works Internally

The repository describes three main layers:

### Runtime orchestration

`createRuntime()` loads server definitions from config/imports/ad-hoc flags, handles auth and transport setup, and manages cleanup.

### CLI surface

Key commands:

- `mcporter list`
- `mcporter call`
- `mcporter generate-cli`
- `mcporter emit-ts`
- `mcporter inspect-cli`

### Proxy helper

`createServerProxy()` exposes MCP tools as friendlier camelCase methods and returns a `CallResult` helper with utilities like:

- `.text()`
- `.markdown()`
- `.json()`
- `.content()`

## Why It Is Interesting

MCPorter is interesting because it treats MCP less like a pure protocol integration problem and more like a **developer experience problem**.

Its value is not new reasoning logic. Its value is:

- easier local adoption
- easier scripting
- easier sharing of tool wrappers
- less MCP boilerplate in TypeScript projects

## How To Use It

### 1. Try it without installation

```bash
npx mcporter list
```

If you already have MCP servers configured in supported tools, MCPorter can often discover them immediately.

### 2. Inspect one server

```bash
npx mcporter list context7 --schema
```

This shows available tools and schema details.

### 3. Call a tool

```bash
npx mcporter call linear.create_comment issueId:ENG-123 body:'Looks good!'
```

Or function-call style:

```bash
npx mcporter call 'linear.create_comment(issueId: "ENG-123", body: "Looks good!")'
```

### 4. Use it from TypeScript

Generate a client or types:

```bash
npx mcporter emit-ts linear --mode client --out clients/linear.ts
```

Then your code can use a generated wrapper instead of wiring the MCP SDK manually.

### 5. Package a dedicated CLI

```bash
npx mcporter generate-cli linear --bundle dist/linear.js
```

That produces a more shareable wrapper around the target MCP server.

## Config Model

MCPorter looks for config in this rough order:

1. explicit `--config`
2. `MCPORTER_CONFIG`
3. project `config/mcporter.json`
4. home `~/.mcporter/mcporter.json`

It can also merge imports from supported editor/tool configs.

This is one of the repo's strongest practical features.

## Best Fit

MCPorter is a strong fit when:

- your stack is TypeScript or Bun/Node
- you already use several MCP-enabled tools
- you want to reuse MCP tools in scripts/tests/agents
- you want a fast way to wrap an MCP server as a CLI

## Less Ideal Fit

It is less central if:

- your stack is mostly Python
- you want a full agent runtime rather than a tooling layer
- you only need one MCP client and are happy using the raw SDK

## Relationship To OpenClaw / Agent Systems

For OpenClaw-style ecosystem thinking, MCPorter is not an agent framework. It is closer to:

- an MCP client utility layer
- a codegen wrapper
- a local integration bridge

That makes it complementary to agent frameworks rather than competitive with them.

An agent system could use MCPorter-generated clients or CLIs as part of its tool surface.

## Bottom Line

MCPorter solves **MCP usability in TypeScript**.

Its core contribution is:

- unify scattered MCP configs
- make MCP tools easy to inspect and call
- generate typed wrappers
- package useful subsets as CLIs

If you work heavily with MCP in Node/Bun, it is a practical repo worth learning from even if you do not adopt its exact runtime.

## Source Pointers

- `README.md` - high-level capabilities and quick start
- `docs/mcp.md` - runtime and command overview
- `docs/import.md` - how config import/discovery works
- `docs/emit-ts.md` - typed client generation
- `docs/cli-generator.md` - dedicated CLI generation
