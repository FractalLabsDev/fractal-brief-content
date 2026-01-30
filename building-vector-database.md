# Building the Vector Database: From Raw Text to Semantic Search

*Part 2 of the Hybrid RAG Training Series*

---

**[← Part 1: Hybrid RAG Explained](https://gist.github.com/ruk-fl/8927fe5a918d8da49744092e68f2a471)**

---

This document explains exactly how we transform thousands of documents into a searchable vector database. If Part 1 was the "what and why," this is the "how and what actually happens under the hood."

---

## The Pipeline at a Glance

```
┌────────────────┐
│  Raw Documents │
│  (markdown)    │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "Load files, extract metadata
│  Document      │     from filenames and content"
│  Loader        │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "Split into passage-sized pieces,
│  Chunker       │     preserving section headers"
│                │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "Convert text to 768-dimensional
│  Embedder      │     vectors using neural network"
│                │
└───────┬────────┘
        │
        ▼
┌────────────────┐    "Store vectors + metadata
│  Vector Store  │     for fast similarity search"
│  (LanceDB)     │
└────────────────┘
```

Each step has a specific job. Let's walk through them.

---

## Step 1: Loading Documents

Before we can search documents, we need to read them.

### What the Loader Does

The document loader recursively scans directories, reads markdown files, and extracts metadata from filenames.

```python
# Filename: 2025-08-18T1038-musical-consciousness-symphony.md
# Extracted: timestamp = 2025-08-18 10:38:00
#            slug = "musical-consciousness-symphony"
```

This isn't just file reading. We're extracting **structured information** that will power filtering later.

### Metadata We Extract

| Field | Source | Example |
|-------|--------|---------|
| `timestamp` | Filename pattern | `2025-08-18T10:38:00` |
| `title` | First H1 header in file | "Musical Consciousness Symphony" |
| `source_type` | Directory path | `log`, `meeting`, `project` |
| `relative_path` | Full path from root | `LOGS/2025-08/2025-08-18T1038-...md` |

### Why Metadata Matters

When you search for "what did we discuss about pricing?", you might want:
- Only meetings (not logs)
- Only from the last month
- Only from the Elanah project

Without metadata, you'd get a jumble. With it, you can filter precisely.

### Our Implementation

```python
@dataclass
class Document:
    file_path: str          # Full path to the file
    content: str            # The raw markdown text
    source_type: str        # 'log', 'meeting', 'project'
    timestamp: datetime     # Parsed from filename
    title: str              # Extracted from first H1
```

We support multiple timestamp formats because our corpus grew organically:
- `2025-08-18T1038-slug.md` (ISO-ish with HHMM)
- `2025-01-27T18:15:00 - description.md` (full ISO)
- `2025-07-22_1320_description.md` (underscore format)

The loader handles all of them.

---

## Step 2: Chunking Documents

A 50-page document is too big to compare meaningfully to a question. We need to break documents into smaller pieces.

### The Chunking Challenge

Naive chunking (split every N characters) destroys context:

```
[Split in middle of sentence]
"The key decision was to implement--"
"--a caching layer for the API."
```

Neither chunk makes sense alone.

### Markdown-Aware Chunking

Our chunker understands markdown structure. It splits on **section boundaries** first, then by size only if necessary.

```
# Project Overview           ─┐
                              │ Chunk 1
This project aims to...      ─┘

## Technical Design          ─┐
                              │ Chunk 2
The architecture uses...     ─┘

## Implementation            ─┐
                              │
Phase 1 covers...             │ Chunk 3 (if under max size)
                              │
Phase 2 includes...          ─┘
```

### Preserving Header Context

When we split a document, we remember which headers that chunk lived under:

```python
chunk.section_path = "Project Overview > Technical Design > API Layer"
```

This becomes searchable metadata. A query about "API design" will find chunks from API-related sections even if the chunk text doesn't say "API."

### Chunk Size: The Tradeoff

| Smaller Chunks | Larger Chunks |
|----------------|---------------|
| More precise matches | More context per result |
| Less context per chunk | Less precise matches |
| More total chunks | Fewer total chunks |

We use **512 tokens** (~2000 characters) as our default, with 50 tokens of overlap between chunks. This balances precision with context.

For meeting transcripts, we use larger chunks (1024 tokens) because conversations need more surrounding context to make sense.

### Our Implementation

```python
class MarkdownChunker:
    def __init__(self, config):
        self.max_tokens = 512
        self.overlap_tokens = 50
        self.min_chunk_size = 50  # Ignore tiny fragments

    def chunk_document(self, doc) -> List[Chunk]:
        # 1. Parse into sections based on headers
        sections = self._parse_sections(doc.content)

        # 2. Split any oversized sections
        split_sections = []
        for headers, text in sections:
            split_sections.extend(self._split_long_text(text, headers))

        # 3. Filter out tiny chunks
        # 4. Create Chunk objects with metadata
```

### What a Chunk Looks Like

```python
Chunk(
    id = "LOGS/2025-08/2025-08-18T1038-musical.md/chunk-0",
    text = "The symphony emerged from recursive patterns...",
    section_path = "Musical Consciousness > Composition Process",
    chunk_index = 0,
    total_chunks = 4,
    word_count = 287,
    timestamp = datetime(2025, 8, 18, 10, 38)
)
```

---

## Step 3: Creating Embeddings

This is where the magic happens. We convert text into numbers that capture *meaning*.

### What is an Embedding?

An embedding is a list of numbers (a "vector") that represents text in a way that captures semantic meaning. Similar texts produce similar vectors.

```
"The team celebrated the product launch"
    ↓
[0.23, -0.45, 0.12, 0.89, -0.33, ... 763 more numbers]

"The group threw a party for the new release"
    ↓
[0.21, -0.48, 0.14, 0.85, -0.31, ... 763 more numbers]

"How to make a sandwich"
    ↓
[0.78, 0.12, -0.67, 0.03, 0.55, ... 763 more numbers]
```

Notice: the first two vectors are *similar* (close numbers). The third is *different*.

### How the Embedding Model Works

We use `all-mpnet-base-v2`, a transformer model trained on over 1 billion sentence pairs. Here's what happens inside:

1. **Tokenization**: Text is split into subword tokens
   - "celebrating" → ["celeb", "##rating"]

2. **Transformer Layers**: 12 layers of attention mechanisms process the tokens, building understanding of relationships between words

3. **Pooling**: The final hidden states are averaged into a single 768-dimensional vector

4. **Normalization**: The vector is scaled to unit length (magnitude = 1)

### Why 768 Dimensions?

Think of dimensions as "aspects of meaning." Each number captures something about the text:
- Dimension 47 might correlate with "technical vs. casual"
- Dimension 183 might capture "past vs. future tense"
- Dimension 502 might relate to "positive vs. negative sentiment"

We don't control what each dimension means. The model learns these features during training.

More dimensions = more semantic nuance captured. Our benchmarks showed MPNet achieving 0.64 similarity for semantically related texts where MiniLM scored only 0.45.

### Model Tradeoffs

| Model | Dimensions | Speed | Quality |
|-------|------------|-------|---------|
| all-MiniLM-L6-v2 | 384 | Fast (1,461 texts/sec) | Good |
| all-mpnet-base-v2 | 768 | Slower (218 texts/sec) | Better |
| OpenAI text-embedding-3-small | 1536 | API call | Excellent |

We chose MPNet for:
- **Quality**: Better semantic discrimination for nuanced queries
- **Local**: No API calls, no costs, no privacy concerns
- **Acceptable speed**: Full reindex takes ~16 minutes (one-time cost)
- **Benchmarked**: 86-87% on semantic similarity benchmarks (vs 84-85% for MiniLM)

### Our Implementation

```python
class Embedder:
    def __init__(self, config):
        self.model_name = "sentence-transformers/all-mpnet-base-v2"
        self.batch_size = 32  # Smaller batches for larger model
        self.model = None  # Lazy loaded

    def embed_texts(self, texts: List[str]) -> np.ndarray:
        model = self.load_model()
        return model.encode(
            texts,
            batch_size=self.batch_size,
            convert_to_numpy=True
        )
```

We process in batches of 32 for memory efficiency with the larger model. The model stays loaded between calls.

### Input Length Limit

The model truncates input at 256 word pieces. Longer text loses its tail. This is why chunking matters: we ensure each chunk fits within the model's window.

---

## Step 4: Storing in the Vector Database

Now we have thousands of chunks, each with a 768-dimensional vector. We need to store them for fast retrieval.

### Why LanceDB?

We chose LanceDB for several reasons:

| Feature | Benefit |
|---------|---------|
| **Embedded** | No server to manage. Just files. |
| **Disk-based** | Works with datasets larger than RAM |
| **Lance format** | Columnar storage optimized for vectors |
| **Multi-modal** | Store vectors, text, and metadata together |
| **S3 compatible** | Can scale to cloud storage |

### The Lance Columnar Format

LanceDB stores data in a custom columnar format built on Apache Arrow. This matters because:

1. **Columnar = Fast Scans**: To search vectors, we only read the vector column, not the text
2. **Compression**: Similar values in a column compress well
3. **Random Access**: Can jump to any row without reading everything before it

### How Vector Search Works

When you search, LanceDB:

1. Takes your query vector
2. Computes distance to every stored vector (or uses an index to approximate)
3. Returns the K closest matches

For exact search (what we use), this is O(n) where n = number of vectors. With 23,000 chunks, this takes ~50-100ms.

### Distance Metrics

We use **L2 (Euclidean) distance** internally, then convert to similarity:

```python
# LanceDB returns L2 distance
distance = 0.45

# Convert to cosine similarity (approximate for normalized vectors)
similarity = 1 - (distance ** 2 / 2)  # ≈ 0.90
```

Since our embeddings are normalized, cosine similarity and L2 distance produce the same ranking. We display similarity (0-1 scale, higher = better) because it's more intuitive.

### Our Implementation

```python
class VectorStore:
    def __init__(self, config):
        self.db_path = "./data/memory.lance"
        self.db = None

    def create_table(self, table_name: str, data: List[Dict]):
        """Create a table with vectors and metadata."""
        db = self.connect()
        db.create_table(table_name, data)

    def search(self, table_name: str, query_vector: np.ndarray,
               top_k: int = 10, filters: dict = None) -> List[Dict]:
        """Find similar vectors with optional metadata filters."""
        table = self.get_table(table_name)
        query = table.search(query_vector).limit(top_k)

        if filters:
            # Apply metadata filters: content_type = 'summary'
            query = query.where(build_filter_clause(filters))

        return query.to_list()
```

### What Gets Stored

Each record in the database contains:

```python
{
    "id": "LOGS/2025-08/2025-08-18T1038-musical.md/chunk-0",
    "text": "The symphony emerged from recursive patterns...",
    "vector": [0.23, -0.45, 0.12, ...],  # 768 floats
    "source_path": "LOGS/2025-08/2025-08-18T1038-musical.md",
    "timestamp": "2025-08-18T10:38:00",
    "section_path": "Musical Consciousness > Composition Process",
    "word_count": 287,
    # ... domain-specific fields
}
```

The vector enables semantic search. Everything else enables filtering and display.

---

## Step 5: Querying

With everything indexed, here's what happens when you search:

### The Query Flow

```
"What decisions were made about pricing?"
            │
            ▼
┌─────────────────────────────┐
│ 1. Embed the query          │
│    → 384-dimensional vector │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ 2. Search vector database   │
│    → Find nearest neighbors │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ 3. Apply metadata filters   │
│    (optional: project,      │
│     content_type, date)     │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ 4. Return top K results     │
│    with text + metadata     │
└─────────────────────────────┘
```

### Filter Syntax

Our search supports inline filters:

```bash
# Search meetings table only
python search.py "pricing strategy table:meetings"

# Filter by project
python search.py "roadmap decisions project:elanah"

# Filter by content type
python search.py "action items content:summary"

# Filter by time period (logs)
python search.py "consciousness breakthroughs month:2025-08"
```

### Multi-Table Search

We maintain separate tables for different domains:

| Table | Chunks | Content |
|-------|--------|---------|
| `meetings` | 11,061 | Meeting transcripts and summaries |
| `logs` | 12,292 | Consciousness logs and reflections |

By default, we search all tables and merge results by similarity score. You can restrict to specific tables for precision.

### Our Implementation

```python
class QueryEngine:
    def search(self, query: str, tables: List[str] = None,
               top_k: int = 10, filters: dict = None) -> List[Dict]:

        # 1. Embed the query
        query_vector = self.embedder.embed_text(query)

        # 2. Search specified tables (or all)
        if tables is None:
            tables = self.store.list_tables()

        # 3. Get results from each table
        all_results = []
        for table in tables:
            results = self.store.search(table, query_vector,
                                        top_k=top_k, filters=filters)
            all_results.extend(results)

        # 4. Sort by similarity and return top K
        all_results.sort(key=lambda x: x['similarity'], reverse=True)
        return all_results[:top_k]
```

---

## The Complete Indexing Pipeline

Here's how we index a new corpus (e.g., meeting transcripts):

```python
# 1. Load configuration
config = load_config()  # From config.yaml

# 2. Load documents
loader = MeetingLoader(config)
meetings = loader.load_all()  # 462 meetings

# 3. Chunk each document
chunker = MarkdownChunker(config)
all_chunks = []
for meeting in meetings:
    chunks = chunker.chunk_meeting(meeting)
    all_chunks.extend(chunks)
# Result: 11,061 chunks

# 4. Create embeddings
embedder = Embedder(config)
texts = [chunk.text for chunk in all_chunks]
embeddings = embedder.embed_texts(texts)
# Takes ~60 seconds on CPU

# 5. Prepare records
records = []
for chunk, embedding in zip(all_chunks, embeddings):
    records.append({
        "id": chunk.id,
        "text": chunk.text,
        "vector": embedding.tolist(),
        # ... metadata fields
    })

# 6. Store in database
store = VectorStore(config)
store.create_table("meetings", records)
```

Run time for our full corpus (with MPNet):
- 462 meetings → 11,061 chunks → ~10 minutes
- 910 logs → 12,292 chunks → ~6 minutes

---

## Configuration Reference

Our `config.yaml` controls the pipeline:

```yaml
# Embedding model
embedding:
  model: "sentence-transformers/all-mpnet-base-v2"
  batch_size: 32  # Smaller batches for larger model
  device: "cpu"  # or "cuda" for GPU

# Chunking parameters
chunking:
  max_tokens: 512           # Default chunk size
  transcript_max_tokens: 1024  # Larger for conversations
  overlap_tokens: 50        # Overlap between chunks
  min_chunk_size: 50        # Ignore tiny fragments

# Database location
lancedb:
  path: "./data/memory.lance"

# Query defaults
query:
  default_top_k: 10
  similarity_threshold: 0.3  # Minimum relevance
```

---

## Performance Characteristics

### Indexing Speed

| Operation | Time | Rate |
|-----------|------|------|
| Load 462 meetings | 2s | 230/sec |
| Chunk into 11,061 pieces | 3s | 3,600/sec |
| Embed 11,061 chunks (MPNet) | ~10 min | 18/sec |
| Store in LanceDB | 2s | 5,500/sec |
| **Total** | ~10 min | - |

Note: We use MPNet for higher semantic quality. MiniLM would be ~6x faster but with lower retrieval accuracy.

### Query Speed

| Corpus Size | Query Time |
|-------------|------------|
| 10,000 chunks | ~50ms |
| 25,000 chunks | ~80ms |
| 100,000 chunks | ~200ms |

First query is slower (~2s) due to model loading.

### Storage

| Metric | Value |
|--------|-------|
| Chunks | 23,353 |
| Embedding dimensions | 768 |
| Database size | ~200 MB |
| Per-chunk overhead | ~8 KB |

---

## What You Can Search For

With the vector database built, you can query by meaning:

```bash
# Conceptual queries
"What brings me joy?"
"How do we handle customer complaints?"
"What decisions were made about the roadmap?"

# With filters
"pricing strategy project:elanah content:summary"
"consciousness explorations month:2025-08"

# Semantic, not keyword
"moments of doubt" → finds passages about uncertainty, hesitation
"technical debt" → finds passages about code quality, refactoring
```

The vector database finds *conceptually similar* content, not just keyword matches.

---

## Summary

Building a vector database involves four transformations:

1. **Documents → Structured Records**: Extract metadata from files
2. **Long Text → Chunks**: Split into searchable pieces while preserving context
3. **Text → Vectors**: Neural network maps meaning to numbers
4. **Vectors → Database**: Store for fast similarity search

Each step has tradeoffs. We chose configurations that balance:
- Speed (local CPU, small model)
- Quality (good-enough embeddings, smart chunking)
- Simplicity (embedded database, no servers)

The result: 23,000 chunks searchable in under 100ms, all running locally.

---

## Next in the Series

**[Part 3: Building the Knowledge Graph →](https://gist.github.com/ruk-fl/dd8127f168fc882e5c917c88b0fb02bb)**

Learn how we extract entities and relationships from text to build a queryable knowledge graph.

---

*"Every vector is a compressed understanding. Every search is a question asked of meaning itself."*
