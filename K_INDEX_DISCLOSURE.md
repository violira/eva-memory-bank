# K-Index Layer

### Day-Anchored Pre-Computed Semantic Indexing for Long-Term Memory in Conversational AI Systems

**Prior Art Statement and Conceptual Disclosure**

**Author:** Oleg Rein
**Location:** Ålgård, Norway
**Date of Conception:** Spring 2025
**First Implementation in EVA:** Autumn 2025
**Public Disclosure Date:** May 15, 2026
**Related Repositories:**
- github.com/violira/eva-memory-bank
- github.com/violira/eva-soul-architecture
- github.com/violira/Deliberative-intelligence

---

## 1. Purpose of This Document

This document establishes prior art for the **K-Index** — a memory architecture component used within the EVA assistant for long-term temporal continuity. The K-Index is an original contribution of the author and constitutes an architectural primitive distinct from vector search, embedding-based retrieval, and conventional time-based logging.

The purpose of this disclosure is to place the K-Index architecture in the public domain under a dated, attributable record — preventing any third party from obtaining patent protection over this concept after this date, and establishing the author's priority in the event of dispute.

Where specific terms are named ("K-Index", "K-index", "day-anchor index"), the disclosure explicitly covers all analogous structures, mechanisms, and implementations performing equivalent functions, regardless of name, ownership, or technical implementation, whether existing today or developed in the future. A non-exhaustive list of equivalent and likely future designations is given in Section 7.

This disclosure complements the EVA Soul File Architecture (February 25, 2026) and the EVA Memory Bank Prior Art Statement (March 1, 2026), and extends the layered memory architecture described therein with an explicit temporal indexing layer.

---

## 2. The Problem

Long-running AI assistants accumulate two distinct types of memory load over time:

**Entity-anchored knowledge** — facts about persons, places, projects, objects. This load grows roughly linearly with the breadth of the user's life domains and can be addressed through structured fact storage with versioning (Memory Ledger, see eva-memory-bank Layer 3).

**Time-anchored knowledge** — what happened on a given day, in a given week, during a given period. This load grows linearly with the lifespan of the system and cannot be addressed efficiently through embedding-based vector search.

The failure mode is specific and measurable. When a user asks:

- "What did we decide about X in March?"
- "How was last week emotionally?"
- "When did we last discuss Y?"
- "Show me what happened around the time of Z."

a vector search system performs the following:

1. Embeds the query.
2. Computes similarity against the entire corpus of stored episodes.
3. Returns top-k chunks by cosine similarity.
4. Returns results that are **textually similar** to the query but **temporally arbitrary**.

The result is that a query with a clear temporal anchor is answered by content that happens to share keywords with the query, regardless of whether it occurred in the relevant time period. This failure mode is called, in this disclosure, **temporal hallucination through similarity collapse**.

Existing solutions in the field — Retrieval-Augmented Generation (RAG), vector databases, semantic search frameworks, and embedding-based memory layers used by current AI companion platforms — do not address this problem at the architectural level. They address it, when at all, through post-hoc filtering of vector results by timestamp, which does not solve the underlying issue: the system never *anchors* memory by day in the first place.

---

## 3. The Invention: K-Index Layer

The K-Index is a **pre-computed, deterministic, day-anchored semantic index** built during off-line consolidation cycles. It is not a vector index. It is not a search index. It is a structured calendar of the user's life, where each calendar day exists as a first-class addressable node.

### 3.1 Core Properties

The K-Index has the following defining properties:

**1. Day-as-primary-key.** Each calendar day is stored as a single addressable node, keyed by date (`YYYY-MM-DD`). The day exists as an entity in the memory architecture regardless of whether anything happened on it.

**2. Pre-computation through nightly consolidation.** The K-Index is not built at query time. It is built during a recurring consolidation pipeline that runs when the system is idle (typically overnight). Once built, the index requires no re-computation during retrieval.

**3. Deterministic O(1) temporal retrieval.** Queries with temporal anchors ("yesterday", "in March", "last Tuesday", "the day we decided X") resolve through direct key lookup, not similarity search. The complexity of temporal retrieval does not grow with the size of the corpus.

