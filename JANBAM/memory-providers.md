# Hermes Agent — Memory Provider Comparison

## Overview

| Provider | Architecture | Key Differentiator | Infrastructure |
|----------|-------------|-------------------|----------------|
| **Honcho** | Hierarchical data model: Workspaces → Peers → Sessions → Messages. Background "Deriver" engine continuously processes messages into summaries and peer representations. | **Peer-centric design** — models both users and agents symmetrically, supporting multi-agent/group chat. Dialectic API lets you query Honcho about any peer's psychology. | Managed cloud or self-hosted server |
| **Mem0** | Memory compression engine with priority scoring, contextual tagging, dynamic forgetting (decay), and consolidation between short/long-term stores. | **Memory lifecycle management** — decides what's worth keeping, decays low-relevance entries, consolidates frequent recalls. SOC 2 + HIPAA compliant. | Managed cloud (YC-backed) or open-source self-hosted |
| **Hindsight** | Biomimetic data structures: facts → entities + relationships + time series with sparse/dense vectors. Three ops: `retain`, `recall`, `reflect`. TEMPR retrieval: semantic + keyword + graph + temporal in parallel. | **Structured fact extraction with evidence tracking** — consolidates facts into observations with provenance, dedup, and freshness trends. State-of-the-art on LongMemEval. | Managed cloud or open-source self-hosted server |
| **Supermemory** | Five-layer context stack: User Profiles, Memory Graph, Retrieval, Extractors, Connectors. | **Context engineering platform** — universal memory API with personalization and continuity. Enterprise integration layer more than a single memory engine. | Managed cloud |
| **OpenViking** | Filesystem paradigm (`viking://`) — memories, resources, skills organized as hierarchical directories. Tiered context: L0 (abstract, <100 tokens), L1 (overview, <2k), L2 (full details). | **Filesystem-as-memory** — context browsable via `ls`, `grep`, `search`. Retrieval paths are traceable. Self-evolving context. | Self-hosted or SaaS; pip-installable |
| **Byterover** | Agent-native: the LLM itself curates knowledge into a Context Tree (Domain → Topic → Subtopic → Entry). All knowledge as markdown files on local filesystem. 5-tier progressive retrieval, sub-100ms for most queries. | **Zero external infrastructure** — no vector DB, no graph DB, no embedding service. Everything is local markdown files. State-of-the-art on LoCoMo. | **Zero** — files on disk |
| **RetainDB** | Multi-backend storage (vector, key-value, graph, full-text). Memory types: preference, context, decision. Async writes, read-after-write consistency. | **Lightweight SaaS with framework integrations** — LangChain, OpenAI Agents SDK. Drop-in cloud memory with minimal config. | Managed cloud |
| **Holographic** | Holographic Reduced Representations (HRR) — facts superposed into fixed-size complex vectors in local SQLite. Algebraic recall via cosine similarity at sub-millisecond latency. Trust scoring. | **Pure local, algebraic memory** — mathematically elegant, extremely fast, zero dependencies beyond SQLite. Facts recalled 3+ times promoted to permanent context. | **Zero** — local SQLite only |

---

## Detailed Descriptions

### Honcho

Honcho is a user context management system built around a hierarchical data model: **Workspaces** (top-level isolation) → **Peers** (users, agents, or any entity) → **Sessions** (conversation threads) → **Messages**. The key design insight is peer-centricity — humans and agents are treated identically, so multi-agent and group chat scenarios are first-class.

A background **Deriver** engine continuously processes incoming messages into two outputs: conversation summaries (short summaries every ~20 messages, recursive long summaries every ~60) and **Peer Representations** (rich, informationally dense models of who and what each peer is). These representations are accessible through a **Dialectic API** — you can literally ask Honcho "what should I know about this user?" and get synthesized insights.

