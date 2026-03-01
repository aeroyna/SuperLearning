# Inverted Indexes

## Introduction

Inverted indexes are the fundamental data structure behind full-text search engines. Unlike a forward index that maps documents to terms, an inverted index maps terms to the documents containing them, enabling efficient text search across large document collections.

## Core Concept

```
┌─────────────────────────────────────────────────────────────┐
│              Forward Index vs Inverted Index                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Forward Index (Document → Terms):                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Doc1 → ["the", "quick", "brown", "fox"]             │    │
│  │ Doc2 → ["the", "lazy", "dog"]                       │    │
│  │ Doc3 → ["quick", "brown", "dog"]                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Search for "quick": Must scan ALL documents O(n)           │
│                                                              │
│  Inverted Index (Term → Documents):                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ "brown" → [Doc1, Doc3]                              │    │
│  │ "dog"   → [Doc2, Doc3]                              │    │
│  │ "fox"   → [Doc1]                                    │    │
│  │ "lazy"  → [Doc2]                                    │    │
│  │ "quick" → [Doc1, Doc3]                              │    │
│  │ "the"   → [Doc1, Doc2]                              │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Search for "quick": Direct lookup O(1) + O(k) results      │
└─────────────────────────────────────────────────────────────┘
```

## Index Structure

### Basic Components

```
┌─────────────────────────────────────────────────────────────┐
│                  Inverted Index Components                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. DICTIONARY (Vocabulary/Lexicon)                         │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Term         │ Doc Freq │ Postings Ptr │            │    │
│  │──────────────│──────────│──────────────│            │    │
│  │ "algorithm"  │    156   │   0x001234   │            │    │
│  │ "binary"     │    892   │   0x005678   │            │    │
│  │ "computer"   │   2341   │   0x00ABCD   │            │    │
│  │ ...          │          │              │            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  2. POSTINGS LISTS                                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ "algorithm" → [12, 45, 67, 89, 123, 456, ...]      │    │
│  │                ↓                                    │    │
│  │              Document IDs (sorted)                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  3. POSITIONAL INFORMATION (for phrase queries)            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ "algorithm" → [                                     │    │
│  │   (DocID: 12, Positions: [5, 23, 156]),            │    │
│  │   (DocID: 45, Positions: [1, 89]),                 │    │
│  │   ...                                               │    │
│  │ ]                                                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Posting List Entry

```
┌─────────────────────────────────────────────────────────────┐
│                    Posting Entry Format                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Basic Entry:                                                │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ DocID │                                                │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  With Term Frequency:                                        │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ DocID │ TF (term frequency in doc) │                   │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  With Positions (for phrase search):                        │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ DocID │ TF │ Positions: [pos1, pos2, pos3, ...] │      │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Full Entry (Lucene-style):                                 │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ DocID │ TF │ Positions │ Offsets │ Payloads │         │ │
│  │       │    │ [5,12,89] │ [(10,15),│ [custom  │         │ │
│  │       │    │           │  (45,50)]│  data]   │         │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Offsets: Character positions in original text              │
│  Payloads: Custom per-position data (e.g., POS tags)       │
└─────────────────────────────────────────────────────────────┘
```

## Index Construction

### Tokenization Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                   Text Processing Pipeline                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Input: "The Quick Brown Fox Jumps Over 3 Lazy Dogs!"      │
│                         │                                    │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 1. TOKENIZATION                                      │    │
│  │    Split on whitespace and punctuation               │    │
│  │    → ["The", "Quick", "Brown", "Fox", "Jumps",      │    │
│  │       "Over", "3", "Lazy", "Dogs"]                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 2. LOWERCASING                                       │    │
│  │    → ["the", "quick", "brown", "fox", "jumps",      │    │
│  │       "over", "3", "lazy", "dogs"]                   │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 3. STOP WORD REMOVAL                                 │    │
│  │    Remove: "the", "over"                             │    │
│  │    → ["quick", "brown", "fox", "jumps", "3",        │    │
│  │       "lazy", "dogs"]                                │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 4. STEMMING/LEMMATIZATION                            │    │
│  │    jumps → jump, dogs → dog                          │    │
│  │    → ["quick", "brown", "fox", "jump", "3",         │    │
│  │       "lazy", "dog"]                                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 5. INDEX TERMS with positions                        │    │
│  │    quick:0, brown:1, fox:2, jump:3, 3:4,            │    │
│  │    lazy:5, dog:6                                     │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### SPIMI (Single-Pass In-Memory Indexing)

```
┌─────────────────────────────────────────────────────────────┐
│            SPIMI Algorithm (Scalable Indexing)               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: Process documents in memory until full             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ In-Memory Dictionary with Posting Lists             │    │
│  │ "cat"    → [1, 5, 7, 12]                            │    │
│  │ "dog"    → [2, 5, 8]                                │    │
│  │ "animal" → [1, 2, 3, 5, 7, 8, 12]                   │    │
│  │ ...                                                 │    │
│  │ MEMORY FULL                                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  Step 2: Sort and write to disk block                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Block 1: [animal → [...], cat → [...], dog → [...]] │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  Step 3: Repeat for remaining documents                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Block 2: [cat → [...], elephant → [...], ...]       │    │
│  │ Block 3: [bird → [...], fish → [...], ...]          │    │
│  │ ...                                                  │    │
│  └─────────────────────────────────────────────────────┘    │
│                         │                                    │
│                         ▼                                    │
│  Step 4: Merge all blocks (n-way merge)                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Final Index:                                         │    │
│  │ animal → [merged postings from all blocks]          │    │
│  │ bird   → [merged postings]                          │    │
│  │ cat    → [merged postings]                          │    │
│  │ ...                                                 │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Compression Techniques

