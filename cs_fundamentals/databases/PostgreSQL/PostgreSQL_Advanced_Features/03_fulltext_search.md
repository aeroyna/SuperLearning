# Full-Text Search

## Learning Objectives
- Configure PostgreSQL full-text search
- Create and use text search vectors
- Implement ranking and highlighting
- Optimize FTS with proper indexing

---

## 1. Full-Text Search Fundamentals

### Why Full-Text Search?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    LIKE vs Full-Text Search                          │
│                                                                      │
│  LIKE/ILIKE:                                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Pattern matching on raw text                              │    │
│  │ • No linguistic awareness                                   │    │
│  │ • Cannot rank results                                       │    │
│  │ • '%word%' cannot use B-tree index                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Full-Text Search:                                                   │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ • Understands language (stemming, stop words)               │    │
│  │ • Ranks results by relevance                                │    │
│  │ • Supports operators (AND, OR, NOT, phrase)                 │    │
│  │ • Uses GIN/GiST indexes efficiently                         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  Example: Searching for "running"                                    │
│  LIKE: Only matches "running"                                        │
│  FTS:  Matches "run", "runs", "running", "ran"                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Core Concepts

```sql
-- Document: Text to search (tsvector)
-- Query: What to search for (tsquery)

-- Create tsvector from text
SELECT to_tsvector('english', 'The quick brown fox jumps over the lazy dog');
-- 'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2

-- Note: Removes stop words (the, over), stems words (jumps→jump, lazy→lazi)

-- Create tsquery from search terms
SELECT to_tsquery('english', 'quick & fox');
-- 'quick' & 'fox'

-- Match tsvector against tsquery
SELECT to_tsvector('english', 'The quick brown fox') @@ to_tsquery('english', 'fox');
-- true
```

---

## 2. Text Search Vectors (tsvector)

### Creating Vectors

```sql
-- Basic tsvector
SELECT 'quick brown fox'::tsvector;
-- 'brown' 'fox' 'quick' (unprocessed, no positions)

-- Processed with language config
SELECT to_tsvector('english', 'Quick brown foxes jumping');
-- 'brown':2 'fox':3 'jump':4 'quick':1

-- Different languages
SELECT to_tsvector('spanish', 'Los rápidos zorros marrones');
SELECT to_tsvector('german', 'Der schnelle braune Fuchs');
SELECT to_tsvector('simple', 'The Quick BROWN fox');  -- No stemming
-- 'brown':3 'fox':4 'quick':2 'the':1
```

### Combining Vectors

```sql
-- Concatenate vectors
SELECT to_tsvector('english', 'title text') || to_tsvector('english', 'body text');

-- With weight (A, B, C, D - A highest)
SELECT setweight(to_tsvector('english', 'Important Title'), 'A') ||
       setweight(to_tsvector('english', 'Less important body'), 'B');
-- 'bodi':5B 'import':1A,4B 'less':3B 'titl':2A
```

### Storing Vectors

```sql
-- Articles table with stored tsvector
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    body TEXT NOT NULL,
    author VARCHAR(100),
    published_at TIMESTAMP,
    -- Pre-computed search vector
    search_vector TSVECTOR GENERATED ALWAYS AS (
        setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(body, '')), 'B')
    ) STORED
);

-- Create GIN index on search vector
CREATE INDEX idx_articles_search ON articles USING GIN (search_vector);
```

---

## 3. Text Search Queries (tsquery)

### Creating Queries

```sql
-- Basic query
SELECT to_tsquery('english', 'cat');
-- 'cat'

-- Multiple terms (AND)
SELECT to_tsquery('english', 'cat & dog');
-- 'cat' & 'dog'

-- OR operator
SELECT to_tsquery('english', 'cat | dog');
-- 'cat' | 'dog'

-- NOT operator
SELECT to_tsquery('english', 'cat & !dog');
-- 'cat' & !'dog'

-- Phrase search (adjacent words)
SELECT to_tsquery('english', 'quick <-> brown');
-- 'quick' <-> 'brown'

-- Phrase with distance
SELECT to_tsquery('english', 'quick <2> fox');  -- Within 2 words
-- 'quick' <2> 'fox'

-- Prefix matching
SELECT to_tsquery('english', 'post:*');
-- 'post':*  (matches post, posts, posting, posted, etc.)
```

### User-Friendly Query Parsing