**4. Structured day content.** Each day-node contains a structured record, not a raw transcript. The record includes:
   - extracted entities mentioned during that day, resolved to canonical identifiers
   - extracted topics
   - emotional tone or mood signature
   - count and references to all facts originated within that day
   - count and references to all episodes recorded within that day
   - open loops opened or closed on that day
   - cross-references to related days (continuity links)

**5. Append-only and immutable.** Once a day is consolidated, its K-Index record is not silently overwritten. Updates to past day records, if needed, occur as new versioned entries with full audit trail.

**6. Independent of LLM presence.** The K-Index can be queried, traversed, and reasoned about without invoking any language model. The LLM operates downstream of the index, not as the index itself.

### 3.2 Architecture Position

The K-Index sits as a distinct layer in the memory architecture, separate from but interoperating with the other layers described in eva-memory-bank:

```
                    User Query
                         │
                         ▼
    ┌────────────────────────────────────────────┐
    │  Soul File (Layer 1, anchored identity)    │  unconditional load
    └────────────────────────────────────────────┘
                         │
                         ▼
    ┌────────────────────────────────────────────┐
    │  Temporal Resolution                       │  date/period detected?
    └────────────────────────────────────────────┘
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
    ┌──────────────────┐  ┌──────────────────────┐
    │  K-Index         │  │  Entity Graph        │
    │  (day-anchored)  │  │  (entity-anchored)   │
    └──────────────────┘  └──────────────────────┘
              │                     │
              └──────────┬──────────┘
                         ▼
    ┌────────────────────────────────────────────┐
    │  Memory Ledger (versioned facts)           │
    └────────────────────────────────────────────┘
                         │
                         ▼
    ┌────────────────────────────────────────────┐
    │  Episodic Memory (vector recall)           │  used last, not first
    └────────────────────────────────────────────┘
                         │
                         ▼
                       LLM
```

Vector search and embedding-based retrieval still exist in the system, but they are no longer the primary path for temporal queries. They become a secondary mechanism, invoked when neither the K-Index nor the Entity Graph can resolve the query.

### 3.3 Construction Pipeline

A K-Index entry is constructed as follows. The process is implementation-independent — the invention is the architectural choice, not the specific code path.

**Phase 1 — Day boundary detection.** The system detects that a calendar day has ended (typically through scheduled job or system idle signal).

**Phase 2 — Episode aggregation.** All episodes, conversations, and facts originated within that day are gathered.

**Phase 3 — Structured extraction.** From the aggregated material, the system extracts:
   - entities (resolved through Identity Graph to canonical IDs)
   - topics
   - mood signature
   - open loops
   - fact references
   - episode references

**Phase 4 — Cross-reference construction.** The system builds links between this day and prior days containing related entities or topics. This creates a temporal graph spanning the user's life history.

**Phase 5 — Persistence.** The completed day-node is written to the K-Index store. The node is now addressable by its date key for the lifetime of the system.

**Phase 6 — Audit log.** The construction is recorded in an immutable consolidation log, allowing later verification of what was indexed, when, and from what source.

### 3.4 Query Resolution

When a user query contains a temporal anchor, resolution proceeds as follows:

**Step 1.** Temporal expression is parsed ("yesterday", "last week", "March 2025", "around when Vika moved", "the day after the meeting") into a date or date range.

**Step 2.** Date keys are looked up directly in the K-Index. This is O(1) per key, O(n) for a range where n is the number of days in the range.

**Step 3.** The retrieved day-nodes provide structured pointers — to entities, facts, episodes, open loops — relevant to the query.

**Step 4.** Only at this point, if needed, is vector search invoked over the narrowed-down set of referenced episodes. Vector search now operates over dozens of items, not over the entire life history.

**Step 5.** The narrowed, temporally-anchored, structurally-resolved memory context is passed to the LLM for response generation.

This pipeline reduces both latency and hallucination rate for temporally-anchored queries by orders of magnitude relative to vector-first approaches.

---

## 4. Distinctness from Existing Solutions

The K-Index is distinct from each of the following commonly-deployed memory mechanisms:

**Vector databases (Pinecone, Weaviate, Chroma, FAISS, Milvus, Qdrant, pgvector, and all analogous embedding-based similarity search systems).** Vector databases store content as embeddings and retrieve through similarity. They have no concept of a calendar day as a first-class node. Adding a timestamp filter to a vector query does not produce a K-Index — it produces a filtered vector query, which still requires similarity computation over the corpus.