### Gap Encoding

```
┌─────────────────────────────────────────────────────────────┐
│                      Gap Encoding                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Original posting list (sorted DocIDs):                      │
│  [3, 5, 20, 21, 23, 76, 77, 78, 512]                        │
│                                                              │
│  Gap encoded (differences):                                  │
│  [3, 2, 15, 1, 2, 53, 1, 1, 434]                            │
│                                                              │
│  Benefits:                                                   │
│  • Smaller numbers (especially for common terms)            │
│  • Better compression ratios                                 │
│  • First value is absolute, rest are gaps                   │
│                                                              │
│  Decoding: Running sum                                       │
│  3 → 3+2=5 → 5+15=20 → 20+1=21 → ...                       │
└─────────────────────────────────────────────────────────────┘
```

### Variable-Byte Encoding

```
┌─────────────────────────────────────────────────────────────┐
│                  Variable-Byte (VByte) Encoding              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Each byte uses:                                             │
│  • 7 bits for data                                           │
│  • 1 bit as continuation flag (0 = more bytes, 1 = last)    │
│                                                              │
│  Examples:                                                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Number    Binary           VByte Encoding              │ │
│  │───────────────────────────────────────────────────────│ │
│  │ 5         101              10000101 (1 byte)           │ │
│  │ 127       1111111          11111111 (1 byte)           │ │
│  │ 128       10000000         00000001 10000000 (2 bytes) │ │
│  │ 16383     11111111111111   01111111 11111111 (2 bytes) │ │
│  │ 16384     100000000000000  00000001 00000000 10000000  │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Compression ratio:                                          │
│  • Numbers < 128: 1 byte (50% savings over 2-byte int)      │
│  • Numbers < 16384: 2 bytes (same as short)                 │
│  • Average for typical gaps: ~1.5 bytes                     │
└─────────────────────────────────────────────────────────────┘
```

### Elias Gamma/Delta Coding

```
┌─────────────────────────────────────────────────────────────┐
│                  Elias Gamma Encoding                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Format: N zeros followed by N+1 bit binary number          │
│                                                              │
│  Examples:                                                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ Number │ Binary │ N (floor log2) │ Gamma Code          │ │
│  │────────│────────│────────────────│─────────────────────│ │
│  │   1    │   1    │      0         │ 1                   │ │
│  │   2    │  10    │      1         │ 0 10                │ │
│  │   3    │  11    │      1         │ 0 11                │ │
│  │   4    │ 100    │      2         │ 00 100              │ │
│  │   5    │ 101    │      2         │ 00 101              │ │
│  │   9    │ 1001   │      3         │ 000 1001            │ │
│  │  13    │ 1101   │      3         │ 000 1101            │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                              │
│  Best for: Small numbers (gaps), heavily skewed data        │
│  Bits needed: 2*floor(log2(n)) + 1                          │
└─────────────────────────────────────────────────────────────┘
```

## Query Processing

### Boolean Queries

