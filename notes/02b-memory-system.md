# Deep Dive: Memory System Architecture

> **Parent:** [Architecture](02-architecture.md) | **User Story:** US2.2

## Two-Tier Architecture

OpenClaw's memory is **plain Markdown files on disk**. No hidden state -- the model only "remembers" what's written to files.

### Tier 1: Daily Logs

| Aspect | Details |
|--------|---------|
| **Location** | `memory/YYYY-MM-DD.md` |
| **Purpose** | Running context, day-to-day observations |
| **Loading** | Today's + yesterday's logs auto-loaded at session start |
| **Format** | Append-only daily entries |
| **Best for** | Transient context, conversation summaries, work-in-progress notes |

### Tier 2: Long-Term Memory

| Aspect | Details |
|--------|---------|
| **Location** | `MEMORY.md` at workspace root |
| **Purpose** | Durable facts, decisions, preferences, rules |
| **Loading** | First **200 lines** loaded every session. Content beyond line 200 is NOT loaded |
| **Priority** | If both `MEMORY.md` and `memory.md` exist, only `MEMORY.md` is loaded |
| **Best for** | Architecture decisions, user preferences, key rules, project conventions |

### Additional Workspace Files

| File | Purpose |
|------|---------|
| `preferences.md` | User habits and preferences |
| `contacts.md` | People the agent has learned about |
| `projects.md` | Active projects and context |
| `learnings.md` | Things the agent has figured out |

These files are **NOT auto-loaded**. The agent reads them on-demand using file tools when needed.

## Default Workspace Layout

```
~/.openclaw/
  ├── memory/
  │   ├── 2026-03-24.md       # Today's daily log (auto-loaded)
  │   ├── 2026-03-23.md       # Yesterday's daily log (auto-loaded)
  │   └── ...                  # Older logs (searchable, not auto-loaded)
  ├── MEMORY.md                # Long-term memory (first 200 lines auto-loaded)
  ├── preferences.md           # On-demand
  ├── contacts.md              # On-demand
  ├── projects.md              # On-demand
  └── learnings.md             # On-demand
```

## Memory Tools

### memory_search (Semantic Recall)

Runs semantic search across all memory files, returning **relevant snippets, not entire files**.

**How it works (QMD — Query-Memory-Document):**

```
Query → [Vector Search (70%)] + [BM25 Keyword Search (30%)] → Reciprocal Rank Fusion → MMR Re-ranking → Results
```

**Chunking:**
- Markdown files split into ~**400-token chunks** with **80-token overlap**
- Follows semantic boundaries (headings and their bodies stay together)
- **SHA-256 deduplication**: identical chunks embedded only once

**Dual Search Strategy (run in parallel):**
- **Vector search (70% weight)**: cosine similarity in shared embedding space. "Redis cache config" finds "Redis L1 cache with 5min TTL"
- **BM25 keyword search (30% weight)**: exact term frequency scoring. "PostgreSQL 16" won't return "PostgreSQL 15"

**Supported embedding providers:** OpenAI, Gemini, Voyage, Mistral, Ollama, local GGUF models

**Storage:** sqlite-vec extension in SQLite. Auto-reindexes if embedding provider/model/chunking params change.

**Post-processing:**
1. Reciprocal Rank Fusion (merge vector + BM25 results)
2. MMR (Maximal Marginal Relevance, lambda=0.7) for diversity
3. Optional temporal decay
4. Optional LLM re-ranking pass

**Default mode:** BM25 keyword-only (fast, sub-second, no ML models). Semantic search is opt-in.

### memory_get (Targeted Read)

Direct file read with optional line range. Used when the agent knows exactly which file contains the needed info.

## Memory Flush (Pre-Compaction)

When a session approaches context limits, OpenClaw triggers a **silent agentic turn** to persist important info before compaction.

### Flush Threshold Formula

```
threshold = contextWindowTokens - reserveTokensFloor - softThresholdTokens
```

| Parameter | Default | Purpose |
|-----------|---------|---------|
| `reserveTokensFloor` | 20,000 tokens | Safety buffer for background operations |
| `softThresholdTokens` | 4,000 tokens | Triggers flush when token estimate crosses threshold |

### Cycle Behavior

1. Session token count approaches threshold
2. OpenClaw triggers **silent turn** with `NO_REPLY` (nothing sent to user)
3. Model is reminded to write durable memory to disk
4. **One flush per compaction cycle** (tracked in `sessions.json`)
5. Auto-compaction summarizes conversation, creates compaction block, continues with compressed context

> **Known issue:** Default `softThresholdTokens` of 4,000 was tuned for ~200K context windows. Community recommends increasing to **8,000 tokens** for larger windows.

## How Memory Interacts with the Agent Loop

During **Context Assembly** (step 1 of agent loop):

1. Load bootstrap files (`AGENTS.md`, `SOUL.md`, `TOOLS.md`)
2. Load first 200 lines of `MEMORY.md`
3. Load today's + yesterday's daily logs
4. Run `memory_search` for semantically similar past context
5. Inject relevant skill descriptions
6. Build final system prompt

The agent can then use `memory_search` and `memory_get` as tools during reasoning to access additional context.

## Manually Editing Memory

All memory files are plain Markdown — you can edit them with any text editor:

- **Edit MEMORY.md** to add/remove durable rules
- **Edit daily logs** to correct or add context
- **Delete old daily logs** to free up search space
- Changes take effect on **next session start** (or next memory_search call)

## Memory in Sandboxed vs Non-Sandboxed

| Aspect | Non-Sandboxed | Sandboxed |
|--------|---------------|-----------|
| Memory access | Full read/write to workspace | Depends on workspace mount config |
| Memory flush | Normal operation | Skipped if workspace is read-only |
| memory_search | Full access to all files | Limited to mounted files |
| Environment vars | Full access | Do NOT inherit from host |

## Comparison with Other Memory Approaches

| Approach | Pros | Cons |
|----------|------|------|
| **Markdown files** (OpenClaw) | Human-readable, Git-versionable, zero infrastructure, cached tokens 10x cheaper | No built-in semantic search (BM25 default), manual organization |
| **Vector DB RAG** | Semantic matching across large corpora | Misses architectural patterns, infrastructure overhead |
| **Knowledge Graphs** | Multi-hop reasoning, factual accuracy | Complex pipelines, schema maintenance |
| **Hybrid RAG** | Best of both worlds | Implementation complexity |

> **2026 consensus:** Well-structured Markdown files with BM25 search outperform expensive vector DB infrastructure for most agent use cases.

## Related Projects

- **memsearch** ([github.com/zilliztech/memsearch](https://github.com/zilliztech/memsearch)) — Markdown-first memory indexing inspired by OpenClaw
- **ClawMem** — On-device context engine with multi-signal retrieval
- **Mnemon** — LLM-supervised persistent memory plugin
- **Mem0** — System-layer memory capture/recall (outside agent session)
- **MemMachine** — +44.3% accuracy improvement on WikiMultiHop benchmark

## Key Docs & Sources

- [Memory - OpenClaw Docs](https://docs.openclaw.ai/concepts/memory)
- [Memory Deep Dive - Study Notes](https://snowan.gitbook.io/study-notes/ai-blogs/openclaw-memory-system-deep-dive)
- [Memory Search - DeepWiki](https://deepwiki.com/openclaw/openclaw/7.3-memory-search)
- [QMD Memory - Hybrid Search](https://openclaw-setup.me/blog/openclaw-internals/qmd-memory/)
- [Memory Flush Issue #31435](https://github.com/openclaw/openclaw/issues/31435)
- [memsearch - GitHub](https://github.com/zilliztech/memsearch)
