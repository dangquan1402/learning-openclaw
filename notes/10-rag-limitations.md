# RAG Tools: Limitations, Problems, and Should You Build Your Own?

> **Parent:** [Chat-with-Documents Landscape](09-chat-with-docs.md)

## TL;DR

Every open-source RAG tool has **serious quality or reliability issues** in production. For small document sets (<500 pages), **skip RAG entirely** — use context stuffing or agentic search. For large corpora, expect significant engineering time regardless of tool. RAG is a **systems engineering problem**, not a plug-and-play feature.

## Common RAG Failures (All Tools)

### 1. Hallucination Despite Retrieval

- Stanford 2025 study: legal AI tools hallucinate **17-33%** of the time despite "hallucination-free" claims
- Even when retrieval works, the LLM's parametric knowledge can **override** retrieved content
- **Unnecessary retrieval can introduce NEW hallucinations** — irrelevant context can be worse than none

### 2. Poor Chunking = Lost Context

- Fixed-size chunking **fragments coherent content** across boundaries
- Tables spanning two pages get **severed** — "perfectly formatted, completely accurate, yet utterly meaningless"
- Hierarchical structure (nested sections, procedures) **destroyed** during chunking
- Headers separated from data cells lose all context

### 3. Retrieval Misses

- Good at keyword matching, bad at **abstract concept retrieval**
- **Embedding drift**: gradual quality degradation invisible to monitoring — teams blame user behavior, not technical decay
- **Data freshness rot**: "no error spike, no latency blip, no alert. Just progressively less accurate"

### 4. Multi-Document Reasoning Failure

- Increasing document count **reduces LLM performance by up to 20%** even when controlling for context length (2025 paper)
- Systems retrieve all necessary facts but **fail at multi-hop inference** — can't connect info across documents

### 5. Tables, Charts, Images

- Text-only extraction **completely ignores** visual elements
- Tables exceed chunk limits; split tables lose row/column relationships
- Vision-guided chunking exists but adds significant complexity and cost

## Specific Tool Problems (Real GitHub Issues)

### Dify (134K stars)
- Knowledge base **breaks at 5,000+ files** — server crashes
- Default relevance algorithms have **limited accuracy** for specialized domains
- Cloud version has unreasonably low size limits
- Support described as "read the docs" even for paid users
- Multiple security CVEs (2025-55182, 55184, 67779)

