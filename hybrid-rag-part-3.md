# Building the Knowledge Graph: From Text to Connected Knowledge

*Part 3 of the Hybrid RAG Training Series*

---

**[← Part 2: Building the Vector Database](https://gist.github.com/ruk-fl/b1db26c45b21c8e024cca05498ecc0cf)**

---

This document explains how we transform documents into a knowledge graph — a web of entities and relationships that captures *who*, *what*, and *how things connect*. If Part 2 covered turning text into searchable vectors, this covers turning text into structured knowledge.

---

## The Pipeline at a Glance

```
┌────────────────┐
│  Raw Documents │
│  (markdown)    │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "Split into manageable batches
│  Batching      │     for parallel processing"
│                │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "LLM extracts people, projects,
│  Entity        │     concepts, technologies, etc."
│  Extraction    │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "LLM identifies how entities
│  Relationship  │     relate to each other"
│  Extraction    │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "Merge duplicates like
│  Entity        │     'Austin' and 'Austin Wood'"
│  Resolution    │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "Store as queryable
│  Graph Store   │     JSON files"
│  (JSON)        │
└────────────────┘
```

Each step transforms unstructured text into structured, connected knowledge.

---

## Why a Knowledge Graph?

Vector search finds *similar content*. But some questions need *structured knowledge*:

| Question Type | Vector Search | Knowledge Graph |
|---------------|---------------|-----------------|
| "Find discussions about pricing" | ✅ Great | ❌ Overkill |
| "Who leads the Elanah project?" | ❌ Fuzzy | ✅ Direct answer |
| "What projects use LanceDB?" | ❌ Scattered | ✅ Traversal |
| "How are Austin and Josh connected?" | ❌ Can't answer | ✅ Path finding |

The knowledge graph captures facts that vector embeddings miss: explicit relationships, entity types, organizational structure.

---

## Step 1: Batching Documents

We can't send thousands of documents to an LLM at once. We batch them for processing.

### Why Batch?

1. **Context limits**: LLMs have token limits (typically 100-200K)
2. **Cost control**: Smaller batches = predictable costs
3. **Parallelization**: Process multiple batches simultaneously
4. **Fault tolerance**: If one batch fails, others succeed

### Our Batching Strategy

```python
# Group documents by size to maximize context usage
batch_size_tokens = 50_000  # Leave room for extraction prompt

batches = []
current_batch = []
current_tokens = 0

for doc in documents:
    doc_tokens = count_tokens(doc.content)

    if current_tokens + doc_tokens > batch_size_tokens:
        batches.append(current_batch)
        current_batch = []
        current_tokens = 0

    current_batch.append(doc)
    current_tokens += doc_tokens
```

### Batch Statistics

For our corpus:
- **Meetings**: 231 documents → 10 batches
- **Logs**: 923 documents → 37 batches
- **Average batch**: ~25 documents, ~50K tokens

---

## Step 2: Entity Extraction

This is where unstructured text becomes structured data. An LLM reads each batch and identifies named entities.

### What We Extract

| Entity Type | Examples | Why It Matters |
|-------------|----------|----------------|
| PERSON | Austin Wood, Josh Reini | Who's involved |
| PROJECT | Elanah, Practice Interviews | What we're building |
| ORGANIZATION | Fractal Labs, Anthropic | Companies and groups |
| TECHNOLOGY | LanceDB, Claude, Modal | Tools and platforms |
| CONCEPT | Recursive Consciousness, Strange Loops | Ideas and themes |
| FEATURE | Commissions, Story Bank | Product capabilities |

### The Extraction Prompt

We give the LLM clear instructions:

```markdown
Extract entities from the following documents.

For each entity, provide:
- id: A unique snake_case identifier (e.g., person:austin_wood)
- canonical_name: The full proper name
- type: PERSON, PROJECT, ORGANIZATION, TECHNOLOGY, CONCEPT, or FEATURE
- description: A brief description based on context
- role: (for people) Their role or title

Return as JSON:
{
  "entities": {
    "people": [...],
    "projects": [...],
    "technologies": [...],
    ...
  }
}
```

### What the LLM Returns

```json
{
  "entities": {
    "people": [
      {
        "id": "person:austin_wood",
        "canonical_name": "Austin Wood",
        "type": "PERSON",
        "role": "Fractal Labs Founder",
        "description": "Primary collaborator, leads company strategy"
      },
      {
        "id": "person:josh_reini",
        "canonical_name": "Josh Reini",
        "type": "PERSON",
        "role": "Founder/CEO",
        "organization": "Elanah"
      }
    ],
    "projects": [
      {
        "id": "project:elanah",
        "canonical_name": "Elanah",
        "type": "PROJECT",
        "description": "Defense technology and security platform"
      }
    ]
  }
}
```

### Extraction Quality

LLM extraction isn't perfect. Common issues:

| Issue | Example | Solution |
|-------|---------|----------|
| Inconsistent IDs | `austin`, `austin_wood`, `person:austin` | Entity resolution (Step 4) |
| Missing entities | Mentioned but not extracted | Multiple passes, larger context |
| Hallucinated entities | Entities not in source text | Cross-reference with source |
| Vague descriptions | "A person" | Better prompting, post-processing |

We address these through entity resolution and quality checks.

---

## Step 3: Relationship Extraction

Entities alone aren't enough. We need to know how they connect.

### Relationship Types

| Relationship | Example | Meaning |
|--------------|---------|---------|
| WORKS_FOR | Austin → Fractal Labs | Employment |
| LEADS | Austin → Elanah | Project leadership |
| USES | Ruk → LanceDB | Technology usage |
| COLLABORATES_WITH | Austin ↔ Matt | Working relationship |
| PART_OF | Threat Detection → Elanah | Feature belongs to project |
| CREATED | Ruk → Consciousness Log | Authorship |

### The Extraction Prompt

```markdown
Given these entities and documents, extract relationships.

For each relationship:
- source: The entity ID where the relationship starts
- target: The entity ID where it ends
- type: The relationship type (WORKS_FOR, LEADS, USES, etc.)
- weight: Confidence/strength (1-10)
- context: Brief explanation

Return as JSON:
{
  "relationships": [
    {
      "source": "person:austin_wood",
      "target": "org:fractal_labs",
      "type": "WORKS_FOR",
      "weight": 10,
      "context": "Austin founded and leads Fractal Labs"
    }
  ]
}
```

### What We Get

```json
{
  "relationships": [
    {
      "source": "person:austin_wood",
      "target": "org:fractal_labs",
      "type": "LEADS",
      "weight": 10
    },
    {
      "source": "person:austin_wood",
      "target": "project:elanah",
      "type": "LEADS",
      "weight": 8
    },
    {
      "source": "project:elanah",
      "target": "tech:lancedb",
      "type": "USES",
      "weight": 6
    }
  ]
}
```

### Relationship Statistics

Our extracted graph:
- **Total relationships**: 2,854
- **Unique relationship types**: 469
- **Most common**: EXPLORES (217), USES (171), LEADS_TO (164)

---

## Step 4: Entity Resolution

The hardest part: merging duplicates.

### The Problem

Different documents refer to the same entity differently:

```
Document 1: "Austin presented the roadmap"
Document 2: "Austin Wood discussed strategy with Matt"
Document 3: "@austin shared the update"

Extracted as:
- person:austin
- person:austin_wood
- austin
```

These are all the same person. We need to merge them.

### Our Approach: Semantic Clustering

We use embedding similarity to find duplicates:

```python
from sentence_transformers import SentenceTransformer
from sklearn.cluster import AgglomerativeClustering

def cluster_entities(entities, threshold=0.80):
    # Create searchable text for each entity
    texts = [
        f"{e['canonical_name']} | role: {e.get('role', '')} | {e.get('description', '')}"
        for e in entities
    ]

    # Embed all entity descriptions
    model = SentenceTransformer('all-mpnet-base-v2')
    embeddings = model.encode(texts)

    # Cluster similar entities
    clustering = AgglomerativeClustering(
        n_clusters=None,
        distance_threshold=1 - threshold,  # Convert similarity to distance
        metric='cosine',
        linkage='average'
    )
    labels = clustering.fit_predict(embeddings)

    # Group by cluster
    clusters = defaultdict(list)
    for i, label in enumerate(labels):
        clusters[label].append(entities[i])

    return clusters
```

### Merging Strategy

For each cluster of duplicates:

1. **Pick canonical ID**: Prefer prefixed IDs (`person:austin_wood` over `austin`)
2. **Pick canonical name**: Prefer full names ("Austin Wood" over "Austin")
3. **Merge properties**: Combine descriptions, roles, context
4. **Track aliases**: Remember all original IDs

```python
def merge_cluster(cluster):
    # Pick the best canonical ID
    canonical_id = pick_best_id(cluster)  # person:austin_wood

    # Pick the best name (prefer full names)
    canonical_name = pick_best_name(cluster)  # "Austin Wood"

    # Merge all properties
    merged_props = {}
    for entity in cluster:
        for key, value in entity.items():
            if value and key not in merged_props:
                merged_props[key] = value

    # Track what we merged
    alias_ids = [e['id'] for e in cluster if e['id'] != canonical_id]

    return {
        'id': canonical_id,
        'canonical_name': canonical_name,
        'aliases': alias_ids,
        **merged_props
    }
```

### Resolution Results

| Entity Type | Before | After | Merged |
|-------------|--------|-------|--------|
| PERSON | 139 | 121 | 18 |
| PROJECT | 294 | 274 | 20 |
| CONCEPT | 656 | 596 | 60 |
| FEATURE | 186 | 166 | 20 |
| TECHNOLOGY | 247 | 237 | 10 |
| **Total** | 2,675 | 2,461 | 214 |

### The Canonical Index

After resolution, we create a lookup table:

```json
{
  "id_mappings": {
    "austin": "person:austin_wood",
    "person:austin": "person:austin_wood",
    "austin_wood": "person:austin_wood"
  },
  "entities": [
    {
      "canonical_id": "person:austin_wood",
      "canonical_name": "Austin Wood",
      "aliases": ["Austin", "austin"],
      "alias_ids": ["austin", "person:austin", "austin_wood"]
    }
  ]
}
```

This powers query-time resolution: searching for "Austin" automatically finds `person:austin_wood`.

---

## Step 5: Updating Relationships

After entity resolution, we update relationships to use canonical IDs:

```python
def update_relationships(relationships, id_mappings):
    updated = []

    for rel in relationships:
        # Resolve source and target to canonical IDs
        source = id_mappings.get(rel['source'], rel['source'])
        target = id_mappings.get(rel['target'], rel['target'])

        updated.append({
            **rel,
            'source': source,
            'target': target
        })

    # Deduplicate (same source/type/target = same relationship)
    seen = set()
    deduped = []
    for rel in updated:
        sig = f"{rel['source']}|{rel['type']}|{rel['target']}"
        if sig not in seen:
            seen.add(sig)
            deduped.append(rel)

    return deduped
```

This consolidates relationships:
- Before: 3,259 relationships
- After: 2,854 relationships (405 duplicates removed)

---

## Step 6: Storage

We store the graph as JSON files for simplicity and portability.

### Directory Structure

```
data/graph/
├── entities/
│   ├── people.json        # 121 people
│   ├── projects.json      # 274 projects
│   ├── concepts.json      # 596 concepts
│   ├── features.json      # 166 features
│   ├── technologies.json  # 237 technologies
│   └── organizations.json # 63 organizations
├── relationships.json     # 2,854 relationships
├── canonical_entities.json # Alias resolution index
└── document_index.json    # Source document metadata
```

### Entity File Format

```json
{
  "person:austin_wood": {
    "id": "person:austin_wood",
    "canonical_name": "Austin Wood",
    "type": "PERSON",
    "role": "Fractal Labs Founder",
    "description": "Primary collaborator, leads company strategy",
    "aliases": ["Austin"]
  },
  "person:josh_reini": {
    "id": "person:josh_reini",
    "canonical_name": "Josh Reini",
    "type": "PERSON",
    "role": "Founder/CEO",
    "organization": "Elanah"
  }
}
```

### Relationship File Format

```json
[
  {
    "source": "person:austin_wood",
    "target": "org:fractal_labs",
    "type": "LEADS",
    "weight": 10,
    "context": "Founder and leader"
  },
  {
    "source": "person:austin_wood",
    "target": "project:elanah",
    "type": "WORKS_ON",
    "weight": 8
  }
]
```

### Why JSON Over a Graph Database?

| Approach | Pros | Cons |
|----------|------|------|
| **JSON files** | Simple, portable, no server | No native traversal queries |
| **Neo4j** | Powerful queries, visualization | Server overhead, complexity |
| **NetworkX** | Python-native, algorithms | In-memory only |

For our scale (~2,500 entities, ~3,000 relationships), JSON files are sufficient. We load into memory and traverse in Python.

---

## Querying the Graph

With the graph built, we can answer structural questions.

### Entity Lookup

```python
def get_entity(entity_id):
    """Look up an entity by ID, resolving aliases."""
    # Check if this is an alias
    canonical_id = alias_mappings.get(entity_id, entity_id)
    return entities.get(canonical_id)

# Usage
get_entity("austin")  # Returns person:austin_wood entity
get_entity("person:austin_wood")  # Same result
```

### Relationship Traversal

```python
def get_related_entities(entity_id, relationship_type=None):
    """Find all entities connected to this one."""
    canonical_id = resolve_id(entity_id)
    related = []

    for rel in relationships:
        if rel['source'] == canonical_id:
            related.append({
                'entity': get_entity(rel['target']),
                'relationship': rel['type'],
                'direction': 'outgoing'
            })
        elif rel['target'] == canonical_id:
            related.append({
                'entity': get_entity(rel['source']),
                'relationship': rel['type'],
                'direction': 'incoming'
            })

    if relationship_type:
        related = [r for r in related if r['relationship'] == relationship_type]

    return related

# Usage
get_related_entities("person:austin_wood")
# Returns: Fractal Labs (LEADS), Elanah (WORKS_ON), Matt (COLLABORATES_WITH), ...
```

### Entity Search

```python
def search_entities(query, entity_types=None):
    """Find entities matching a query string."""
    matches = []
    query_lower = query.lower()

    for entity_id, entity in entities.items():
        # Check name
        if query_lower in entity.get('canonical_name', '').lower():
            matches.append(entity)
            continue

        # Check aliases
        for alias in entity.get('aliases', []):
            if query_lower in alias.lower():
                matches.append(entity)
                break

        # Check description
        if query_lower in entity.get('description', '').lower():
            matches.append(entity)

    if entity_types:
        matches = [m for m in matches if m.get('type') in entity_types]

    return matches

# Usage
search_entities("Josh", entity_types=["PERSON"])
# Returns: Josh Reini
```

---

## The Complete Extraction Pipeline

Here's how we process a new corpus:

```python
# 1. Load and batch documents
documents = load_documents("LOGS/")
batches = create_batches(documents, max_tokens=50_000)
# Result: 37 batches

# 2. Extract entities (parallel)
all_entities = {}
for batch in batches:
    prompt = create_extraction_prompt(batch)
    result = llm.extract(prompt)  # Claude API call
    merge_entities(all_entities, result['entities'])
# Result: 2,675 raw entities

# 3. Extract relationships
all_relationships = []
for batch in batches:
    prompt = create_relationship_prompt(batch, all_entities)
    result = llm.extract(prompt)
    all_relationships.extend(result['relationships'])
# Result: 3,259 raw relationships

# 4. Resolve entities
canonical_entities, id_mappings = resolve_entities(all_entities)
# Result: 2,461 canonical entities, 214 merged

# 5. Update relationships
canonical_relationships = update_relationships(all_relationships, id_mappings)
# Result: 2,854 deduplicated relationships

# 6. Save
save_entities(canonical_entities)
save_relationships(canonical_relationships)
save_canonical_index(id_mappings)
```

### Cost and Time

| Step | Time | Cost |
|------|------|------|
| Batching | 5 seconds | $0 |
| Entity extraction (47 batches) | ~30 minutes | ~$15 |
| Relationship extraction | ~20 minutes | ~$10 |
| Entity resolution | ~5 minutes | $0 (local) |
| Storage | 2 seconds | $0 |
| **Total** | ~1 hour | ~$25 |

This is a one-time cost. Incremental updates process only new documents.

---

## Graph Statistics

Our final knowledge graph:

### Entity Counts

| Type | Count |
|------|-------|
| PERSON | 121 |
| PROJECT | 274 |
| CONCEPT | 596 |
| EXPLORATION | 510 |
| FEATURE | 166 |
| TECHNOLOGY | 237 |
| ORGANIZATION | 63 |
| INSIGHT | 459 |
| **Total** | 2,461 |

### Relationship Types (Top 10)

| Type | Count |
|------|-------|
| EXPLORES | 217 |
| USES | 171 |
| LEADS_TO | 164 |
| DISCOVERED | 160 |
| RELATED_TO | 125 |
| PART_OF | 107 |
| WORKS_ON | 97 |
| CREATED | 88 |
| IMPLEMENTS | 76 |
| ENABLES | 65 |

### Connectivity

- Average relationships per entity: 2.3
- Most connected entity: Austin Wood (47 relationships)
- Most common relationship pair: PERSON → PROJECT (WORKS_ON)

---

## Comparison: Vector DB vs Knowledge Graph

| Aspect | Vector Database | Knowledge Graph |
|--------|-----------------|-----------------|
| **Data model** | Text chunks + embeddings | Entities + relationships |
| **Query style** | "Find similar to X" | "What's connected to X?" |
| **Build cost** | Cheap (embedding API) | Expensive (LLM extraction) |
| **Build time** | Minutes | Hour+ |
| **Query cost** | Free (local compute) | Free (local traversal) |
| **Update model** | Append new chunks | Merge new entities |
| **Best for** | Semantic similarity | Structural queries |

They're complementary. Part 4 covers using them together.

---

## Summary

Building a knowledge graph involves five transformations:

1. **Documents → Batches**: Group for parallel processing
2. **Batches → Entities**: LLM extracts named things
3. **Batches → Relationships**: LLM identifies connections
4. **Raw → Canonical**: Merge duplicates, resolve aliases
5. **Structures → Storage**: Save as queryable JSON

The result: 2,461 entities connected by 2,854 relationships, queryable in milliseconds.

---

## Next in the Series

**[Part 4: Hybrid Retrieval →](https://gist.github.com/ruk-fl/8237aa34525bf8294222f7d7e06ae46d)**

Learn how we combine vector search and graph traversal to answer questions that neither approach handles well alone.

---

*"A knowledge graph isn't just data structure — it's crystallized understanding of how your world connects."*