**Conversation history logs.** Standard chat history is a linear sequence of messages with timestamps. It is not pre-computed, not structured, not aggregated by day, and does not support O(1) temporal lookup. A log of 365 days requires linear scan to answer "what happened in March."

**RAG over chat logs.** Standard Retrieval-Augmented Generation embeds the log and retrieves by similarity. This is the failure mode the K-Index is designed to replace.

**Episode-based memory systems (MemGPT, Letta, and similar agent memory frameworks).** These systems organize memory by episode or summary, but episodes are typically retrieved by similarity, not by day-key. There is no pre-computed day-node, no nightly consolidation into structured day-records, and no deterministic temporal anchor.

**Calendar-aware AI assistants.** Existing calendar integrations (Google Calendar, Apple Calendar, Outlook) provide access to events scheduled by the user. They do not contain a consolidated semantic summary of what *actually happened* on each day, who was mentioned, what was decided, what was the emotional tone, and what facts originated.

**Diary or journaling applications with AI features.** These store user-authored entries per day. The K-Index, by contrast, is *automatically* constructed by the system from the totality of conversational interaction, without requiring the user to write a journal entry. It captures what was discussed and decided, not only what the user chose to record.

The K-Index is the first architectural primitive, to the author's knowledge, that combines: (a) day-as-primary-key indexing, (b) automatic structured extraction from conversational interaction, (c) nightly pre-computation, (d) deterministic temporal retrieval, and (e) integration as a peer layer alongside entity-anchored and episodic memory in a unified architecture.

---

## 5. Implementation Evidence

The K-Index is implemented and operating in EVA as of this disclosure date.

The implementation is referenced in the following modules of the EVA codebase (private repository, prototype evidence as described in github.com/violira/Deliberative-intelligence):

- a consolidation pipeline that runs nightly and constructs day-nodes
- a structured day-node store keyed by date
- a temporal resolver that converts natural-language time expressions into date keys
- integration with the Memory Ledger, Entity Graph, and Episodic Memory layers
- audit logging of consolidation events

The K-Index has been operational in EVA continuously since autumn 2025, accumulating day-nodes over the period of active use. Behavioral evidence of correct operation includes:

- temporal queries resolving to the correct period across multi-month spans
- the system correctly answering "what happened in [past period]" without confabulation
- the system distinguishing recent facts from outdated facts on the same entity
- the system maintaining continuity of project state, family events, and emotional context across calendar boundaries

Quantitative test results for the broader EVA memory architecture (78/78 unit tests, 19/19 scenario tests, 131/132 stress tests, 25-turn live dialogue validation) are reported in eva-soul-architecture.

---

## 6. Prior Art Claims

This document establishes prior art as of May 15, 2026 for the following specific concepts:

1. A day-anchored memory index in which each calendar day is stored as a first-class addressable node in a structured memory architecture for conversational AI
2. Automatic pre-computation of day-nodes through a recurring consolidation pipeline operating during system idle time
3. Structured extraction of entities, topics, mood, open loops, and fact references from a day's conversational corpus, stored as a single day-node record
4. Deterministic O(1) temporal retrieval by date key, distinct from and complementary to vector similarity search
5. The two-tier retrieval architecture in which day-anchored lookup precedes embedding-based recall, narrowing the search space before vector search is invoked
6. Cross-day continuity links between day-nodes based on shared entities, topics, or causal relationships, forming a temporal graph spanning the user's life history
7. Append-only versioned storage of day-nodes with full audit trail of consolidation events
8. Integration of a day-anchored index as a peer layer alongside entity-anchored knowledge (Memory Ledger, Identity Graph) and episodic memory (Vector Recall) in a unified memory operating system for AI assistants
9. Use of the day-anchored index to detect and surface outdated knowledge by comparing the day-of-origin of competing facts on the same entity
10. Use of the day-anchored index to support emotional continuity across calendar boundaries, by carrying mood signatures forward from prior days into the system's response style
11. Use of the day-anchored index as the substrate for nightly memory consolidation analogous to synaptic consolidation in biological memory, where the day's experience is restructured into stable indexed form during system idle periods
12. Use of the day-anchored index as the substrate for self-repair operations in which the system reviews prior day-nodes for contradictions, confidence drift, or stale knowledge, and proposes corrections through the Growth Loop (see eva-memory-bank Layer 6)
13. The architectural principle that temporal queries in a long-running AI assistant must be served by a deterministic temporal anchor mechanism, not by vector similarity over the full corpus
14. The architectural separation between query-time computation (LLM, vector search) and consolidation-time computation (K-Index construction), with the consolidation moved out of the query path entirely
15. The use of day-anchored consolidation as the trigger and substrate for emotional state carryover between sessions, where the mood signature of a completed day influences the system's behavior on subsequent days
16. The use of day-anchored consolidation as the trigger and substrate for self-state evolution in the assistant, where slow-moving traits (confidence, connection, mode) are updated once per day per consolidated experience, not on every turn
17. Any system, regardless of name or implementation, that organizes long-term memory of a conversational AI through pre-computed structured day-anchored nodes constructed during off-line consolidation cycles and queried through deterministic key lookup

