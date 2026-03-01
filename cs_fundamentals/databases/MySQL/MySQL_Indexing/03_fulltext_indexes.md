# Full-Text Indexes in MySQL

## Learning Objectives
- Understand full-text index structure
- Master MATCH...AGAINST syntax
- Learn natural language and boolean search modes
- Optimize full-text search performance

---

## 1. Full-Text Index Overview

Full-text indexes enable searching for words within text columns, unlike regular indexes that match exact values or prefixes.

```
┌─────────────────────────────────────────────────────────────────┐
│               Full-Text Index vs Regular Index                   │
│                                                                  │
│  Regular Index:                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ WHERE title = 'MySQL Database Guide'     ← Exact match      ││
│  │ WHERE title LIKE 'MySQL%'                ← Prefix match     ││
│  │ WHERE title LIKE '%Database%'            ← Full scan!       ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Full-Text Index:                                                │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ WHERE MATCH(title) AGAINST('Database')   ← Word search      ││
│  │ WHERE MATCH(title) AGAINST('MySQL Guide')← Multiple words   ││
│  │ WHERE MATCH(title,body) AGAINST('query') ← Multiple columns ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Creating Full-Text Indexes

### Basic Syntax

```sql
-- Create table with full-text index
CREATE TABLE articles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255),
    body TEXT,
    author VARCHAR(100),
    created_at DATETIME,
    FULLTEXT INDEX ft_title (title),
    FULLTEXT INDEX ft_body (body),
    FULLTEXT INDEX ft_title_body (title, body)
) ENGINE = InnoDB;

-- Add full-text index to existing table
ALTER TABLE articles ADD FULLTEXT INDEX ft_content (title, body);

-- Or using CREATE INDEX
CREATE FULLTEXT INDEX ft_author ON articles (author);

-- Drop full-text index
ALTER TABLE articles DROP INDEX ft_author;
```

### Storage Engine Support

```sql
-- InnoDB: Full support since MySQL 5.6
-- MyISAM: Original full-text support

-- InnoDB is recommended:
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    description TEXT,
    FULLTEXT INDEX ft_search (name, description)
) ENGINE = InnoDB;  -- Full ACID compliance + full-text
```

---

## 3. Full-Text Index Structure

### Inverted Index

```
┌─────────────────────────────────────────────────────────────────┐
│                    Inverted Index Structure                      │
│                                                                  │
│  Documents:                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Doc 1: "MySQL is a database management system"             │ │
│  │ Doc 2: "PostgreSQL is also a database system"              │ │
│  │ Doc 3: "MongoDB is a NoSQL database"                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Inverted Index:                                                 │
│  ┌─────────────┬─────────────────────────────────┐              │
│  │ Word        │ Document List (doc_id:position) │              │
│  ├─────────────┼─────────────────────────────────┤              │
│  │ database    │ 1:4, 2:5, 3:5                   │              │
│  │ management  │ 1:5                              │              │
│  │ mongodb     │ 3:1                              │              │
│  │ mysql       │ 1:1                              │              │
│  │ nosql       │ 3:4                              │              │
│  │ postgresql  │ 2:1                              │              │
│  │ system      │ 1:6, 2:6                         │              │
│  └─────────────┴─────────────────────────────────┘              │
│                                                                  │
│  Search "database system":                                       │
│  → database: docs 1,2,3                                          │
│  → system: docs 1,2                                              │
│  → Intersection with proximity: docs 1,2 (best matches)          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Search Modes

### Natural Language Mode (Default)

```sql
-- Searches for words, ranks by relevance
SELECT id, title,
       MATCH(title, body) AGAINST('database optimization') AS relevance
FROM articles
WHERE MATCH(title, body) AGAINST('database optimization')
ORDER BY relevance DESC;

-- Natural language mode features:
-- - Ignores stopwords (a, the, is, etc.)
-- - Ignores words shorter than ft_min_word_len (default 4)
-- - Ranks results by relevance score
-- - Word must appear in less than 50% of rows
```

### Boolean Mode

```sql
-- Full control with operators
SELECT * FROM articles
WHERE MATCH(title, body) AGAINST('+MySQL -PostgreSQL' IN BOOLEAN MODE);

-- Boolean operators:
-- +word    : Must contain
-- -word    : Must NOT contain
-- word*    : Prefix wildcard
-- "phrase" : Exact phrase
-- >word    : Increase relevance
-- <word    : Decrease relevance
-- ~word    : Negative contribution to relevance
-- (...)    : Grouping

-- Examples:
-- '+database +optimization'        : Both words required
-- '"MySQL database"'               : Exact phrase
-- '+database optimization*'        : database required, optimization prefix
-- '+MySQL +(>InnoDB <MyISAM)'     : MySQL required, prefer InnoDB over MyISAM
-- '+database -"NoSQL database"'   : database but not the phrase "NoSQL database"
```

