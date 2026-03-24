# Trending Chat-with-Documents Projects (March 2026)

> **User Story:** US7

## Landscape Overview

"Chat with documents" = upload PDFs/docs/knowledge bases → ask questions → get answers with citations. Core tech is **RAG (Retrieval-Augmented Generation)**: chunk documents → embed → store in vector DB → retrieve relevant chunks → feed to LLM.

## Top 15 Projects Ranked by Stars

### Tier 1: Mega-Projects (100K+ Stars)

#### 1. Dify — 134K stars
| | |
|---|---|
| **Repo** | [langgenius/dify](https://github.com/langgenius/dify) |
| **Stack** | Python, TypeScript |
| **Type** | Full LLM app platform with visual workflow builder |
| **RAG** | Built-in pipeline: PDF/PPT/DOCX extraction, vector search |
| **Multi-user** | Yes, team workspaces, RBAC |
| **Hosting** | Self-hosted (Docker) or cloud |
| **Vector DB** | Weaviate, Qdrant, Milvus, pgvector, etc. |
| **Best for** | Teams building production AI apps with document chat as one capability |

#### 2. Open WebUI — 128K stars
| | |
|---|---|
| **Repo** | [open-webui/open-webui](https://github.com/open-webui/open-webui) |
| **Stack** | Python (FastAPI), Svelte |
| **Type** | Self-hosted ChatGPT interface with built-in RAG |
| **RAG** | 9 vector DB options; extraction via Apache Tika, Docling, Mistral OCR, Azure Doc Intelligence |
| **Parsing** | PDF, Word, Excel, PowerPoint, images (OCR) |
| **Multi-user** | Yes, admin controls |
| **Hosting** | Self-hosted (Docker), connects to Ollama or OpenAI-compatible |
| **Best for** | Polished ChatGPT replacement with document chat (282M+ downloads) |

### Tier 2: Major Projects (50K-100K Stars)

#### 3. RAGFlow — 76K stars
| | |
|---|---|
| **Repo** | [infiniflow/ragflow](https://github.com/infiniflow/ragflow) |
| **Stack** | Python, TypeScript |
| **Type** | Dedicated RAG engine with deep document understanding |
| **Parsing** | **Best-in-class** — "DeepDoc" module: OCR, layout detection, table structure recognition, figure extraction, auto-rotation, scanned PDF support |
| **Multi-user** | Yes, enterprise-scale (tens of millions of docs) |
| **Vector DB** | Built-in (Elasticsearch + vector search) |
| **Best for** | When document parsing quality is the #1 priority |

#### 4. PrivateGPT — 57K stars
| | |
|---|---|
| **Repo** | [zylon-ai/private-gpt](https://github.com/zylon-ai/private-gpt) |
| **Stack** | Python (FastAPI, LlamaIndex) |
| **Type** | 100% offline document chat — no data leaves your machine |
| **Multi-user** | No (personal privacy tool) |
| **Vector DB** | Qdrant (default) |
| **Best for** | Privacy-conscious individual users |

#### 5. AnythingLLM — 54K stars
| | |
|---|---|
| **Repo** | [Mintplex-Labs/anything-llm](https://github.com/Mintplex-Labs/anything-llm) |
| **Stack** | Node.js, React |
| **Type** | All-in-one AI productivity tool with document chat |
| **RAG** | Workspace-based, drag-and-drop uploads |
| **Multi-user** | Yes |
| **Hosting** | Self-hosted (Docker) or **desktop app** (Mac/Windows/Linux) |
| **Vector DB** | LanceDB (default), Chroma, Pinecone, Weaviate, Qdrant, Milvus |
| **Best for** | Easiest setup — zero config, desktop app for non-technical users |

### Tier 3: Strong Contenders (25K-50K Stars)

#### 6. Flowise — 43K+ stars
| | |
|---|---|
| **Repo** | [FlowiseAI/Flowise](https://github.com/FlowiseAI/Flowise) |
| **Stack** | TypeScript, Node.js |
| **Type** | Drag-and-drop visual RAG pipeline builder |
| **Best for** | Non-coders building custom RAG pipelines visually |

#### 7. Quivr — 38K stars
| | |
|---|---|
| **Repo** | [QuivrHQ/quivr](https://github.com/QuivrHQ/quivr) |
| **Stack** | Python, TypeScript |
| **Type** | "Second Brain" RAG framework |
| **Note** | Pivoted toward enterprise customer support automation |

#### 8. LibreChat — 35K stars
| | |
|---|---|
| **Repo** | [danny-avila/LibreChat](https://github.com/danny-avila/LibreChat) |
| **Stack** | TypeScript (MERN) |
| **Type** | Enhanced ChatGPT clone with multi-provider + RAG |
| **Multi-user** | Yes, RBAC |
| **Best for** | Best ChatGPT alternative UI with document chat as secondary feature |

#### 9. Khoj — 31K stars
| | |
|---|---|
| **Repo** | [khoj-ai/khoj](https://github.com/khoj-ai/khoj) |
| **Stack** | Python |
| **Type** | Personal AI "second brain" with deep research |
| **RAG** | Integrates with Notion, Obsidian, personal files, web |
| **Multi-user** | No (personal) |
| **Best for** | Persistent AI assistant that knows your personal knowledge base |

#### 10. LightRAG — 27K stars
| | |
|---|---|
| **Repo** | [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG) |
| **Stack** | Python |
| **Type** | Graph-based RAG using knowledge graphs |
| **Best for** | Use cases where entity relationships matter |

#### 11. Kotaemon — 25K stars
| | |
|---|---|
| **Repo** | [Cinnamon/kotaemon](https://github.com/Cinnamon/kotaemon) |
| **Stack** | Python (Gradio UI) |
| **Type** | Purpose-built document chat |
| **RAG** | Hybrid (full-text + vector + re-ranking), multi-modal QA |
| **Parsing** | PDF with in-browser preview and **citation highlighting** |
| **Multi-user** | Yes, private/public collections |
| **Best for** | Clean, focused document chat with great citation UX |

### Tier 4: Notable Mentions

#### 12. Haystack — 24K stars
- [deepset-ai/haystack](https://github.com/deepset-ai/haystack) — Enterprise NLP/RAG framework (developer tool, not ready-made app)

#### 13. Onyx (formerly Danswer) — 15K+ stars
- [onyx-dot-app/onyx](https://github.com/onyx-dot-app/onyx) — Enterprise AI search with **40+ SaaS connectors** (Slack, Google Drive, Confluence, Notion, Salesforce). Used by Netflix, Ramp, Thales Group.

#### 14. h2oGPT — 12K stars
- [h2oai/h2ogpt](https://github.com/h2oai/h2ogpt) — Private document chat supporting PDF, images, video

#### 15. Verba — 7K+ stars
- [weaviate/Verba](https://github.com/weaviate/Verba) — RAG chatbot powered by Weaviate vector DB

## Comparison Matrix

| Project | Stars | App? | Multi-User | Doc Parsing | Self-Hosted | Key Differentiator |
|---------|-------|------|------------|-------------|-------------|-------------------|
| **Dify** | 134K | Yes | RBAC | Good | Yes + Cloud | Full platform (RAG + agents + workflows) |
| **Open WebUI** | 128K | Yes | Yes | Very Good | Yes | Most popular self-hosted ChatGPT UI |
| **RAGFlow** | 76K | Yes | Enterprise | **Best** | Yes | DeepDoc parsing engine |
| **PrivateGPT** | 57K | Yes | No | Basic | Yes only | 100% offline privacy |
| **AnythingLLM** | 54K | Yes | Yes | Good | Yes + Desktop | Zero-config desktop app |
| **Flowise** | 43K+ | Builder | Yes | Configurable | Yes | Visual drag-and-drop RAG builder |
| **Quivr** | 38K | Yes | Yes | Good | Yes + Cloud | Second brain / customer support pivot |
| **LibreChat** | 35K | Yes | RBAC | Basic | Yes | Best multi-provider chat UI |
| **Khoj** | 31K | Yes | No | Good | Yes + Cloud | Personal second brain + deep research |
| **LightRAG** | 27K | Framework | — | — | Yes | Knowledge graph-based RAG |
| **Kotaemon** | 25K | Yes | Yes | Very Good | Yes | Citation highlighting in PDF preview |
| **Onyx** | 15K+ | Yes | Enterprise | Good | Yes + Ent. | 40+ SaaS connectors (Slack, GDrive, etc.) |

## Decision Guide

| Use Case | Best Pick | Why |
|----------|-----------|-----|
| **Best document parsing** | RAGFlow | DeepDoc handles scanned PDFs, tables, figures |
| **Personal/privacy** | PrivateGPT or AnythingLLM | Fully offline, your data stays local |
| **Easiest setup** | AnythingLLM | Desktop app, zero config |
| **Enterprise team** | Dify or Onyx | RBAC, connectors, production-grade |
| **ChatGPT replacement** | Open WebUI | Most polished UI, 282M+ downloads |
| **Non-technical users** | AnythingLLM or Flowise | Desktop app / visual builder |
| **SaaS tool integration** | Onyx | 40+ connectors (Slack, Drive, Confluence) |
| **Knowledge graphs** | LightRAG | Entity relationship-based retrieval |
| **Citation quality** | Kotaemon | PDF preview with highlighted citations |
| **Full platform (RAG + agents)** | Dify | Workflows, agents, RAG, model management |

## Retrieval Methods Used

| Method | Projects Using It |
|--------|-------------------|
| **Vector search** (cosine similarity) | All of them |
| **BM25 / full-text** | RAGFlow, Kotaemon, Open WebUI |
| **Hybrid (vector + BM25 + re-ranking)** | Kotaemon, RAGFlow, Open WebUI |
| **Knowledge graphs** | LightRAG |
| **Multi-modal** (images, tables, figures) | RAGFlow, Kotaemon, h2oGPT |

## How This Relates to OpenClaw/GoClaw

OpenClaw and GoClaw are **agent platforms** (chat gateway + tools + memory). Chat-with-documents tools are **RAG platforms** (document ingestion + retrieval + QA). They can complement each other:

- Use **Dify/RAGFlow** as a backend for document knowledge → connect to OpenClaw/GoClaw via MCP or API skills
- Use **OpenClaw/GoClaw** as the chat interface across messaging platforms → call a RAG API for document answers
- GoClaw's built-in **knowledge graph** is a lightweight version of what LightRAG does

## Key Sources

- [ByteByteGo: Top AI GitHub Repositories 2026](https://blog.bytebytego.com/p/top-ai-github-repositories-in-2026)
- [Apidog: 15 Best Open-Source RAG Frameworks 2026](https://apidog.com/blog/best-open-source-rag-frameworks/)
- [Firecrawl: 15 Best Open-Source RAG Frameworks 2026](https://www.firecrawl.dev/blog/best-open-source-rag-frameworks)
- [Meilisearch: 10 Best RAG Tools 2026](https://www.meilisearch.com/blog/rag-tools)
- [Medium: Top 10 RAG Frameworks by Stars Jan 2026](https://florinelchis.medium.com/top-10-rag-frameworks-on-github-by-stars-january-2026-e6edff1e0d91)