```
Query: "database" AND "optimization"

┌─────────────────────────────────────────────────────────────┐
│               Posting List Intersection                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  "database"     → [2, 4, 8, 16, 32, 64, 128, 256]          │
│  "optimization" → [3, 8, 15, 32, 33, 256, 257]              │
│                                                              │
│  Two-pointer merge:                                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ p1=2, p2=3   → 2<3, advance p1                       │   │
│  │ p1=4, p2=3   → 4>3, advance p2                       │   │
│  │ p1=4, p2=8   → 4<8, advance p1                       │   │
│  │ p1=8, p2=8   → MATCH! Output 8, advance both        │   │
│  │ p1=16, p2=15 → 16>15, advance p2                     │   │
│  │ p1=16, p2=32 → 16<32, advance p1                     │   │
│  │ p1=32, p2=32 → MATCH! Output 32, advance both       │   │
│  │ ...                                                  │   │
│  │ p1=256, p2=256 → MATCH! Output 256, advance both    │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  Result: [8, 32, 256]                                        │
│  Time: O(n + m) where n, m are posting list lengths         │
└─────────────────────────────────────────────────────────────┘
```

### Skip Pointers

```
┌─────────────────────────────────────────────────────────────┐
│                      Skip Pointers                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Long posting list with skip pointers (every sqrt(n)):      │
│                                                              │
│  Skip List: [0 → 16 → 64 → 128 → 256 → ...]                │
│              ↓    ↓     ↓     ↓      ↓                      │
│  Postings:  [2, 4, 8, 10, 12, 16, 20, 25, 32, 40, 55,      │
│              64, 70, 80, 95, 100, 110, 128, ...]            │
│                                                              │
│  Intersection with short list [60, 100, 300]:               │
│                                                              │
│  Looking for 60:                                             │
│  • Skip to 16? 16 < 60, skip                                │
│  • Skip to 64? 64 > 60, stop skipping                       │
│  • Linear scan from 16 to 64                                 │
│  • 60 not found between 55 and 64                           │
│                                                              │
│  Looking for 100:                                            │
│  • Current position at 64                                    │
│  • Skip to 128? 128 > 100, stop                             │
│  • Linear scan from 64: 70, 80, 95, 100 ✓                   │
│                                                              │
│  Skip distance: sqrt(L) for optimal performance             │
└─────────────────────────────────────────────────────────────┘
```

### Phrase Queries

```
Query: "machine learning"

┌─────────────────────────────────────────────────────────────┐
│                    Phrase Query Processing                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Positional postings:                                        │
│  "machine"  → [(Doc1, [5,42,100]), (Doc2, [3,67]), ...]    │
│  "learning" → [(Doc1, [6,88,101]), (Doc2, [4,15]), ...]    │
│                                                              │
│  Step 1: Find docs with both terms                          │
│  Intersect: [Doc1, Doc2, ...]                               │
│                                                              │
│  Step 2: Check positional adjacency in each doc             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Doc1:                                                │   │
│  │   "machine" positions: [5, 42, 100]                  │   │
│  │   "learning" positions: [6, 88, 101]                 │   │
│  │                                                      │   │
│  │   Check: learning_pos = machine_pos + 1?             │   │
│  │   5+1=6 ✓ Found at position 5-6                     │   │
│  │   42+1=43 ✗                                          │   │
│  │   100+1=101 ✓ Found at position 100-101             │   │
│  │                                                      │   │
│  │ Doc2:                                                │   │
│  │   3+1=4 ✓ Found at position 3-4                     │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  Result: [(Doc1, [5, 100]), (Doc2, [3])]                   │
└─────────────────────────────────────────────────────────────┘
```

## Scoring and Ranking

### TF-IDF