### Query Expansion Mode

```sql
-- Expands search with related words found in results
SELECT * FROM articles
WHERE MATCH(title, body) AGAINST('database' WITH QUERY EXPANSION);

-- How it works:
-- 1. Perform initial search for 'database'
-- 2. Find most relevant words in top results
-- 3. Re-run search with original + found words

-- Useful for: finding related content
-- Risky: can return unexpected results (noise)
```

---

## 5. Relevance Scoring

### Understanding Scores

```sql
-- Get relevance score
SELECT
    id,
    title,
    MATCH(title, body) AGAINST('MySQL database') AS score
FROM articles
WHERE MATCH(title, body) AGAINST('MySQL database')
ORDER BY score DESC;

-- Score factors:
-- - Term frequency (TF): How often word appears in document
-- - Inverse document frequency (IDF): Rarity across all documents
-- - Field length: Shorter fields get higher scores per match
```

### Boosting Relevance

```sql
-- Weight different columns
SELECT
    id,
    title,
    (MATCH(title) AGAINST('MySQL') * 2 +
     MATCH(body) AGAINST('MySQL')) AS weighted_score
FROM articles
WHERE MATCH(title, body) AGAINST('MySQL')
ORDER BY weighted_score DESC;

-- Title matches worth 2x body matches
```

---

## 6. Configuration

### InnoDB Full-Text Settings

```sql
-- View settings
SHOW VARIABLES LIKE 'innodb_ft%';

-- Key settings:
-- innodb_ft_min_token_size: Minimum word length (default 3)
-- innodb_ft_max_token_size: Maximum word length (default 84)
-- innodb_ft_enable_stopword: Enable/disable stopwords
-- innodb_ft_server_stopword_table: Custom stopword table

-- Change minimum word length
SET GLOBAL innodb_ft_min_token_size = 2;
-- Requires index rebuild after change!
```

### MyISAM Full-Text Settings

```sql
SHOW VARIABLES LIKE 'ft_%';

-- ft_min_word_len: Minimum word length (default 4)
-- ft_max_word_len: Maximum word length (default 84)
-- ft_stopword_file: Path to stopword file
```

### Custom Stopwords

```sql
-- Create stopword table
CREATE TABLE my_stopwords (
    value VARCHAR(18) NOT NULL DEFAULT ''
) ENGINE = InnoDB;

INSERT INTO my_stopwords VALUES ('the'), ('a'), ('an'), ('and'), ('or');

-- Configure to use custom stopwords
SET GLOBAL innodb_ft_server_stopword_table = 'mydb/my_stopwords';

-- Rebuild index to apply
ALTER TABLE articles DROP INDEX ft_title_body;
ALTER TABLE articles ADD FULLTEXT INDEX ft_title_body (title, body);
```

---

## 7. Performance Optimization

### Index Maintenance

```sql
-- Rebuild full-text index (after config changes)
ALTER TABLE articles DROP INDEX ft_title_body;
ALTER TABLE articles ADD FULLTEXT INDEX ft_title_body (title, body);

-- Or optimize table
OPTIMIZE TABLE articles;

-- Check index status
SELECT * FROM information_schema.INNODB_FT_CONFIG
WHERE TABLE_ID = (
    SELECT TABLE_ID FROM information_schema.INNODB_TABLES
    WHERE NAME = 'mydb/articles'
);
```

### Query Optimization

```sql
-- Use full-text index in WHERE, not just ORDER BY
-- Bad: Full scan then sort
SELECT * FROM articles
WHERE category_id = 5
ORDER BY MATCH(body) AGAINST('database');

-- Good: Full-text search with additional filter
SELECT * FROM articles
WHERE MATCH(body) AGAINST('database')
  AND category_id = 5;

-- Even better: Combined index strategy
CREATE INDEX idx_category ON articles (category_id);
-- Query will use full-text index, then filter by category
```

### Pagination with Full-Text

```sql
-- Efficient pagination using relevance
SELECT id, title,
       MATCH(title, body) AGAINST('MySQL') AS score
FROM articles
WHERE MATCH(title, body) AGAINST('MySQL')
  AND MATCH(title, body) AGAINST('MySQL') > 0.5  -- Score threshold
ORDER BY score DESC
LIMIT 20 OFFSET 40;

-- For deep pagination, consider search-after pattern
-- Store last score and id, query for score < last_score
```

---

## 8. Practical Examples

### Product Search

