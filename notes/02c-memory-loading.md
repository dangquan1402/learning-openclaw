# Deep Dive: Efficient Memory Loading Strategies

> **Parent:** [Architecture](02-architecture.md) | **User Story:** US2.3

## How OpenClaw Selectively Loads Memory

OpenClaw does **NOT** load all memory files into context. It uses targeted loading patterns:

| Loading Pattern | What | When |
|-----------------|------|------|
| **MEMORY.md** | First 200 lines only | Every session start |
| **Daily logs** | Today + yesterday | Every session start |
| **Topic files** | Referenced content only | On-demand (via file tools) |
| **memory_search** | Relevant snippets | During reasoning (tool call) |
| **Subdirectory files** | Only when accessed | Lazy/on-demand |

**Key insight:** A 50-page memory document does NOT get dumped into context. `memory_search` returns relevant snippets (~400 token chunks), not entire files.

## Token Budget Management

The context window is divided between competing consumers:

```
┌─────────────────────────────────────────┐
│           Context Window                │
├─────────────────────────────────────────┤
│  System Prompt (base + bootstrap files) │
│  ─────────────────────────────────────  │
│  MEMORY.md (first 200 lines)            │
│  ─────────────────────────────────────  │
│  Daily Logs (today + yesterday)         │
│  ─────────────────────────────────────  │
│  Skills (compact list, full on-demand)  │
│  ─────────────────────────────────────  │
│  Conversation History                   │
│  ─────────────────────────────────────  │
│  memory_search results (as needed)      │
│  ─────────────────────────────────────  │
│  Reserved Buffer (~16.5% of window)     │
└─────────────────────────────────────────┘
```

### Key Numbers

| Metric | Value |
|--------|-------|
| Reserved buffer | ~33,000 tokens (16.5%) for 200K window |
| Auto-compaction trigger | ~83.5% of total context usage |
| Compaction override | `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` env var (1-100) |
| Thinking blocks | Auto-stripped from previous turns to save space |

For 1M context windows (Opus 4.6 / Sonnet 4.6): same percentages, proportionally larger absolute numbers.

## The Compaction/Flush Cycle

### Formula

```
flush_threshold = contextWindowTokens - reserveTokensFloor - softThresholdTokens
```

### Defaults and Tuning

| Parameter | Default | Recommended for Large Windows |
|-----------|---------|-------------------------------|
| `reserveTokensFloor` | 20,000 tokens | Keep default |
| `softThresholdTokens` | 4,000 tokens | Increase to **8,000 tokens** |

### Cycle Steps

```
1. Token count approaches threshold
   ↓
2. Silent agentic turn (NO_REPLY — user sees nothing)
   ↓
3. Model writes durable info to MEMORY.md / daily logs
   ↓
4. Auto-compaction summarizes conversation
   ↓
5. Session continues with compressed context
```

One flush per compaction cycle (tracked in `sessions.json`).

## Memory File Organization — What Works Best

### Recommended Structure (Hub-and-Spoke)

```
MEMORY.md                    # Hub: <200 lines, durable rules + pointers
├── memory/2026-03-24.md     # Daily: today's context
├── memory/2026-03-23.md     # Daily: yesterday's context
├── debugging.md             # Spoke: debugging patterns
├── architecture.md          # Spoke: architecture decisions
├── patterns.md              # Spoke: code patterns
└── contacts.md              # Spoke: people directory
```

### Why Many Small Files > Few Large Files

| Aspect | Many Small Files | Few Large Files |
|--------|-----------------|-----------------|
| **On-demand loading** | Only load what's needed | Loads irrelevant content |
| **Search quality** | Better semantic boundaries | Chunks may split across topics |
| **Deduplication** | SHA-256 per chunk, less waste | More duplicate content |
| **MEMORY.md limit** | Stays under 200-line threshold | Degrades past 200 lines |
| **Git diffs** | Cleaner, topic-focused changes | Large, mixed diffs |

### What Goes Where

| Content | Where | Why |
|---------|-------|-----|
| Durable rules, key decisions | `MEMORY.md` | Loaded every session |
| Today's work context | `memory/YYYY-MM-DD.md` | Auto-loaded, discarded after 2 days |
| Deep reference material | Topic files | Read on-demand, not wasted on irrelevant turns |
| User preferences | `preferences.md` | On-demand, stable content |
| Temporary debugging notes | Daily log | Expires naturally |

## Optimizing memory_search Retrieval

### How the Vector Index Works