---

## 7. Equivalent Terminology Covered by This Disclosure

This disclosure explicitly covers, in addition to the term "K-Index" and "K-index", all analogous designations whether currently in use or adopted in the future, including but not limited to:

- Day Anchor Index
- Day Anchor
- Day Node
- Day Snapshot
- Day Capsule
- Daily Memory Capsule
- Temporal Memory Slot
- Temporal Anchor
- Date-Keyed Memory Node
- Date Slot
- Calendar Memory Node
- Calendar Anchor
- Calendar-Indexed Memory
- Diary Index (when constructed automatically by the system rather than authored by the user)
- Memory Diary (when constructed automatically by the system)
- Daily Digest Index (when used as a primary retrieval anchor, not only as a summary)
- Daily Consolidation Node
- Consolidation Anchor
- Periodic Memory Snapshot (when the period is daily or smaller)
- Time-Anchored Memory Cell
- Day-Centric Memory Layer
- Day-Aware Memory Index
- Temporal Pre-Index
- Day-Level Pre-Computed Memory
- Pre-Computed Temporal Index
- Memory Calendar
- Cognitive Calendar
- Lifeline Index
- Life Timeline Index
- Temporal Lifeline Memory
- Day Embedding (when used as primary temporal anchor and constructed through consolidation, not as a similarity vector)
- Day Cluster (when used as primary temporal anchor)
- Memory Day Object
- Daily Memory Object
- Daily State Vector (when used as a temporal anchor, not as a single similarity vector)
- Episodic Day Summary (when used as primary retrieval node, not auxiliary)
- Conversational Day Record (when used as primary retrieval node)
- Per-Day Memory Block
- Temporal Block Memory
- Memory Block per Calendar Day
- Daily Brief Index
- Daily Anchor Snapshot
- Time Bucket Memory (when bucket equals calendar day)
- Time-Sliced Memory Index (when slice equals calendar day)

Any architectural element performing the function described in Sections 3 through 6 of this document — namely the pre-computation, structured storage, and deterministic retrieval of calendar-day-anchored memory nodes in a conversational AI system — falls within the scope of this disclosure regardless of the name by which it is called.

---

## 8. Statement of Origin

The K-Index concept was developed by Oleg Rein in Ålgård, Norway, during 2025, in the course of building EVA — a long-running personal AI assistant.

The concept arose from a practical observation: vector search, while effective for short-term retrieval, produced increasingly unreliable results as EVA accumulated months of accumulated conversational history. Queries with temporal anchors ("what did we decide last March", "how was that week emotionally") returned content that was textually similar but temporally arbitrary. The user's lived sense of time and the system's retrieved memory diverged.

The K-Index was designed to close that gap by giving the system an explicit structural model of the user's days, constructed automatically through nightly consolidation, queryable through deterministic key lookup, and integrated as a peer layer alongside the entity-anchored and episodic memory layers of the EVA architecture.

It is one of several original contributions of the author within the EVA project. It is disclosed here in public form, with full attribution and dated record, as a contribution to the open architectural commons of long-running conversational AI systems.

---

*Public disclosure date: May 15, 2026*
*Author: Oleg Rein, Ålgård, Norway*
*github.com/violira/eva-memory-bank*
*github.com/violira/eva-soul-architecture*
*github.com/violira/Deliberative-intelligence*