```sql
-- Product catalog search
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category_id INT,
    price DECIMAL(10,2),
    in_stock BOOLEAN DEFAULT TRUE,
    FULLTEXT INDEX ft_search (name, description),
    INDEX idx_category (category_id),
    INDEX idx_price (price)
);

-- Search with filters
SELECT
    id,
    name,
    price,
    MATCH(name, description) AGAINST('wireless headphones') AS relevance
FROM products
WHERE MATCH(name, description) AGAINST('wireless headphones')
  AND category_id = 5
  AND in_stock = TRUE
  AND price BETWEEN 50 AND 200
ORDER BY relevance DESC
LIMIT 20;

-- Boolean search for specific features
SELECT * FROM products
WHERE MATCH(name, description) AGAINST(
    '+wireless +bluetooth -wired "noise canceling"' IN BOOLEAN MODE
)
ORDER BY price ASC;
```

### Blog/CMS Search

```sql
-- Blog post search
CREATE TABLE posts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    content LONGTEXT,
    tags VARCHAR(500),
    author_id INT,
    published_at DATETIME,
    status ENUM('draft', 'published', 'archived'),
    FULLTEXT INDEX ft_post (title, content),
    FULLTEXT INDEX ft_tags (tags)
);

-- Search published posts
SELECT
    id,
    title,
    LEFT(content, 200) AS excerpt,
    published_at,
    MATCH(title, content) AGAINST('MySQL performance') AS score
FROM posts
WHERE MATCH(title, content) AGAINST('MySQL performance')
  AND status = 'published'
ORDER BY score DESC, published_at DESC
LIMIT 10;

-- Search by tags
SELECT * FROM posts
WHERE MATCH(tags) AGAINST('+mysql +tutorial' IN BOOLEAN MODE)
  AND status = 'published';
```

### Search Suggestions

```sql
-- Build search suggestions from common terms
CREATE TABLE search_terms (
    term VARCHAR(100) PRIMARY KEY,
    search_count INT DEFAULT 0,
    FULLTEXT INDEX ft_term (term)
);

-- Get suggestions as user types
SELECT term, search_count
FROM search_terms
WHERE MATCH(term) AGAINST('dat*' IN BOOLEAN MODE)
ORDER BY search_count DESC
LIMIT 10;

-- Alternatively, prefix search with LIKE for suggestions
SELECT DISTINCT LEFT(title, 50) as suggestion
FROM articles
WHERE title LIKE 'dat%'
LIMIT 10;
```

---

## 9. Limitations and Alternatives

### Full-Text Limitations

```sql
-- 1. Minimum word length (default 3-4 characters)
--    Solution: Lower innodb_ft_min_token_size (rebuild index)

-- 2. No partial word matching (except with *)
--    Solution: Use LIKE for single partial match, or external search

-- 3. Limited relevance tuning
--    Solution: Combine with application-level scoring

-- 4. Index rebuild required for configuration changes

-- 5. Boolean mode ignores relevance ranking
```

### When to Use External Search

```sql
-- Consider Elasticsearch, Solr, or Meilisearch when:
-- - Need advanced relevance tuning
-- - Need faceted search
-- - Need fuzzy matching
-- - Need synonym support
-- - Need real-time indexing at scale
-- - Need highlighting
-- - Need aggregations

-- MySQL full-text is good for:
-- - Simple search requirements
-- - Small to medium datasets
-- - When you want to avoid external dependencies
```

---

## 10. Monitoring and Troubleshooting

### Check Index Usage

```sql
-- Explain full-text query
EXPLAIN SELECT * FROM articles
WHERE MATCH(title, body) AGAINST('database');

-- Should show: fulltext in Extra column

-- Check if index is being used
EXPLAIN FORMAT=JSON SELECT * FROM articles
WHERE MATCH(title, body) AGAINST('database');
```

### Common Issues

```sql
-- Issue: No results returned
-- Cause 1: Word too short
SHOW VARIABLES LIKE 'innodb_ft_min_token_size';

-- Cause 2: Word is a stopword
SELECT * FROM information_schema.INNODB_FT_DEFAULT_STOPWORD;

-- Cause 3: Word appears in > 50% of rows (natural language mode)
-- Solution: Use boolean mode

-- Issue: Unexpected results
-- Use boolean mode for more control
SELECT * FROM articles
WHERE MATCH(body) AGAINST('+expected -unexpected' IN BOOLEAN MODE);
```

---

## Summary

| Mode | Best For | Features |
|------|----------|----------|
| Natural Language | General search | Automatic relevance, stopwords |
| Boolean | Precise control | Operators (+,-,*,"") |
| Query Expansion | Discover related | Expands to related terms |

### Key Takeaways

1. **Full-text indexes** enable word-based searching within text
2. **Boolean mode** gives precise control with operators
3. **Natural language mode** handles relevance automatically
4. **Configure** min word length and stopwords for your needs
5. **Consider external search** for advanced requirements

---

## Further Reading

- MySQL Full-Text Search Functions documentation
- InnoDB Full-Text Index internals
- Elasticsearch vs MySQL Full-Text comparison