1. **Chunking:** Markdown → ~400-token chunks with 80-token overlap, following heading boundaries
2. **Embedding:** Chunks embedded via configured provider (OpenAI, Gemini, Ollama, etc.)
3. **Storage:** sqlite-vec in SQLite. Auto-reindexes on config changes
4. **Search:** Dual strategy — vector (70%) + BM25 (30%) in parallel
5. **Ranking:** Reciprocal Rank Fusion → MMR diversity → optional temporal decay

### Tips for Better Retrieval

1. **Use clear headings** — chunking follows Markdown structure
2. **One topic per section** — prevents cross-topic contamination in chunks
3. **Include keywords** — BM25 (30% of scoring) relies on exact term matches
4. **Short, descriptive filenames** — helps with file-level filtering
5. **Avoid very long paragraphs** — they cross chunk boundaries
6. **Remove stale content** — reduces noise in search results

## Monitoring and Debugging

### Available Tools

| Tool | Purpose |
|------|---------|
| `/context` command | Show current context data and token usage |
| Custom status bar | Always-visible context window usage |
| `sessions.json` | Track flush cycles and compaction events |

### Current Limitations

- **No self-inspection** — the agent cannot see its own context window consumption during a session (GitHub issue #34879)
- **Pruning is in-memory only** — old tool results trimmed per-request, disk session history untouched
- Context usage is effectively opaque profiling data

## Performance Benchmarks

### LongMemEval (100-question snapshot)

| Metric | Score |
|--------|-------|
| hit@k | 0.92 |
| MRR | 0.8695 |
| nDCG@k | 0.8352 |
| p50 latency | ~150ms |
| Failed queries | 0/100 |

### MemMachine Plugin (WikiMultiHop)

- **+44.3 percentage points** accuracy improvement (2.2x over vanilla)
- **59% token reduction** (103K → 42K tokens)

## Key Questions Answered

**Q: How does the vector index get built?**
Markdown files are chunked (~400 tokens, 80 overlap), embedded via configured provider, stored in SQLite with sqlite-vec. Auto-reindexes on config changes.

**Q: Optimal MEMORY.md structure?**
Under 200 lines. Durable rules and pointers to topic files. One topic per section with clear headings.

**Q: How much context does memory consume?**
Varies. MEMORY.md (200 lines) + 2 daily logs + memory_search results. Typically 10-20% of context window.

**Q: Daily logs vs MEMORY.md vs workspace files?**
Daily logs = transient context (2-day auto-load). MEMORY.md = durable rules (every session). Workspace files = deep reference (on-demand).

**Q: Sandboxed environments?**
Memory flush is skipped if workspace is read-only. Env vars don't inherit into sandbox containers.

## Community Tips for Long-Running Agents

1. **Enable memory flush** — verify buffer is large enough before compaction triggers
2. **Put durable rules in files, not chat** — chat instructions vanish on compaction
3. **Keep MEMORY.md under 200 lines** — move details to topic files
4. **Use `memory_search`** instead of reading entire directories
5. **Increase `softThresholdTokens` to 8,000** for large context windows
6. **Consider Mnemon plugin** for cross-session knowledge surviving compaction
7. **Consider Mem0 plugin** for system-layer memory capture/recall
8. **Tiered hibernation** — keep costs sane, monitor memory, recycle aggressively
9. **Modular `.claude/rules/` segments** — reduces hallucinations ~40% per SFEIR findings
10. **"Markdown files beat $50M vector databases"** — well-structured text + BM25 often outperforms expensive infrastructure

## Key Docs & Sources

- [Memory - OpenClaw Docs](https://docs.openclaw.ai/concepts/memory)
- [Agent Loop - OpenClaw Docs](https://docs.openclaw.ai/concepts/agent-loop)
- [QMD Memory - Hybrid Search](https://openclaw-setup.me/blog/openclaw-internals/qmd-memory/)
- [Memory Search - DeepWiki](https://deepwiki.com/openclaw/openclaw/7.3-memory-search)
- [How to Optimize OpenClaw Memory](https://medium.com/@creativeaininja/how-to-optimize-openclaw-memory-concurrency-and-context-that-actually-works-84690c2de3d7)
- [OpenClaw Memory Masterclass](https://velvetshark.com/openclaw-memory-masterclass)
- [memsearch - GitHub](https://github.com/zilliztech/memsearch)
- [Context Buffer Management](https://claudefa.st/blog/guide/mechanics/context-buffer-management)
