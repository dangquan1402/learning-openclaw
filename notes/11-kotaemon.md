# Kotaemon: Deep Investigation

> **Parent:** [Chat-with-Documents Landscape](09-chat-with-docs.md)
> **Repo:** [Cinnamon/kotaemon](https://github.com/Cinnamon/kotaemon) | **License:** Apache 2.0

## What Is It?

An open-source RAG application for chatting with documents — both a **ready-to-use app** and a **developer framework** for custom RAG pipelines. Made by Cinnamon AI (Tokyo, Japan; R&D in Vietnam; $39M Series C).

| Metric | Value |
|--------|-------|
| Stars | 25,237 |
| Forks | 2,115 |
| Contributors | 30 |
| Created | March 2024 |
| Latest | v0.11.2 (Mar 2026) |
| Language | Python 3.10+ |
| UI | Gradio |
| Listed on | ThoughtWorks Technology Radar ("Assess" ring) |

## Architecture (4 Layers)

```
1. PRESENTATION     Gradio web UI, custom theme, PDF.js viewer
       │
2. APPLICATION      BaseApp orchestrator, pluggy plugin system, pub-sub events
       │
3. SERVICE          Index management, LLM coordination, embeddings, reasoning pipelines
       │
4. DATA             SQLite (users/conversations), vector stores (Chroma/LanceDB), file storage
```

**Two-library design:**
- `libs/kotaemon` — core framework (LLMs, embeddings, vector stores, retrievers)
- `libs/ktem` — application layer (Gradio UI, index management, user management)

## RAG Pipeline

### Indexing

```
Document → Parse (Azure/Adobe/Docling/Unstructured)
         → Chunk (recursive character splitting, configurable size/overlap)
         → Embed (OpenAI/Cohere/local)
         → Store in vector store + document store
```

### Retrieval (Hybrid Search)

```
Query → [Full-text search (BM25/keyword)] + [Vector search (semantic)]
         → Merge results
         → Re-rank (Cohere Rerank or TEI Fast Rerank)
         → Top-k results to LLM
```

### 5 Reasoning Pipelines

| Pipeline | How It Works |
|----------|-------------|
| **Simple/Full QA** | Standard retrieve → generate |
| **Question Decomposition** | Break complex question into sub-questions → retrieve each → synthesize |
| **ReAct Agent** | Reason-and-act loop with tool use |
| **ReWOO Agent** | Plan all steps upfront, then execute |
| **Multi-modal QA** | Handle figures, tables alongside text |

## The Standout Feature: PDF Citation Highlighting

**How it works:**
1. LLM generates answer + structured citation JSON (`"evidences"` with source passages)
2. CitationPipeline matches evidence texts against retrieved chunks
3. Matched passages **highlighted directly in in-browser PDF viewer** (PDF.js)
4. Each citation includes a **relevance score**

**Result:** See the AI's answer alongside the original PDF with highlighted source sentences — immediate hallucination detection.

**Limitation:** Requires strong instruction-following LLMs. With small models (8B params), citation works **<10% of the time** (Issue #541). Works well with GPT-4, Claude, etc.

## Document Parsing

### Supported Formats

- **Native:** PDF, HTML, MHTML, XLSX
- **Extended (via Unstructured):** DOC, DOCX, and many more

### Parsing Backends (Selectable in UI)

| Backend | Type | Capabilities |
|---------|------|-------------|
| **Azure Document Intelligence** | Cloud API | OCR, tables, figures, layout analysis |
| **Adobe PDF Extract** | Cloud API | Advanced structure preservation |
| **Docling** | Local, open-source | No API dependency |
| **Unstructured** | Local | Broad format support |

### Multi-Modal Support

- **Tables** — extracted and indexed separately
- **Figures/images** — extracted for multi-modal QA
- **Charts** — handled through document intelligence backends

## LLM Support

| Type | Options |
|------|---------|
| **Cloud** | OpenAI, Azure OpenAI, Cohere, Groq, any OpenAI-compatible |
| **Local** | Ollama, llama-cpp-python (direct GGUF loading) |
| **Lightweight** | Qwen1.5-1.8B-Chat-GGUF for resource-constrained setups |

Multiple LLMs and embedding models can be configured simultaneously via UI.

## Vector DB Support

| Backend | Role | Notes |
|---------|------|-------|
| **ChromaDB** | Default vector store | Good for small-medium |
| **LanceDB** | Default document store | Also available as vector store |
| **Milvus** | Enterprise vector store | Zilliz integration documented |
| **Qdrant** | Vector store | Cloud and self-hosted |
| **Elasticsearch** | Document store | Full-text search backend |
| **In-memory** | Testing | Small datasets only |

## GraphRAG (3 Implementations)

| Implementation | Config | Notes |
|---------------|--------|-------|
| **NanoGraphRAG** | `USE_NANO_GRAPHRAG=true` | Recommended for Kotaemon |
| **LightRAG** | `USE_LIGHTRAG=true` | Uses default LLM/embedding |
| **Microsoft GraphRAG** | `USE_MS_GRAPHRAG=true` | Only works with OpenAI or Ollama |

Enables relationship-aware retrieval across multiple documents. **Currently the buggiest area** — KeyErrors, write failures, cross-document search issues.

## Multi-User

- Login system with default credentials (admin/admin)
- User management via admin UI
- **Private and public collections** with access control
- Chat sharing between users
- **SSO support** (`KH_SSO_ENABLED` flag)
- Demo mode for public instances
- Per-user data isolation via `user_id` in SQLite

## Installation

### Docker (Recommended)

Three image variants:
- **Lite** — minimal dependencies
- **Full** — all features
- **Ollama-bundled** — fully local (no internet needed)

Platforms: linux/amd64, linux/arm64

### Without Docker

```bash
# Option 1: uv (faster)
uv venv && uv pip install -e "libs/kotaemon[all]" -e "libs/ktem"

# Option 2: Conda
conda create -n kotaemon python=3.10 && pip install -e "libs/kotaemon[all]" -e "libs/ktem"
```

### Requirements

- No GPU required (CPU-only works with API-based LLMs)
- 16 GB RAM recommended for local 7B-8B models
- Python 3.10+
- PDF.js viewer setup needed separately for citation highlighting

## Limitations (Real Problems)

### From GitHub Issues (225 open)

1. **GraphRAG instability** — most bugs are here: KeyErrors, NanoGraphRAG write failures, can't search across document groups (Issue #548)
2. **Citation fails with small LLMs** — <10% success rate with 8B models (Issue #541)
3. **MS GraphRAG locked to OpenAI/Ollama** — no other LLM providers for graph indexing
4. **Pydantic version conflicts** — ChromaDB requires <2.0, conflicts with other packages
5. **225 open issues** — significant bug backlog
6. **SQLite backend** — no horizontal scaling, not for large enterprise
7. **Gradio UI constraints** — less flexible than React/Vue frontends
8. **Documentation gaps** — Quick Start exists, deep technical docs sparse
9. **Release cadence gaps** — 7-month gap between some releases

### Honest Assessment

- **Works great for:** Personal/team document QA with cloud LLMs (GPT-4, Claude)
- **Struggles with:** Large-scale enterprise, small local LLMs, GraphRAG stability
- **Not designed for:** Multi-tenant SaaS, horizontal scaling, real-time high-throughput

## Extensibility

| Extension Point | How |
|----------------|-----|
| Custom reasoning pipelines | Add in `libs/ktem/ktem/reasoning/`, register in `KH_REASONINGS` |
| Custom indexing pipelines | Add in `libs/ktem/ktem/index/`, register in `KH_INDEX_TYPES` |
| Custom LLM/embedding providers | Register via `KH_LLMS` / `KH_EMBEDDINGS` |
| Custom storage backends | Override `KH_VECTORSTORE` / `KH_DOCSTORE` |
| Plugins | `pluggy` framework, declare via `ktem_declare_extensions()` |
| MCP tools | Supported since v0.11.2 |

Data directory (`ktem_app_data`) is **fully portable** — move between installations.

## Comparison with Competitors

| Feature | Kotaemon | AnythingLLM | Open WebUI | RAGFlow |
|---------|----------|-------------|------------|---------|
| **Focus** | Document QA with verification | All-in-one AI desktop | LLM chat UI | Enterprise RAG engine |
| **Stars** | 25K | 54K | 128K | 76K |
| **RAG quality** | High (hybrid + rerank + GraphRAG) | Medium (basic) | Low (basic, buggy) | Very High (DeepDoc) |
| **PDF citations** | **Yes (standout)** | No | No | Partial |
| **Multi-modal** | Yes (tables, figures) | Limited | No | Yes (best parsing) |
| **GraphRAG** | 3 implementations | No | No | Yes |
| **Agent reasoning** | ReAct + ReWOO | Yes | Limited | Yes |
| **Ease of setup** | Moderate | **Easiest** | Moderate | Complex |
| **Resource needs** | Low-moderate | Low | Low-moderate | **Very high** |
| **Best for** | Doc QA + source verification | General AI desktop | ChatGPT replacement | Complex doc processing at scale |

## Unique Features (No Other Tool Has)

1. **In-browser PDF citation highlighting with relevance scores** — see exactly which sentences were used
2. **3 GraphRAG implementations** toggleable in one tool
3. **Question decomposition pipeline** — built-in multi-hop reasoning
4. **ReAct + ReWOO agents** as selectable reasoning modes
5. **Multiple parsing backends selectable per document** from the UI
6. **MCP support** (v0.11.2) for tool integration
7. **Portable data directory** — move `ktem_app_data` between installations

## When to Choose Kotaemon

| Choose Kotaemon When | Choose Something Else When |
|---------------------|---------------------------|
| You need **citation verification** (PDF highlighting) | You need enterprise scale (→ RAGFlow or Dify) |
| You want hybrid search + re-ranking + GraphRAG | You want zero-config setup (→ AnythingLLM) |
| You're using GPT-4/Claude (citation works best) | You need 40+ SaaS connectors (→ Onyx) |
| You want extensible Python framework | You need polished ChatGPT-style UI (→ Open WebUI) |
| Document QA is your primary use case | RAG is just one feature among many (→ Dify) |
| Team of <50 users | You need multi-tenant SaaS |

## Key Sources

- [Kotaemon GitHub](https://github.com/Cinnamon/kotaemon)
- [Kotaemon Docs](https://cinnamon.github.io/kotaemon/)
- [DeepWiki Architecture Analysis](https://deepwiki.com/Cinnamon/kotaemon)
- [ThoughtWorks Radar](https://www.thoughtworks.com/radar/languages-and-frameworks/kotaemon)
- [Skywork AI Guide](https://skywork.ai/blog/kotaemon-comprehensive-guide-2025-everything-you-need-to-know/)
- [Cinnamon AI](https://cinnamon.ai/en/company/)
- [Citation Issue #541](https://github.com/Cinnamon/kotaemon/issues/541)
- [GraphRAG Issue #548](https://github.com/Cinnamon/kotaemon/issues/548)