```sql
-- plainto_tsquery: Simple space-separated AND
SELECT plainto_tsquery('english', 'quick brown fox');
-- 'quick' & 'brown' & 'fox'

-- phraseto_tsquery: Phrase search
SELECT phraseto_tsquery('english', 'quick brown fox');
-- 'quick' <-> 'brown' <-> 'fox'

-- websearch_to_tsquery: Google-like syntax (PostgreSQL 11+)
SELECT websearch_to_tsquery('english', 'quick brown -lazy');
-- 'quick' & 'brown' & !'lazi'

SELECT websearch_to_tsquery('english', '"quick brown" fox');
-- 'quick' <-> 'brown' & 'fox'

SELECT websearch_to_tsquery('english', 'cat or dog');
-- 'cat' | 'dog'
```

---

## 4. Performing Searches

### Basic Search

```sql
-- Search articles
SELECT title, body
FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql & performance');

-- Using websearch for user input
SELECT title, body
FROM articles
WHERE search_vector @@ websearch_to_tsquery('english', 'postgresql performance');

-- On-the-fly search (no stored vector)
SELECT title, body
FROM articles
WHERE to_tsvector('english', title || ' ' || body) @@ to_tsquery('english', 'search & term');
```

### Search with Ranking

```sql
-- ts_rank: Frequency-based ranking
SELECT
    title,
    ts_rank(search_vector, query) AS rank
FROM articles, to_tsquery('english', 'postgresql') query
WHERE search_vector @@ query
ORDER BY rank DESC
LIMIT 10;

-- ts_rank_cd: Cover density ranking (considers proximity)
SELECT
    title,
    ts_rank_cd(search_vector, query) AS rank
FROM articles, websearch_to_tsquery('english', 'database optimization') query
WHERE search_vector @@ query
ORDER BY rank DESC;

-- Rank normalization options
-- 0: Default
-- 1: Divides by (1 + log(document length))
-- 2: Divides by document length
-- 4: Divides by mean harmonic distance between extents
-- 8: Divides by number of unique words
-- 16: Divides by (1 + log(unique words))
-- 32: Divides by (1 + log(rank))

SELECT
    title,
    ts_rank(search_vector, query, 32) AS rank  -- Normalized
FROM articles, to_tsquery('english', 'postgresql') query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

### Highlighting Results

```sql
-- ts_headline: Highlight matching terms
SELECT
    title,
    ts_headline('english', body, query,
        'StartSel=<b>, StopSel=</b>, MaxWords=50, MinWords=25, MaxFragments=3'
    ) AS snippet
FROM articles, websearch_to_tsquery('english', 'database performance') query
WHERE search_vector @@ query;

-- Custom highlighting options
SELECT ts_headline('english',
    'PostgreSQL is a powerful database system with advanced features.',
    to_tsquery('english', 'database'),
    'StartSel=[[, StopSel=]], HighlightAll=true'
);
-- PostgreSQL is a powerful [[database]] system with advanced features.
```

---

## 5. Text Search Configuration

### Available Configurations

```sql
-- List all configurations
SELECT cfgname FROM pg_ts_config;

-- Common configurations:
-- simple, danish, dutch, english, finnish, french, german,
-- hungarian, italian, norwegian, portuguese, romanian, russian,
-- spanish, swedish, turkish

-- Show current default
SHOW default_text_search_config;

-- Set default
SET default_text_search_config = 'english';
```

### Configuration Components

```
┌─────────────────────────────────────────────────────────────────────┐
│                Text Search Configuration Pipeline                    │
│                                                                      │
│  Input Text: "The PostgreSQL database stores running queries"       │
│       │                                                              │
│       ▼                                                              │
│  ┌─────────────┐                                                     │
│  │   PARSER    │  Breaks text into tokens                           │
│  │             │  (words, numbers, emails, URLs, etc.)              │
│  └─────────────┘                                                     │
│       │                                                              │
│       ▼                                                              │
│  Tokens: [The] [PostgreSQL] [database] [stores] [running] [queries] │
│       │                                                              │
│       ▼                                                              │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                      DICTIONARIES                             │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐              │   │
│  │  │ Stop Words │  │  Synonym   │  │  Stemmer   │              │   │
│  │  │ (the, a)   │  │ Dictionary │  │ (Snowball) │              │   │
│  │  └────────────┘  └────────────┘  └────────────┘              │   │
│  └──────────────────────────────────────────────────────────────┘   │
│       │                                                              │
│       ▼                                                              │
│  Lexemes: 'postgresql' 'databas' 'store' 'run' 'queri'              │
└─────────────────────────────────────────────────────────────────────┘
```

### Custom Configuration

```sql
-- Create custom configuration based on English
CREATE TEXT SEARCH CONFIGURATION my_english (COPY = english);

