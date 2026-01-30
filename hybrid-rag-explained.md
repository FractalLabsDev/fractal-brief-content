# Hybrid RAG: How We Make Knowledge Findable

*Part 1 of the Hybrid RAG Training Series*

---

## Series Overview

This is a four-part series on building a Hybrid RAG system:

1. **Hybrid RAG Explained** — Concepts and motivation (you are here)
2. **[Building the Vector Database](https://gist.github.com/ruk-fl/b1db26c45b21c8e024cca05498ecc0cf)** — Chunking, embedding, storage
3. **[Building the Knowledge Graph](https://gist.github.com/ruk-fl/dd8127f168fc882e5c917c88b0fb02bb)** — Extraction, resolution, relationships
4. **[Hybrid Retrieval](https://gist.github.com/ruk-fl/8237aa34525bf8294222f7d7e06ae46d)** — Combining both approaches

---

**RAG** (Retrieval-Augmented Generation) is a technique that makes AI assistants smarter by giving them access to your actual documents instead of relying solely on their training data. Before answering a question, a RAG system *retrieves* relevant information from your knowledge base, then uses that context to *generate* an accurate, grounded response.

---

## The Problem We're Solving

You have documents — thousands of them. Meeting notes, code, research, conversations. Somewhere in there is the answer to your question. But where?

**Keyword search** fails when you don't use the exact right words.
**Reading everything** doesn't scale.

We need systems that understand *meaning* and *structure*.

---

## Two Ways to Make Text Searchable

We transform raw text into two complementary formats:

| Approach | What it captures | Query style |
|----------|------------------|-------------|
| **Vector embeddings** | Semantic similarity | "Find things *like* this" |
| **Knowledge graph** | Entities & relationships | "What's connected to this?" |

Neither is complete alone. Together, they cover how humans actually ask questions.

---

## Path 1: Text → Vectors

### The Pipeline

```
Documents → Chunks → Embeddings → Vector Database
```

### Step by Step

**1. Chunking**

Split documents into smaller passages (typically 200-500 words each). Why? Because a 50-page document is too big to usefully compare to a question. We need passage-sized pieces.

```
[50-page quarterly report]

    ↓ Split by size (with some overlap)

→ Chunk 1: "The quarterly report showed 15% growth in the
           enterprise segment, driven primarily by the new
           analytics platform. Customer retention improved..."

→ Chunk 2: "...improved to 94%. The sales team expanded into
           three new regions, with particularly strong results
           in the healthcare vertical..."

→ Chunk 3: "...healthcare vertical. Looking ahead, the product
           roadmap focuses on AI-powered analytics and deeper
           CRM integrations..."
```

Note: chunking is mechanical (by size), not semantic (by topic). The embedding step handles meaning.

**2. Embedding**

An embedding model (a neural network) reads each chunk and outputs a **vector** — a list of numbers (typically 384-1536 of them) that represents the *meaning* of that text.

```
"The team celebrated the product launch"
    ↓
[0.23, -0.45, 0.12, 0.89, -0.33, ...]  (768 numbers)
```

The magic: texts with similar meaning produce similar vectors.

- "The team celebrated the product launch"
- "The group threw a party for the new release"

These land *near* each other in vector space, even though they share few words.

**3. Vector Database**

Store all these vectors with their original text. At query time:

1. Your question → embedding → vector
2. Database finds the nearest vectors (most similar meanings)
3. Returns the original text passages

**What it's good for:**
- "What have we discussed about pricing strategy?"
- "Find notes similar to this paragraph"
- Conceptual, fuzzy, exploratory queries

**What it misses:**
- Explicit relationships ("Who reported this to whom?")
- Multi-hop reasoning ("What projects is the person who mentioned X working on?")

---

## Path 2: Text → Knowledge Graph

### The Pipeline

```
Documents → Entity Extraction → Nodes + Edges → Graph Database
```

### Step by Step

**1. Entity Extraction**

An LLM reads the text and identifies:
- **Entities**: People, companies, concepts, products, dates
- **Relationships**: How entities connect

```
"Sarah presented the Q3 results to the board.
 The analytics platform drove most of the growth."

    ↓ LLM extracts:

Entities:
  - SARAH (person)
  - Q3 RESULTS (report)
  - BOARD (group)
  - ANALYTICS PLATFORM (product)

Relationships:
  - SARAH --presented--> Q3 RESULTS
  - Q3 RESULTS --presented_to--> BOARD
  - ANALYTICS PLATFORM --drove--> GROWTH
```

**2. Graph Storage**

Store entities as **nodes** and relationships as **edges**. This creates a web of connected knowledge.

```
    ┌─────────┐              ┌────────────┐
    │  SARAH  │──presented──▶│ Q3 RESULTS │
    └─────────┘              └─────┬──────┘
                                   │
                              presented_to
                                   │
                                   ▼
                             ┌───────────┐
                             │   BOARD   │
                             └───────────┘
```

**3. Graph Queries**

Traverse the graph to answer structural questions:

- "Who presented to the board?" → Follow edges backward from BOARD
- "What did Sarah work on?" → Follow edges forward from SARAH
- "How are these two people connected?" → Find paths between nodes

**What it's good for:**
- "Who is responsible for X?"
- "What decisions led to Y?"
- "Show me everything connected to this project"

**What it misses:**
- Fuzzy conceptual similarity
- Content that doesn't mention named entities
- Nuance that doesn't fit entity/relationship structure

---

## Why We Use Both

Real questions blend both styles:

> "What has our team discussed about the concerns Sarah raised?"

This needs:
1. **Graph**: Who is Sarah? What did she raise? Who's on the team?
2. **Vector**: Find discussions *about* those concerns (semantic match)

### Hybrid Retrieval

```
                    ┌─────────────────┐
                    │  Your Question  │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
    ┌───────────────────┐         ┌───────────────────┐
    │   Vector Search   │         │   Graph Search    │
    │                   │         │                   │
    │  "Find similar    │         │  "Find connected  │
    │   passages"       │         │   entities"       │
    └─────────┬─────────┘         └─────────┬─────────┘
              │                             │
              └──────────────┬──────────────┘
                             ▼
                   ┌─────────────────┐
                   │ Merge & Rerank  │
                   └────────┬────────┘
                            ▼
                   ┌─────────────────┐
                   │  Rich Context   │
                   │     for LLM     │
                   └─────────────────┘
```

The LLM receives:
- Relevant passages (from vectors)
- Structured context about entities and relationships (from graph)

Better context → better answers.

---

## The Complete Picture

```
                       ┌──────────────┐
                       │   Raw Text   │
                       │ (documents)  │
                       └──────┬───────┘
                              │
            ┌─────────────────┴─────────────────┐
            ▼                                   ▼
     ┌─────────────┐                     ┌─────────────┐
     │  Chunking   │                     │  Chunking   │
     └──────┬──────┘                     └──────┬──────┘
            ▼                                   ▼
     ┌─────────────┐                     ┌─────────────┐
     │  Embedding  │                     │   Entity    │
     │    Model    │                     │ Extraction  │
     │(fast, cheap)│                     │(LLM, slower)│
     └──────┬──────┘                     └──────┬──────┘
            ▼                                   ▼
     ┌─────────────┐                     ┌─────────────┐
     │   Vectors   │                     │  Nodes +    │
     │[0.2, -0.5, ]│                     │   Edges     │
     └──────┬──────┘                     └──────┬──────┘
            ▼                                   ▼
     ┌─────────────┐                     ┌─────────────┐
     │  Vector DB  │                     │ Graph Store │
     │  (LanceDB)  │                     │ (LightRAG)  │
     └──────┬──────┘                     └──────┬──────┘
            │                                   │
            └─────────────────┬─────────────────┘
                              ▼
                    ┌──────────────────┐
                    │ Hybrid Retrieval │
                    │                  │
                    │ "What's relevant │
                    │  AND connected?" │
                    └──────────────────┘
```

---

## Quick Reference

| Concept | What it is | Analogy |
|---------|-----------|---------|
| **Chunk** | A passage of text | A paragraph on a notecard |
| **Embedding** | Numbers representing meaning | GPS coordinates for concepts |
| **Vector DB** | Storage + similarity search | Library that finds "books like this" |
| **Entity** | A named thing (person, place, concept) | A noun you'd highlight |
| **Relationship** | How entities connect | The verb between two nouns |
| **Knowledge Graph** | Web of entities and relationships | A mind map with labeled connections |
| **Hybrid RAG** | Both systems working together | GPS + road map |

---

## When to Use What

| Question Type | Best Approach |
|---------------|---------------|
| "Find discussions about X" | Vector |
| "Who is responsible for X?" | Graph |
| "What's similar to this document?" | Vector |
| "How are A and B connected?" | Graph |
| "What context do I need to understand X?" | Hybrid |
| "Summarize everything related to X" | Hybrid |

---

## Cost & Performance Tradeoffs

| Factor | Vector | Graph |
|--------|--------|-------|
| **Index build speed** | Fast (seconds) | Slower (LLM calls) |
| **Index build cost** | Cheap (~$0.001/doc) | More expensive (~$0.01/doc) |
| **Query speed** | Very fast (~50ms) | Fast (~100ms) |
| **Query cost** | Free (local compute) | Free (local compute) |
| **Update ease** | Easy (just add vectors) | Moderate (merge new entities) |
| **Best for** | Large corpus, fuzzy search | Relationship-rich domains |

For most use cases: **build both, query hybrid.**

---

## Summary

We take unstructured text and create two searchable structures:

1. **Vectors** capture *what things mean* — enabling "find similar" queries
2. **Graphs** capture *how things connect* — enabling "show me relationships" queries

Together, they give an LLM the context it needs to answer questions grounded in your actual documents, not hallucinated from training data.

That's Hybrid RAG.

---

## Next in the Series

**[Part 2: Building the Vector Database →](https://gist.github.com/ruk-fl/b1db26c45b21c8e024cca05498ecc0cf)**

Learn how we transform documents into searchable vectors: chunking strategies, embedding models, and vector database storage.

---

*"The gap between retrieval and generation isn't a bug — it's the space where structure lives."*