```
┌─────────────────────────────────────────────────────────────┐
│                     TF-IDF Scoring                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  TF (Term Frequency):                                        │
│  How often term appears in document                          │
│                                                              │
│  tf(t,d) = count(t in d)                                    │
│  or                                                          │
│  tf(t,d) = 1 + log(count(t in d))  [log normalization]     │
│                                                              │
│  IDF (Inverse Document Frequency):                          │
│  How rare is the term across all documents                   │
│                                                              │
│  idf(t) = log(N / df(t))                                    │
│  where N = total docs, df(t) = docs containing t            │
│                                                              │
│  TF-IDF Score:                                               │
│  score(t,d) = tf(t,d) × idf(t)                              │
│                                                              │
│  Example:                                                    │
│  ┌────────────────────────────────────────────────────────┐ │
│  │ N = 10,000 documents                                   │ │
│  │ Query: "rare algorithm"                                │ │
│  │                                                        │ │
│  │ "rare": df=100, appears 3x in Doc1                    │ │
│  │   tf = 1 + log(3) = 1.48                              │ │
│  │   idf = log(10000/100) = 2                            │ │
│  │   score = 1.48 × 2 = 2.96                             │ │
│  │                                                        │ │
│  │ "algorithm": df=5000, appears 10x in Doc1             │ │
│  │   tf = 1 + log(10) = 2                                │ │
│  │   idf = log(10000/5000) = 0.30                        │ │
│  │   score = 2 × 0.30 = 0.60                             │ │
│  │                                                        │ │
│  │ Total for Doc1 = 2.96 + 0.60 = 3.56                   │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### BM25

```
┌─────────────────────────────────────────────────────────────┐
│                        BM25 Scoring                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  BM25 (Best Match 25) - Industry standard ranking:          │
│                                                              │
│               IDF(t) × f(t,d) × (k1 + 1)                    │
│  score(t,d) = ─────────────────────────────────────         │
│               f(t,d) + k1 × (1 - b + b × |d|/avgdl)         │
│                                                              │
│  Where:                                                      │
│  • f(t,d) = term frequency in document                      │
│  • |d| = document length                                     │
│  • avgdl = average document length                          │
│  • k1 = term frequency saturation (typically 1.2-2.0)       │
│  • b = length normalization (typically 0.75)                │
│                                                              │
│  IDF for BM25:                                               │
│  IDF(t) = log((N - df(t) + 0.5) / (df(t) + 0.5))           │
│                                                              │
│  Key Differences from TF-IDF:                               │
│  • TF saturation: diminishing returns for high TF           │
│  • Length normalization: penalizes long documents           │
│  • Better empirical performance                              │
└─────────────────────────────────────────────────────────────┘
```

## Database Implementations

### PostgreSQL Full-Text Search

```sql
-- Create tsvector column (term positions)
ALTER TABLE articles ADD COLUMN tsv tsvector;
UPDATE articles SET tsv = to_tsvector('english', title || ' ' || body);

-- Create GIN index on tsvector
CREATE INDEX idx_articles_tsv ON articles USING GIN(tsv);

-- Query with tsquery
SELECT title, ts_rank(tsv, query) as rank
FROM articles, to_tsquery('english', 'database & optimization') query
WHERE tsv @@ query
ORDER BY rank DESC
LIMIT 10;

-- Phrase search
SELECT * FROM articles
WHERE tsv @@ phraseto_tsquery('english', 'machine learning');

-- View index contents
SELECT * FROM ts_stat('SELECT tsv FROM articles')
ORDER BY nentry DESC
LIMIT 20;
```

### Elasticsearch/Lucene

```json
// Index mapping
PUT /articles
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "english",
        "index_options": "positions"
      },
      "body": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}

// BM25 query
GET /articles/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "database" }},
        { "match": { "body": "optimization" }}
      ]
    }
  }
}

// Phrase query
GET /articles/_search
{
  "query": {
    "match_phrase": {
      "body": {
        "query": "machine learning",
        "slop": 1
      }
    }
  }
}
```

### MongoDB Text Index

```javascript
// Create text index
db.articles.createIndex({
  title: "text",
  body: "text"
}, {
  weights: { title: 10, body: 1 },
  default_language: "english"
});

// Text search
db.articles.find({
  $text: {
    $search: "database optimization",
    $language: "english"
  }
}, {
  score: { $meta: "textScore" }
}).sort({ score: { $meta: "textScore" }});

// Phrase search
db.articles.find({
  $text: { $search: "\"machine learning\"" }
});
```

## Performance Optimization

```
┌─────────────────────────────────────────────────────────────┐
│              Inverted Index Optimization                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Query Optimization:                                      │
│     • Process terms by ascending DF (rarest first)          │
│     • Short-circuit AND queries early                       │
│     • Use skip pointers for long posting lists              │
│                                                              │
│  2. Index Compression:                                       │
│     • Gap encoding + VByte (typical)                        │
│     • Block-based compression (PFOR-Delta)                  │
│     • Dictionary compression for terms                       │
│                                                              │
│  3. Caching:                                                 │
│     • Cache frequently accessed posting lists               │
│     • Cache query results                                    │
│     • Cache term dictionary in memory                        │
│                                                              │
│  4. Sharding:                                                │
│     • Document partitioning (each shard has all terms)      │
│     • Term partitioning (each shard has subset of terms)    │
│     • Hybrid approaches                                      │
│                                                              │
│  5. Real-time Updates:                                       │
│     • In-memory buffer for new docs                         │
│     • Background merging with disk segments                 │
│     • Near real-time search with refresh                    │
└─────────────────────────────────────────────────────────────┘
```

## Key Takeaways

1. **Inverted index maps terms to documents** - Fundamental for text search
2. **Positional information enables phrase queries** - Store term positions
3. **Compression is essential** - Gap encoding + variable-byte encoding
4. **Query processing uses posting list operations** - Intersection, union
5. **BM25 is the modern standard** - Better than TF-IDF in practice
6. **Real databases use sophisticated implementations** - Lucene, PostgreSQL GIN