-- Add synonym dictionary
CREATE TEXT SEARCH DICTIONARY my_synonyms (
    TEMPLATE = synonym,
    SYNONYMS = my_synonyms  -- Points to my_synonyms.syn file
);

-- Alter configuration to use synonym dictionary
ALTER TEXT SEARCH CONFIGURATION my_english
    ALTER MAPPING FOR asciiword
    WITH my_synonyms, english_stem;

-- Test configuration
SELECT * FROM ts_debug('my_english', 'quick fast database db');
```

### Dictionaries

```sql
-- List dictionaries
SELECT dictname FROM pg_ts_dict;

-- Test a dictionary
SELECT ts_lexize('english_stem', 'running');  -- {run}
SELECT ts_lexize('english_stem', 'postgresql');  -- {postgresql}

-- Simple dictionary (no stemming, just lowercase)
SELECT to_tsvector('simple', 'RUNNING Foxes');
-- 'foxes':2 'running':1

-- Create ispell dictionary
CREATE TEXT SEARCH DICTIONARY english_ispell (
    TEMPLATE = ispell,
    DictFile = english,
    AffFile = english,
    StopWords = english
);
```

---

## 6. Indexing for Full-Text Search

### GIN Index

```sql
-- GIN (Generalized Inverted Index) - Most common
CREATE INDEX idx_articles_fts ON articles USING GIN (search_vector);

-- GIN is best for:
-- - Static data or moderate updates
-- - Many unique words
-- - Fast lookup, slower updates

-- Query uses index
EXPLAIN SELECT * FROM articles
WHERE search_vector @@ to_tsquery('postgresql');
-- Index Scan using idx_articles_fts
```

### GiST Index

```sql
-- GiST (Generalized Search Tree) - Alternative
CREATE INDEX idx_articles_fts_gist ON articles USING GiST (search_vector);

-- GiST is best for:
-- - Frequent updates
-- - Smaller documents
-- - When combining with other GiST-compatible types (geometry, ranges)

-- GiST can be lossy (may need recheck)
-- Generally slower than GIN for pure FTS
```

### Expression Index

```sql
-- Index on-the-fly computation
CREATE INDEX idx_articles_content ON articles
USING GIN (to_tsvector('english', title || ' ' || body));

-- Must use exact same expression in query
SELECT * FROM articles
WHERE to_tsvector('english', title || ' ' || body) @@ to_tsquery('search');
```

### Index Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GIN vs GiST for FTS                               │
│                                                                      │
│  Characteristic        GIN                  GiST                     │
│  ─────────────────────────────────────────────────────────────────  │
│  Index Size            Larger               Smaller                  │
│  Build Time            Slower               Faster                   │
│  Update Speed          Slower               Faster                   │
│  Search Speed          Faster               Slower                   │
│  Exact Matches         Yes                  Lossy (recheck)          │
│                                                                      │
│  Recommendation:                                                     │
│  - Read-heavy: GIN                                                   │
│  - Write-heavy: GiST                                                 │
│  - Most cases: GIN with fastupdate                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 7. Multi-Language Search

### Language Detection

```sql
-- Store language per document
CREATE TABLE multilingual_content (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    language VARCHAR(20) DEFAULT 'english',
    search_vector TSVECTOR
);

-- Trigger to update search vector with correct language
CREATE OR REPLACE FUNCTION update_search_vector()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector := to_tsvector(NEW.language::regconfig, NEW.content);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER content_search_update
    BEFORE INSERT OR UPDATE ON multilingual_content
    FOR EACH ROW EXECUTE FUNCTION update_search_vector();
```

### Multi-Language Index

```sql
-- Multiple language vectors in one field
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content_en TEXT,
    content_es TEXT,
    content_fr TEXT,
    search_vector TSVECTOR GENERATED ALWAYS AS (
        setweight(to_tsvector('english', coalesce(content_en, '')), 'A') ||
        setweight(to_tsvector('spanish', coalesce(content_es, '')), 'B') ||
        setweight(to_tsvector('french', coalesce(content_fr, '')), 'C')
    ) STORED
);

CREATE INDEX idx_docs_search ON documents USING GIN (search_vector);
```

---

## 8. Advanced Techniques

### Fuzzy Search with Trigrams

```sql
-- Enable pg_trgm extension
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Trigram similarity (handles typos)
SELECT similarity('postgresql', 'postgrsql');  -- 0.7

-- Combine with FTS
SELECT
    title,
    similarity(title, 'postgrsql') AS sim,
    ts_rank(search_vector, query) AS rank
FROM articles, to_tsquery('english', 'postgresql') query
WHERE search_vector @@ query
   OR similarity(title, 'postgrsql') > 0.3