Honcho distinguishes between local representations (one peer's view of another, within a session) and global representations (based on all messages ever produced by a peer across all sessions).

### Mem0

Mem0 treats memory as a first-class infrastructure problem with lifecycle management modeled after human memory: **intelligent filtering** (priority scoring, not everything gets stored), **dynamic forgetting** (low-relevance entries decay over time), **memory consolidation** (frequent recalls move from short-term to long-term storage), and **cross-session continuity** (context persists across sessions, devices, and time).

Memory retrieval is graph-based rather than flat-vector: it builds a dynamic graph of connected memories and retrieves the subgraph relevant to the current query. Mem0 also offers an **OpenMemory MCP** for interoperability across agent frameworks.

Backed by Y Combinator, SOC 2 and HIPAA compliant. Available as both a managed platform and open-source self-hosted option. Benchmarked on LoCoMo, LongMemEval, and BEAM.

### Hindsight

Hindsight takes a biomimetic approach — it structures memories the way human memory works: facts are extracted, normalized into entities and relationships, timestamped, and stored with sparse/dense vector representations. Three operations define the API:

- **Retain**: extract key facts, temporal data, entities, and relationships from input
- **Recall**: TEMPR retrieval — four strategies run in parallel (semantic, keyword/BM25, graph-based, temporal) then merged and reranked with a cross-encoder
- **Reflect**: agentic reasoning over memories to form new connections and build deeper understanding

Behind the scenes, Hindsight auto-consolidates related facts into **Observations** — deduplicated, evidence-grounded beliefs with provenance tracking (exact source quotes), proof counts, and freshness trends (stable/strengthening/weakening/stale). Users can also create **Mental Models** — curated summaries for common queries.

Memory banks are configurable with a **Mission** (identity/priorities), **Directives** (hard rules), and **Disposition** traits (skepticism, literalism, empathy on 1-5 scales) that shape how `reflect` reasons.

State-of-the-art on the LongMemEval benchmark as of January 2026. Available as managed cloud or open-source self-hosted server.

### Supermemory

Supermemory positions itself as a **context engineering platform** rather than just a memory store. It provides five layers of context: **User Profiles** (who the user is), **Memory Graph** (structured knowledge), **Retrieval** (search infrastructure), **Extractors** (data ingestion), and **Connectors** (integrations with external tools and platforms).

The company raised $2.6M in seed funding and targets enterprise use cases where multiple agents, tools, and systems need shared context. Think of it as the "enterprise integration" option — more platform than plugin.

### OpenViking

OpenViking takes the most unusual approach: it treats AI agent context as a **filesystem**. All memories, resources, and skills live under a `viking://` URI scheme organized as hierarchical directories. You browse context with familiar operations: `ls` (list directory), `search` (hybrid vector + grep), `read` (fetch content by URI), `overview` (directory summary).

The tiered loading system is clever: **L0** (abstract summary, <100 tokens) for quick awareness, **L1** (structured overview, <2k tokens) for planning, and **L2** (full details, on-demand by URI) for execution. This dramatically reduces token costs while keeping full context accessible.

Every retrieval path is traceable — you can audit why the agent pulled a specific piece of context. OpenViking is also self-evolving: it extracts experience from execution and dialogue to continuously optimize memory.

Pip-installable. Integrates with OpenClaw, OpenCode, LangChain, and DeerFlow.

### Byterover

Byterover inverts the conventional memory pipeline: instead of delegating storage to an external service, **the same LLM that reasons about a task also curates, structures, and retrieves knowledge**. Memory operations (`ADD`, `UPDATE`, `UPSERT`, `MERGE`, `DELETE`) are tools in the agent's toolkit, not API calls to an external service. This eliminates semantic drift — the system that stores knowledge actually understands it.

Knowledge is represented in a **Context Tree**: `Domain → Topic → Subtopic → Entry`. Each entry carries explicit relations, provenance, and an **Adaptive Knowledge Lifecycle (AKL)** with importance scoring, maturity tiers, and recency decay.

Retrieval uses a 5-tier progressive strategy that resolves most queries at sub-100ms latency without any LLM calls, escalating to agentic reasoning only for genuinely novel questions. All knowledge is stored as **human-readable markdown files on the local filesystem** — version-controllable, portable, and requiring zero external infrastructure: no vector database, no graph database, no embedding service.

State-of-the-art accuracy on LoCoMo, competitive on LongMemEval. Exposes `brv-query` and `brv-curate` as MCP tools.

### RetainDB

RetainDB is the pragmatic, framework-friendly option. It stores memories organized by **memory type** (preference, context, decision) with multi-backend storage: vector DB for semantic search, key-value for fast lookups, graph DB for relationships, and full-text for keyword queries.

Key features include async writes (non-blocking agent responses), read-after-write consistency (newly written memories are immediately readable), and built-in integrations for LangChain, OpenAI Agents SDK, and custom agent frameworks.

Positioned as a lightweight SaaS for developers who want persistent memory without managing infrastructure. Straightforward API: `memory.add()`, `memory.search()`, `memory.update()`.

### Holographic

Holographic uses **Holographic Reduced Representations (HRR)** — a mathematical framework where facts are key-value pairs compressed into fixed-size complex vectors. Multiple facts superpose into one mathematical object but remain individually retrievable through algebraic binding/unbinding operations.

The operations are: **remember** (bind a key-value pair into the holographic vector), **recall** (unbind and decode via cosine similarity, ~1ms), **forget** (subtract a binding from the superposition), and **promote** (facts recalled 3+ times across sessions get written to permanent context).

Storage is local SQLite. No external services, no vector database, no embedding API — just an algebraic engine over SQLite. Trust scoring is applied to recalled memories as a correctness signal. Retrieval latency is sub-millisecond since it's pure math, not a search.

The trade-off is clear: you get extremely fast, dependency-free local memory, but without the rich structured extraction, entity resolution, or multi-strategy retrieval that systems like Hindsight offer. Best where simplicity and local operation matter most.