### Open WebUI (128K stars)
- **RAG is very basic** with few capabilities
- v0.5.15-20: "very bad results, sometimes almost completely fabricated" (database scoring bugs)
- **Hybrid Search still broken** as of July 2025 (Issue #15915)
- top_k parameter **ignored** in v0.6.33 — retrieves ALL files instead of relevant ones (Issue #18173)
- Markdown Header Splitter creates tiny useless fragments

### RAGFlow (76K stars)
- **Extremely resource-hungry**: simple "hi" took **1 minute** on 64GB RAM / 8 CPUs (Issue #2065)
- Requires managing Redis + Elasticsearch + MySQL + MinIO
- Document parsing can queue indefinitely
- Graph features, search, and RAPTOR all reported as very slow

### PrivateGPT (57K stars)
- **Painfully slow**: <4 words/second with Mistral 7b on $300+/month servers
- Response times can exceed **600 seconds** (Issue #316 — most upvoted)
- CPU-only by default
- Privacy advantage comes at extreme performance cost

### AnythingLLM (54K stars)
- v1.8.2 produced **22 vectors** for a doc that had 2,960 in v1.8.1 (Issue #4030)
- Significant information **missing during embedding** (Issue #4556)
- **No reranker or advanced retrieval** — RAG is "not very sophisticated"
- Ollama embedding service frequently fails

## The "Too Many Tools" Problem

### Why Do So Many Exist?

Each optimizes for a different axis:

| Tool | Primary Axis |
|------|-------------|
| Dify | Workflow builder for non-devs |
| Open WebUI | ChatGPT-like UI for local models |
| RAGFlow | Best document parsing quality |
| PrivateGPT | Privacy-first, fully offline |
| AnythingLLM | Easiest setup |
| Onyx | Enterprise SaaS connectors |

### Are They All the Same?

At the core, **yes**: chunk → embed → store → retrieve → LLM. Differentiation is in UI polish, setup complexity, privacy guarantees, and production features.

### Are Any Production-Ready?

**Honestly, no** — not without significant engineering:
- Open WebUI has version-breaking regressions
- RAGFlow requires massive resources for basic ops
- AnythingLLM has embedding reliability issues
- PrivateGPT is too slow for real-time use
- Dify's knowledge base breaks at scale

## Should You Build Your Own Lightweight Version?

### A Minimal RAG System (~40-500 Lines)

```
1. Document loader    → PDF/text parser (pymupdf, unstructured)
2. Chunker            → Recursive text splitter (by headings/paragraphs)
3. Embedder           → OpenAI/Cohere/local model API call
4. Vector store       → ChromaDB, or just numpy cosine similarity
5. Retriever          → Top-k similarity search
6. LLM call           → Send retrieved chunks as context
```

That's it. The core of every RAG tool in 6 steps.

### When to Build Your Own

| Build Your Own | Use Existing Tool |
|---------------|-------------------|
| Narrow use case (one doc type, one domain) | Need polished UI for non-technical users |
| Need full control over chunking strategy | Need multi-user/multi-tenant |
| Want to avoid Redis + ES + MySQL + MinIO | Need enterprise connectors (Slack, Confluence) |
| Can iterate quickly on retrieval quality | Don't have engineering resources to maintain |
| Small document set | Large-scale deployment |

### The "Just Stuff It In Context" Alternative

**This is increasingly the winning approach for small-to-medium document sets:**

| Approach | When It Works |
|----------|--------------|
| **Context stuffing** | <500 pages (~200K tokens). Just send the docs directly to the LLM |
| **Agentic search** (grep/read) | Codebases, structured docs. Claude Code uses this — "outperformed RAG. By a lot." |
| **RAG** | 10K+ documents, real-time data, cost-sensitive at scale |

Anthropic tried RAG internally for Claude Code but switched to **agentic search** because it was superior.

## The "RAG Is Dead" Debate

### Arguments Against RAG

- Claude: 1M tokens. Gemini: 2M tokens. You can fit entire books.
- Newer models have **mostly eliminated "lost in the middle"** positional bias
- Long-context models match or beat RAG on most benchmarks when data fits

### Arguments For RAG

| Factor | Long Context | RAG |
|--------|-------------|-----|
| **Tokens per query** | 1M (everything) | 2K-10K (relevant only) |
| **Cost** | High (pay for all tokens) | **50-200x cheaper** per query |
| **Latency** | 20-30+ seconds (1M tokens) | 100-500ms added |
| **Accuracy** | 90% retrieval (1 in 10 wrong) | Depends on pipeline quality |
| **Real-time data** | Snapshot only | Can query live databases |
| **Scale** | Capped at context window | Unlimited corpus |

### The 2026 Consensus: Hybrid

**Use RAG to find relevant context, then use long context to reason across it.**

```
Large corpus → RAG retrieval (narrow to relevant docs) → Long context window (reason across them)
```

This is what "agentic RAG" / "context engineering" means in practice.

### Cost Reality

- RAG infrastructure costs **2-3x higher than teams expect** (vector storage + embedding + reranking + engineering + compliance)
- But 1M token API calls at scale are also expensive
- For <500 pages: context stuffing is simpler AND cheaper
- For 10M+ docs: RAG is the only viable option

## The Honest Bottom Line

```
Small docs (<500 pages)  → Skip RAG. Context stuff or agentic search.
Medium docs (500-10K)    → Lightweight custom RAG or Kotaemon/AnythingLLM
Large corpus (10K+)      → RAGFlow or Dify (expect engineering work)
Enterprise (SaaS tools)  → Onyx (40+ connectors)
Privacy-critical         → PrivateGPT (accept the speed cost)
```

**The dirty secret:** Most teams that deploy RAG spend more time fixing retrieval quality than building features. The tools handle the easy 80% — the remaining 20% (your specific document types, your edge cases, your quality bar) is where all the work lives.

> *"RAG is a systems engineering problem, not a plug-and-play feature. No tool makes it easy."*

## Key Sources

- [Stanford Legal RAG Hallucinations Study (2025)](https://dho.stanford.edu/wp-content/uploads/Legal_RAG_Hallucinations.pdf)
- [Why Claude Code Uses Agentic Search Not RAG](https://zerofilter.medium.com/why-claude-code-is-special-for-not-doing-rag-vector-search-agent-search-tool-calling-versus-41b9a6c0f4d9)
- [1M Token Context vs RAG - MindStudio](https://www.mindstudio.ai/blog/1m-token-context-window-vs-rag-claude)
- [Ten Failure Modes of RAG - DEV](https://dev.to/kuldeep_paul/ten-failure-modes-of-rag-nobody-talks-about-and-how-to-detect-them-systematically-7i4)
- [Five Silent Killers of Production RAG](https://www.analyticsvidhya.com/blog/2025/07/silent-killers-of-production-rag/)
- [Multi-Document RAG Challenges - arxiv](https://arxiv.org/abs/2503.04388)
- [From RAG to Context - RAGFlow Review](https://ragflow.io/blog/rag-review-2025-from-rag-to-context)
- [Open WebUI RAG Broken #11388](https://github.com/open-webui/open-webui/discussions/11388)
- [AnythingLLM Embedding Issues #4030](https://github.com/Mintplex-Labs/anything-llm/issues/4030)
- [RAGFlow Slow #2065](https://github.com/infiniflow/ragflow/issues/2065)
- [PrivateGPT Slow #316](https://github.com/imartinez/privateGPT/issues/316)
- [HN: RAG is a Hack](https://news.ycombinator.com/item?id=37792515)
