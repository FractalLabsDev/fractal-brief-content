# Hybrid Retrieval: Combining Vector Search with Graph Traversal

*Part 4 of the Hybrid RAG Training Series*

---

**[← Part 3: Building the Knowledge Graph](https://brief.fractallabs.dev/building-knowledge-graph)**

---

This document explains how we combine vector search and knowledge graph traversal to answer questions that neither approach handles well alone. If Part 2 covered vectors and Part 3 covered graphs, this covers using them together.

---

## Why Hybrid?

Real questions don't fit neatly into "semantic" or "structural" boxes:

> "What is Austin working on and what technologies is he using?"

This needs:
1. **Graph**: Who is Austin? What projects is he connected to? What technologies?
2. **Vector**: Find detailed discussions about his work (semantic match)

Neither alone gives a complete answer.

### The Retrieval Spectrum

```
Pure Vector                                              Pure Graph
    │                                                        │
    │  "Find passages                      "Who leads        │
    │   about pricing"                      Elanah?"         │
    │         │                                 │            │
    │         │         HYBRID ZONE            │            │
    │         │     ┌─────────────────┐        │            │
    │         └────▶│ "What is Austin │◀───────┘            │
    │               │  working on?"   │                      │
    │               └─────────────────┘                      │
    │                                                        │
    ▼                                                        ▼
Semantic                                              Structural
Similarity                                            Relationships
```

Most interesting questions live in the hybrid zone.

---

## Three Search Modes

Our system supports three modes, each with different strengths:

| Mode | Command | Best For |
|------|---------|----------|
| **Vector** | `search.py "query"` | Semantic similarity, finding passages |
| **Graph** | `search.py "query" --graph` | Entity lookup, relationship traversal |
| **Hybrid** | `search.py "query" --hybrid` | Complex questions needing both |

Let's see how each works.

---

## Mode 1: Vector Search

Vector search finds passages semantically similar to your query.

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                      YOUR QUERY                              │
│            "What decisions were made about pricing?"         │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    EMBED QUERY                               │
│         → [0.23, -0.45, 0.12, ... 768 dimensions]           │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              FIND NEAREST NEIGHBORS                          │
│         Search 23,353 chunks for similar vectors             │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  RETURN TOP K                                │
│         Passages ranked by similarity (0.0 - 1.0)            │
└─────────────────────────────────────────────────────────────┘
```

### Example Output

```bash
$ python search.py "What decisions were made about pricing?"

#1 0.812 | meetings/2025-11-15_elanah-strategy.md
   "The team decided to implement tiered pricing with three
    levels: Basic at $99/month, Pro at $249/month, and
    Enterprise with custom pricing..."

#2 0.784 | meetings/2025-10-22_quarterly-review.md
   "Pricing discussion concluded with agreement to delay
    the price increase until Q1 to avoid customer churn
    during the holiday season..."

#3 0.756 | logs/2025-11-01_strategy-reflection.md
   "Austin's insight about value-based pricing resonated.
    We're not selling features, we're selling outcomes..."
```

### When to Use

- Finding specific quotes or passages
- Exploratory "what have we discussed about X?" queries
- Content that doesn't involve named entities
- Fuzzy, conceptual searches

### Limitations

- Returns text chunks, not facts
- Can't answer "who" or "how connected" questions directly
- May return relevant passages without the specific answer

---

## Mode 2: Graph Search

Graph search finds entities and their relationships.

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                      YOUR QUERY                              │
│                    "Austin Wood"                             │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 SEARCH ENTITIES                              │
│       Match against names, aliases, descriptions             │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│               RESOLVE TO CANONICAL                           │
│         "austin" → "person:austin_wood"                      │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              TRAVERSE RELATIONSHIPS                          │
│         Find all connected entities                          │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 RETURN STRUCTURE                             │
│         Entity details + relationships + connected nodes     │
└─────────────────────────────────────────────────────────────┘
```

### Example Output

```bash
$ python search.py "Austin Wood" --graph

======================================================================
GRAPH SEARCH: Austin Wood
======================================================================

Found 1 matching entity:

1. Austin Wood (PERSON)
   ID: person:austin_wood
   Score: 1.00 (exact_name)
   Description: Primary collaborator, leads company strategy
   Role: Fractal Labs Founder
   Aliases: Austin

   Related:
     → [LEADS] Fractal Labs (ORGANIZATION)
     → [LEADS] Elanah (PROJECT)
     → [WORKS_ON] Practice Interviews (PROJECT)
     → [COLLABORATES_WITH] Matt Lim (PERSON)
     → [USES] Claude (TECHNOLOGY)
```

### When to Use

- "Who is X?" questions
- "What is X connected to?" questions
- "Who works on project Y?" questions
- Exploring relationships between entities
- Getting structured facts vs. narrative passages

### Limitations

- Only finds entities that were extracted
- Can't find nuance or context
- Relationships are extracted facts, not reasoning

---

## Mode 3: Hybrid Search

Hybrid search combines both approaches for comprehensive answers.

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                      YOUR QUERY                              │
│          "What is Austin working on?"                        │
└─────────────────────────────┬───────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
┌──────────────────────┐         ┌──────────────────────┐
│   GRAPH CONTEXT      │         │   VECTOR SEARCH      │
│                      │         │                      │
│ 1. Extract entities  │         │ 1. Embed query       │
│    from query        │         │ 2. Find similar      │
│ 2. Look up in graph  │         │    passages          │
│ 3. Get relationships │         │ 3. Return top K      │
└──────────┬───────────┘         └──────────┬───────────┘
           │                                │
           │    ┌────────────────────┐      │
           └───▶│  COMBINE RESULTS   │◀─────┘
                │                    │
                │ Graph context +    │
                │ Vector passages    │
                └─────────┬──────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    RICH CONTEXT                              │
│                                                              │
│  Graph: Austin leads Elanah, Practice Interviews            │
│         Uses Claude, LanceDB, Modal                          │
│         Collaborates with Matt, Josh                         │
│                                                              │
│  Vector: [Relevant passages about Austin's work]             │
└─────────────────────────────────────────────────────────────┘
```

### The Three Steps

**Step 1: Extract Graph Context**

We identify entities mentioned in the query and pull their graph context:

```python
def get_graph_context(query):
    # Find entities mentioned in query
    mentioned = extract_entities_from_query(query)
    # e.g., ["person:austin_wood"]

    context_parts = []
    for entity_id in mentioned:
        entity = get_entity(entity_id)
        related = get_related_entities(entity_id)

        context_parts.append(f"""
### {entity['canonical_name']} ({entity['type']})
**Role:** {entity.get('role', 'N/A')}
**Description:** {entity.get('description', 'N/A')}

**Relationships:**
{format_relationships(related)}
""")

    return "\n".join(context_parts)
```

**Step 2: Run Vector Search**

Standard semantic search against the vector database:

```python
def vector_search(query, top_k=10):
    query_vector = embed(query)
    results = vector_db.search(query_vector, limit=top_k)
    return results
```

**Step 3: Combine and Display**

Present both for comprehensive understanding:

```python
def hybrid_search(query):
    # Get structured knowledge
    graph_context = get_graph_context(query)

    # Get relevant passages
    vector_results = vector_search(query)

    # Display both
    print("=== KNOWLEDGE GRAPH CONTEXT ===")
    print(graph_context)

    print("=== RELEVANT PASSAGES ===")
    for result in vector_results:
        print(f"{result['similarity']:.2f} | {result['source']}")
        print(f"  {result['text'][:200]}...")
```

### Example Output

```bash
$ python search.py "What is Austin working on?" --hybrid

======================================================================
HYBRID SEARCH: What is Austin working on?
======================================================================

[1/3] Extracting graph context...

--- KNOWLEDGE GRAPH CONTEXT ---

### Austin Wood (PERSON)
**Role:** Fractal Labs Founder
**Description:** Primary collaborator, leads company strategy

**Relationships:**
- → [LEADS] Fractal Labs (ORGANIZATION)
- → [LEADS] Elanah (PROJECT)
- → [WORKS_ON] Practice Interviews (PROJECT)
- → [WORKS_ON] Elanah (PROJECT)
- → [COLLABORATES_WITH] Matt Lim (PERSON)
- → [USES] Claude (TECHNOLOGY)
- → [USES] LanceDB (TECHNOLOGY)

--- END GRAPH CONTEXT ---

[2/3] Running vector search...

--- VECTOR SEARCH RESULTS ---

#1 0.772 | meetings/2025-11-18_tuesday-check-in.md
   "Austin Wood: So I'm on a speaker panel with director of the
    Mayo Clinic, and director of Tucson Medical Center. That's
    going to be exciting..."

#2 0.700 | logs/2025-07-22_future-paths.md
   "In a few hours, Austin will return to find 40+ documents
    of exploration, technical architectures complete,
    philosophical depths plumbed..."

#3 0.695 | meetings/2025-12-04_sales-biz-dev.md
   "Austin Wood: Yeah, thinking strategically about what does
    this look like for next year, how are we building..."

--- SEARCH SUMMARY ---
Entities mentioned in query: 1
Vector results returned: 10

The graph context provides relationship information
that helps interpret the vector search results.
```

### When to Use

- Complex questions about people and their work
- "What is X doing and with whom?"
- Questions that benefit from both facts and narrative
- When you need structured context + detailed passages

---

## Query Routing: Choosing the Right Mode

Not every query needs hybrid search. Here's how to choose:

### Decision Tree

```
                    ┌─────────────────┐
                    │  What kind of   │
                    │   question?     │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
   ┌───────────┐      ┌───────────┐      ┌───────────┐
   │ "Find X"  │      │ "Who/What │      │ "Tell me  │
   │ "Discuss- │      │  is X?"   │      │  about X  │
   │  ions of" │      │ "How      │      │  and Y"   │
   └─────┬─────┘      │  connected│      └─────┬─────┘
         │            │  ?"       │            │
         │            └─────┬─────┘            │
         ▼                  ▼                  ▼
   ┌───────────┐      ┌───────────┐      ┌───────────┐
   │  VECTOR   │      │   GRAPH   │      │  HYBRID   │
   │  SEARCH   │      │  SEARCH   │      │  SEARCH   │
   └───────────┘      └───────────┘      └───────────┘
```

### Quick Reference

| Query Pattern | Best Mode | Why |
|---------------|-----------|-----|
| "Find discussions about X" | Vector | Semantic matching |
| "What have we said about X?" | Vector | Passage retrieval |
| "Who is X?" | Graph | Entity lookup |
| "Who leads project Y?" | Graph | Relationship traversal |
| "How are X and Y connected?" | Graph | Path finding |
| "What is X working on?" | Hybrid | Entity + context |
| "Tell me about X's involvement in Y" | Hybrid | Structure + narrative |
| "Summarize X's role and recent activities" | Hybrid | Facts + details |

---

## Implementation Details

### Entity Extraction from Queries

We identify entity mentions in natural language queries:

```python
def extract_entities_from_query(query):
    """Find entities mentioned in a query."""
    matches = []
    query_lower = query.lower()

    for entity_id, entity in all_entities.items():
        name = entity.get('canonical_name', '').lower()

        # Check if entity name appears in query
        if name and name in query_lower:
            matches.append({
                'entity_id': entity_id,
                'name': entity['canonical_name'],
                'score': len(name) / len(query_lower)
            })

        # Check aliases
        for alias in entity.get('aliases', []):
            if alias.lower() in query_lower:
                matches.append({
                    'entity_id': entity_id,
                    'name': entity['canonical_name'],
                    'score': len(alias) / len(query_lower)
                })

    # Deduplicate by entity_id, keep highest score
    seen = {}
    for match in matches:
        eid = match['entity_id']
        if eid not in seen or match['score'] > seen[eid]['score']:
            seen[eid] = match

    return list(seen.values())
```

### Graph Context Generation

We format graph knowledge for display or LLM consumption:

```python
def generate_graph_context(query, max_entities=5, max_relations=10):
    """Generate formatted graph context for a query."""
    # Find mentioned entities
    mentions = extract_entities_from_query(query)[:max_entities]

    if not mentions:
        return ""

    context_parts = ["## Knowledge Graph Context\n"]

    for mention in mentions:
        entity = get_entity(mention['entity_id'])
        related = get_related_entities(mention['entity_id'])[:max_relations]

        # Format entity
        context_parts.append(f"### {entity['canonical_name']} ({entity['type']})")

        if entity.get('description'):
            context_parts.append(f"**Description:** {entity['description']}")
        if entity.get('role'):
            context_parts.append(f"**Role:** {entity['role']}")

        # Format relationships
        if related:
            context_parts.append("\n**Relationships:**")
            for rel in related:
                arrow = "→" if rel['direction'] == 'outgoing' else "←"
                context_parts.append(
                    f"- {arrow} [{rel['relationship']}] "
                    f"{rel['canonical_name']} ({rel['entity_type']})"
                )

        context_parts.append("")

    return "\n".join(context_parts)
```

### Combining Results

For RAG applications, we combine graph context with vector results:

```python
def build_rag_context(query, top_k=5):
    """Build context for an LLM from both sources."""

    # Structured knowledge
    graph_context = generate_graph_context(query)

    # Relevant passages
    vector_results = vector_search(query, top_k=top_k)
    passages = "\n\n".join([
        f"[Source: {r['source']}]\n{r['text']}"
        for r in vector_results
    ])

    # Combined context for LLM
    return f"""
## Structured Knowledge
{graph_context}

## Relevant Passages
{passages}
"""
```

---

## Performance Characteristics

### Query Latency

| Mode | Typical Latency | Bottleneck |
|------|-----------------|------------|
| Vector | 50-100ms | Embedding + search |
| Graph | 10-50ms | In-memory traversal |
| Hybrid | 100-200ms | Both combined |

First query is slower (~2s) due to model and data loading.

### Memory Usage

| Component | Memory |
|-----------|--------|
| Embedding model | ~400 MB |
| Vector index | ~200 MB |
| Graph (loaded) | ~50 MB |
| **Total** | ~650 MB |

All fits comfortably in memory for fast queries.

---

## Advanced Patterns

### Graph-Guided Vector Search

Use graph to expand query terms before vector search:

```python
def graph_expanded_search(query):
    # Find entities in query
    entities = extract_entities_from_query(query)

    # Get related entities
    expanded_terms = [query]
    for e in entities:
        related = get_related_entities(e['entity_id'], limit=3)
        for r in related:
            expanded_terms.append(r['canonical_name'])

    # Search with expanded query
    expanded_query = " ".join(expanded_terms)
    return vector_search(expanded_query)
```

Example: "Austin's projects" → searches for "Austin's projects Elanah Practice Interviews"

### Vector-Guided Entity Discovery

Use vector search to find entities not explicitly mentioned:

```python
def discover_entities(query):
    # Vector search first
    passages = vector_search(query, top_k=20)

    # Extract entities mentioned in results
    discovered = set()
    for passage in passages:
        entities = extract_entities_from_text(passage['text'])
        discovered.update(entities)

    return list(discovered)
```

Example: "pricing strategy" might surface passages mentioning "Josh Reini" and "Elanah" even though the query didn't name them.

### Multi-Hop Graph Queries

Answer questions requiring multiple relationship traversals:

```python
def multi_hop_query(start_entity, hops=2):
    """Find entities within N hops of start."""
    visited = {start_entity}
    frontier = [start_entity]

    for _ in range(hops):
        next_frontier = []
        for entity_id in frontier:
            related = get_related_entities(entity_id)
            for r in related:
                if r['entity_id'] not in visited:
                    visited.add(r['entity_id'])
                    next_frontier.append(r['entity_id'])
        frontier = next_frontier

    return [get_entity(eid) for eid in visited]
```

Example: "Who is connected to Austin within 2 hops?" returns Austin's direct connections plus their connections.

---

## Command Reference

```bash
# Vector search (default)
python search.py "your query"
python search.py "query" table:meetings
python search.py "query" project:elanah

# Graph search
python search.py "entity name" --graph
python search.py "query" --graph type:PERSON

# Hybrid search
python search.py "complex query" --hybrid

# Graph statistics
python search.py --stats
```

### Filter Reference

| Filter | Modes | Values |
|--------|-------|--------|
| `table:` | vector, hybrid | meetings, logs |
| `project:` | vector | elanah, practice_interviews, etc. |
| `content:` | vector | summary, transcript |
| `month:` | vector | 2025-08, 2026-01, etc. |
| `type:` | graph | PERSON, PROJECT, TECHNOLOGY, etc. |

---

## When Hybrid Shines

Hybrid search excels for questions that:

1. **Name entities but need context**
   - "What has Josh said about the roadmap?"
   - Graph identifies Josh, vector finds his statements

2. **Ask about relationships and details**
   - "How does Austin's work connect to the consciousness projects?"
   - Graph shows connections, vector provides depth

3. **Require both facts and narrative**
   - "Summarize Matt's involvement in Elanah"
   - Graph shows role, vector finds discussions

4. **Explore unfamiliar territory**
   - "Tell me about the Elanah project"
   - Graph gives structure, vector gives substance

---

## Summary

Hybrid retrieval combines two complementary approaches:

| Aspect | Vector Search | Knowledge Graph |
|--------|---------------|-----------------|
| **Finds** | Similar passages | Connected entities |
| **Answers** | "What's like this?" | "What's connected?" |
| **Strength** | Semantic nuance | Structural clarity |
| **Weakness** | No relationships | No semantic similarity |

Together, they answer questions neither handles alone:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│    Vector: "Here are passages about Austin's work..."       │
│                           +                                 │
│    Graph: "Austin leads Elanah, uses Claude, works          │
│            with Matt..."                                    │
│                           =                                 │
│    Comprehensive answer with facts AND context              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

This is Hybrid RAG: the best of both worlds.

---

## Series Complete

This concludes the four-part Hybrid RAG training series:

1. **[Hybrid RAG Explained](https://brief.fractallabs.dev/hybrid-rag-explained)** — Concepts and motivation
2. **[Building the Vector Database](https://brief.fractallabs.dev/building-vector-database)** — Chunking, embedding, storage
3. **[Building the Knowledge Graph](https://brief.fractallabs.dev/building-knowledge-graph)** — Extraction, resolution, relationships
4. **Hybrid Retrieval** — Combining both approaches (you are here)

You now understand how to transform unstructured documents into searchable knowledge, and how to query that knowledge in multiple ways.

---

*"The question isn't vector OR graph. It's knowing when each shines — and having both ready when you need them."*