ORDER BY rank DESC, sim DESC;

-- Trigram index for LIKE
CREATE INDEX idx_title_trgm ON articles USING GIN (title gin_trgm_ops);

SELECT * FROM articles WHERE title ILIKE '%postgres%';  -- Uses index
```

### Phrase Search Optimization

```sql
-- Phrase search with position checking
SELECT * FROM articles
WHERE search_vector @@ phraseto_tsquery('english', 'database performance');

-- Ensure positions are stored in tsvector
-- (They are by default with to_tsvector)
```

### Autocomplete / Suggestions

```sql
-- Prefix search for autocomplete
SELECT DISTINCT word
FROM ts_stat('SELECT search_vector FROM articles')
WHERE word LIKE 'post%'
ORDER BY ndoc DESC, word
LIMIT 10;

-- With frequency
SELECT word, ndoc, nentry
FROM ts_stat('SELECT search_vector FROM articles')
WHERE word LIKE 'data%'
ORDER BY nentry DESC
LIMIT 10;
```

---

## 9. Performance Optimization

### Query Optimization

```sql
-- Use stored tsvector (not on-the-fly)
-- Good:
SELECT * FROM articles WHERE search_vector @@ to_tsquery('postgresql');

-- Slower:
SELECT * FROM articles
WHERE to_tsvector('english', body) @@ to_tsquery('postgresql');

-- Limit expensive headline generation
SELECT
    id,
    title,
    CASE
        WHEN rank > 0.1 THEN ts_headline('english', body, query, 'MaxWords=30')
        ELSE left(body, 100) || '...'
    END AS snippet
FROM (
    SELECT id, title, body, search_vector,
           ts_rank(search_vector, query) AS rank,
           query
    FROM articles, to_tsquery('postgresql') query
    WHERE search_vector @@ query
    ORDER BY rank DESC
    LIMIT 100
) sub;
```

### Index Maintenance

```sql
-- GIN pending list (for fast updates)
SHOW gin_pending_list_limit;  -- Default 4MB

-- Force GIN cleanup
VACUUM articles;

-- Analyze for statistics
ANALYZE articles;

-- Check index size
SELECT pg_size_pretty(pg_relation_size('idx_articles_search'));
```

---

## 10. Complete Example

```sql
-- Blog with full-text search
CREATE TABLE blog_posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    slug VARCHAR(200) UNIQUE NOT NULL,
    content TEXT NOT NULL,
    tags TEXT[],
    author_id INTEGER REFERENCES users(id),
    published BOOLEAN DEFAULT FALSE,
    published_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),

    -- Generated search vector
    search_vector TSVECTOR GENERATED ALWAYS AS (
        setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(array_to_string(tags, ' '), '')), 'B') ||
        setweight(to_tsvector('english', coalesce(content, '')), 'C')
    ) STORED
);

-- Indexes
CREATE INDEX idx_blog_search ON blog_posts USING GIN (search_vector);
CREATE INDEX idx_blog_published ON blog_posts (published, published_at DESC);

-- Search function
CREATE OR REPLACE FUNCTION search_posts(
    search_query TEXT,
    p_limit INTEGER DEFAULT 20,
    p_offset INTEGER DEFAULT 0
)
RETURNS TABLE (
    id INTEGER,
    title VARCHAR,
    slug VARCHAR,
    snippet TEXT,
    rank REAL,
    published_at TIMESTAMP
) AS $$
DECLARE
    query tsquery;
BEGIN
    query := websearch_to_tsquery('english', search_query);

    RETURN QUERY
    SELECT
        p.id,
        p.title,
        p.slug,
        ts_headline('english', p.content, query,
            'MaxWords=50, MinWords=25, StartSel=<mark>, StopSel=</mark>'
        ) AS snippet,
        ts_rank(p.search_vector, query) AS rank,
        p.published_at
    FROM blog_posts p
    WHERE p.published = TRUE
      AND p.search_vector @@ query
    ORDER BY rank DESC, p.published_at DESC
    LIMIT p_limit
    OFFSET p_offset;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT * FROM search_posts('postgresql optimization tips');
```

---

## Summary

| Function | Purpose |
|----------|---------|
| `to_tsvector()` | Create searchable document |
| `to_tsquery()` | Create search query |
| `@@` | Match operator |
| `ts_rank()` | Rank results |
| `ts_headline()` | Highlight matches |
| `setweight()` | Assign importance (A-D) |

---

## Further Reading

- PostgreSQL Full Text Search documentation
- Text Search Dictionaries
- ts_stat() for corpus analysis
