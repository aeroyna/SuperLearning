# Google Case Studies

> Deep dives into designing Google-scale systems, demonstrating the thought process expected in interviews.

## Overview

These case studies show how to approach Google-style system design problems, emphasizing scale, trade-offs, and Google-specific technologies.

---

## Case Study 1: Design Google Search

### Problem Statement
Design a web search engine that can handle billions of queries per day with sub-second latency.

### Clarifying Questions
```
1. What's our scale target? (Assume 8.5B searches/day)
2. Geographic scope? (Global)
3. What types of content? (Web pages primarily)
4. Real-time indexing required? (Near real-time, <minutes)
5. Personalization depth? (Location, language, history)
```

### Requirements

**Functional:**
- Crawl and index web pages
- Process search queries
- Rank results by relevance
- Support advanced operators (site:, filetype:)
- Autocomplete suggestions

**Non-Functional:**
- Latency: < 200ms for 99th percentile
- Availability: 99.99%
- Scale: 100K QPS globally
- Freshness: Critical pages indexed within minutes

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Google Search Architecture                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────┐                                                           │
│   │   Browser   │                                                           │
│   └──────┬──────┘                                                           │
│          │                                                                   │
│          ▼                                                                   │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │                    Global Load Balancer                      │          │
│   │                    (Anycast + GSLB)                          │          │
│   └─────────────────────────────────┬───────────────────────────┘          │
│                                     │                                        │
│          ┌──────────────────────────┼──────────────────────────┐           │
│          ▼                          ▼                          ▼            │
│   ┌─────────────┐           ┌─────────────┐           ┌─────────────┐      │
│   │   Region    │           │   Region    │           │   Region    │      │
│   │   US-East   │           │   Europe    │           │   Asia      │      │
│   └──────┬──────┘           └──────┬──────┘           └──────┬──────┘      │
│          │                         │                          │             │
│          └─────────────────────────┼──────────────────────────┘            │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                        Query Processing                          │      │
│   │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐    │      │
│   │  │  Query    │  │  Spell    │  │  Query    │  │  Query    │    │      │
│   │  │  Parser   │─▶│  Check    │─▶│  Expansion│─▶│  Rewrite  │    │      │
│   │  └───────────┘  └───────────┘  └───────────┘  └───────────┘    │      │
│   └────────────────────────────────┬────────────────────────────────┘      │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                       Index Servers                              │      │
│   │  ┌─────────────────────────────────────────────────────────┐   │      │
│   │  │                   Document Index                         │   │      │
│   │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │   │      │
│   │  │  │ Shard 1 │ │ Shard 2 │ │ Shard 3 │ │ Shard N │       │   │      │
│   │  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │   │      │
│   │  └─────────────────────────────────────────────────────────┘   │      │
│   └────────────────────────────────┬────────────────────────────────┘      │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │                       Ranking Pipeline                           │      │
│   │                                                                   │      │
│   │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐    │      │
│   │  │  Initial  │  │  Quality  │  │    ML     │  │   Final   │    │      │
│   │  │   Score   │─▶│  Signals  │─▶│  Ranking  │─▶│   Sort    │    │      │
│   │  └───────────┘  └───────────┘  └───────────┘  └───────────┘    │      │
│   └────────────────────────────────┬────────────────────────────────┘      │
│                                    │                                        │
│                                    ▼                                        │
│                           ┌─────────────┐                                   │
│                           │   Results   │                                   │
│                           └─────────────┘                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Crawling Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                        Web Crawler                               │
│                                                                  │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│   │   URL        │    │   Fetcher    │    │   Content    │     │
│   │   Frontier   │───▶│   Workers    │───▶│   Parser     │     │
│   │              │    │   (1000s)    │    │              │     │
│   └──────────────┘    └──────────────┘    └──────────────┘     │
│          ▲                   │                    │              │
│          │                   │                    │              │
│          │            ┌──────▼──────┐            │              │
│          │            │   DNS       │            │              │
│          │            │   Cache     │            │              │
│          │            └─────────────┘            │              │
│          │                                       │              │
│          │            ┌──────────────────────────▼─────────┐   │
│          │            │              Indexer                │   │
│          │            │  ┌─────────┐  ┌─────────┐          │   │
│          └────────────│  │ Extract │  │ Forward │          │   │
│            new links  │  │  Links  │  │  Index  │          │   │
│                       │  └─────────┘  └─────────┘          │   │
│                       └────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Index Structure

```python
# Inverted Index Structure
class InvertedIndex:
    """
    Term -> List of (DocID, positions, term_frequency)
    """
    def __init__(self):
        self.index = {}  # term -> posting_list

    def add_document(self, doc_id: str, content: str):
        terms = self.tokenize(content)
        for position, term in enumerate(terms):
            if term not in self.index:
                self.index[term] = []
            self.index[term].append(Posting(
                doc_id=doc_id,
                position=position,
                tf=terms.count(term)
            ))

    def search(self, query: str) -> List[DocID]:
        terms = self.tokenize(query)
        # Intersect posting lists
        result = self.index.get(terms[0], [])
        for term in terms[1:]:
            result = self.intersect(result, self.index.get(term, []))
        return result
```

### Ranking Signals

| Signal Category | Examples | Weight |
|-----------------|----------|--------|
| **Content** | TF-IDF, keyword proximity | High |
| **PageRank** | Link authority, anchor text | High |
| **Freshness** | Last updated, content changes | Medium |
| **User Signals** | CTR, dwell time, pogo-sticking | High |
| **Quality** | Spam score, E-A-T signals | High |

### Deep Dive: Query Processing

```python
class QueryProcessor:
    def process(self, raw_query: str, context: UserContext) -> ProcessedQuery:
        # 1. Parse and tokenize
        tokens = self.tokenize(raw_query)

        # 2. Spell correction
        corrected = self.spell_check(tokens)

        # 3. Query expansion (synonyms, related terms)
        expanded = self.expand_query(corrected)

        # 4. Personalization
        personalized = self.personalize(expanded, context)

        # 5. Query rewriting for efficiency
        optimized = self.optimize_for_index(personalized)

        return ProcessedQuery(
            original=raw_query,
            tokens=optimized,
            intent=self.classify_intent(raw_query)
        )
```

### Interview Discussion Points

1. **Sharding Strategy**: How to partition the index? (By document, by term, hybrid)
2. **Cache Layers**: Query result cache, document cache, term cache
3. **Ranking Updates**: How often to recompute PageRank?
4. **Freshness vs Quality**: Trade-offs in real-time indexing

---

## Case Study 2: Design Google Maps

### Problem Statement
Design a mapping service that provides directions, real-time traffic, and location search.

### Scale Estimation

```
Users: 1B monthly active users
Daily requests: 100M direction requests
Map tiles: Trillions of tiles at various zoom levels
Roads: Billions of road segments
POIs: 200M+ points of interest
```

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Google Maps Architecture                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                              ┌─────────────┐                                │
│                              │   Client    │                                │
│                              │   (Mobile/  │                                │
│                              │    Web)     │                                │
│                              └──────┬──────┘                                │
│                                     │                                        │
│   ┌─────────────────────────────────┼─────────────────────────────────┐    │
│   │                          API Gateway                               │    │
│   └───┬─────────────┬──────────────┼──────────────┬────────────────┬──┘    │
│       │             │              │              │                │        │
│       ▼             ▼              ▼              ▼                ▼        │
│  ┌─────────┐  ┌──────────┐  ┌───────────┐  ┌──────────┐  ┌─────────────┐  │
│  │  Map    │  │ Location │  │ Routing   │  │ Traffic  │  │   Search    │  │
│  │  Tile   │  │  Search  │  │  Engine   │  │  Engine  │  │   Service   │  │
│  │ Service │  │  Service │  │           │  │          │  │             │  │
│  └────┬────┘  └────┬─────┘  └─────┬─────┘  └────┬─────┘  └──────┬──────┘  │
│       │            │              │              │               │         │
│       │            │              │              │               │         │
│   ┌───▼────────────▼──────────────▼──────────────▼───────────────▼───┐    │
│   │                        Data Layer                                 │    │
│   │                                                                   │    │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │    │
│   │  │  Map Data   │  │   Graph     │  │   Traffic   │              │    │
│   │  │  (Tiles,    │  │   (Roads,   │  │   (Real-    │              │    │
│   │  │   POIs)     │  │    Nodes)   │  │    time)    │              │    │
│   │  └─────────────┘  └─────────────┘  └─────────────┘              │    │
│   └───────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Map Tile System

```
┌─────────────────────────────────────────────────────────────────┐
│                        Tile Pyramid                              │
│                                                                  │
│   Zoom 0:    ┌───────────────────────────────────────┐          │
│              │                1 tile                  │          │
│              │               (world)                  │          │
│              └───────────────────────────────────────┘          │
│                                                                  │
│   Zoom 1:    ┌─────────────┬─────────────┐                      │
│              │      1      │      2      │                      │
│              ├─────────────┼─────────────┤  4 tiles             │
│              │      3      │      4      │                      │
│              └─────────────┴─────────────┘                      │
│                                                                  │
│   Zoom 2:    ┌──────┬──────┬──────┬──────┐                      │
│              │  1   │  2   │  3   │  4   │                      │
│              ├──────┼──────┼──────┼──────┤  16 tiles            │
│              │  5   │  6   │  7   │  8   │                      │
│              ├──────┼──────┼──────┼──────┤                      │
│              │  9   │  10  │  11  │  12  │                      │
│              ├──────┼──────┼──────┼──────┤                      │
│              │  13  │  14  │  15  │  16  │                      │
│              └──────┴──────┴──────┴──────┘                      │
│                                                                  │
│   Zoom 20:   Billions of tiles (street-level detail)            │
│                                                                  │
│   Tile URL: /tiles/{zoom}/{x}/{y}.png                           │
│   Total tiles at zoom Z: 4^Z                                     │
└─────────────────────────────────────────────────────────────────┘
```

### Routing Algorithm

```python
class RoutingEngine:
    def find_route(
        self,
        origin: LatLng,
        destination: LatLng,
        mode: TransportMode
    ) -> Route:
        # 1. Map matching - snap to nearest road
        start_node = self.map_match(origin)
        end_node = self.map_match(destination)

        # 2. Hierarchical routing
        if self.distance(start_node, end_node) > 50_km:
            # Use highway graph for long distances
            route = self.hierarchical_dijkstra(start_node, end_node)
        else:
            # Use detailed graph for short distances
            route = self.bidirectional_dijkstra(start_node, end_node)

        # 3. Apply real-time traffic
        route = self.apply_traffic(route)

        # 4. Generate turn-by-turn instructions
        route.instructions = self.generate_instructions(route)

        return route

    def hierarchical_dijkstra(self, start, end):
        """
        Contraction Hierarchies algorithm:
        1. Preprocess: Contract nodes by importance
        2. Query: Bidirectional search on contracted graph
        """
        # Find path up the hierarchy from start
        forward_path = self.search_up(start)
        # Find path up the hierarchy from end
        backward_path = self.search_up(end)
        # Connect via highway-level nodes
        return self.merge_paths(forward_path, backward_path)
```

### Traffic Data Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                    Real-Time Traffic Pipeline                    │
│                                                                  │
│   Data Sources:                                                  │
│   ┌───────────────────────────────────────────────────────┐     │
│   │  📱 Phone GPS   🚗 Sensors   🛰️ Satellite   📡 Partners│     │
│   └───────────────────────────────┬───────────────────────┘     │
│                                   │                              │
│                                   ▼                              │
│   ┌───────────────────────────────────────────────────────┐     │
│   │                    Pub/Sub Ingestion                   │     │
│   │                    (millions msg/sec)                  │     │
│   └───────────────────────────────┬───────────────────────┘     │
│                                   │                              │
│                                   ▼                              │
│   ┌───────────────────────────────────────────────────────┐     │
│   │                  Dataflow Processing                   │     │
│   │  • Map matching (GPS to road segment)                  │     │
│   │  • Speed aggregation (per segment)                     │     │
│   │  • Anomaly detection                                   │     │
│   │  • Incident inference                                  │     │
│   └───────────────────────────────┬───────────────────────┘     │
│                                   │                              │
│                                   ▼                              │
│   ┌───────────────────────────────────────────────────────┐     │
│   │                  Traffic State Store                   │     │
│   │           (Bigtable, segment_id → speed)              │     │
│   └───────────────────────────────────────────────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Tile Storage** | CDN + Object Storage | High cache-hit rate for tiles |
| **Routing Graph** | Contraction Hierarchies | O(log n) query time |
| **Traffic Updates** | Streaming | Sub-minute freshness |
| **Location Search** | Geohash + Text Search | Efficient spatial queries |

---

## Case Study 3: Design Google Drive

### Problem Statement
Design a cloud file storage and synchronization system.

### Requirements

**Functional:**
- Upload/download files up to 5TB
- Folder hierarchy
- Real-time sync across devices
- Sharing with permissions
- Version history
- Search within files

**Non-Functional:**
- 99.99% durability
- 99.9% availability
- Sync latency < 10s for small files
- Handle 1B+ files per user

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Google Drive Architecture                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                           Client Layer                                │  │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐  │  │
│   │  │  Desktop │  │  Mobile  │  │   Web    │  │  Local File System   │  │  │
│   │  │   Sync   │  │   App    │  │   App    │  │      Watcher         │  │  │
│   │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └──────────┬───────────┘  │  │
│   └───────┼─────────────┼─────────────┼───────────────────┼──────────────┘  │
│           │             │             │                   │                  │
│           └─────────────┴──────┬──────┴───────────────────┘                  │
│                                │                                              │
│                                ▼                                              │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │                           API Gateway                                 │  │
│   └────────────────────────────────┬─────────────────────────────────────┘  │
│                                    │                                         │
│          ┌─────────────────────────┼──────────────────────┐                 │
│          │                         │                      │                  │
│          ▼                         ▼                      ▼                  │
│   ┌─────────────┐          ┌─────────────┐        ┌─────────────┐           │
│   │  Metadata   │          │   Upload    │        │  Download   │           │
│   │   Service   │          │   Service   │        │   Service   │           │
│   │             │          │             │        │             │           │
│   │  • Folder   │          │  • Chunking │        │  • Chunk    │           │
│   │    hierarchy│          │  • Dedup    │        │    assembly │           │
│   │  • Sharing  │          │  • Compress │        │  • CDN      │           │
│   │  • Versions │          │             │        │             │           │
│   └──────┬──────┘          └──────┬──────┘        └──────┬──────┘           │
│          │                        │                      │                   │
│          │                        │                      │                   │
│          ▼                        ▼                      ▼                   │
│   ┌─────────────┐          ┌─────────────────────────────────┐              │
│   │   Spanner   │          │         Colossus (GFS)          │              │
│   │  (Metadata) │          │         (File Content)          │              │
│   └─────────────┘          └─────────────────────────────────┘              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### File Chunking & Deduplication

```python
class ChunkManager:
    CHUNK_SIZE = 4 * 1024 * 1024  # 4MB chunks

    def upload_file(self, file: File, user_id: str) -> FileMetadata:
        chunks = []

        for chunk_data in self.split_into_chunks(file):
            # Content-addressable storage
            chunk_hash = sha256(chunk_data)

            # Check if chunk already exists (deduplication)
            if not self.chunk_exists(chunk_hash):
                self.store_chunk(chunk_hash, chunk_data)

            chunks.append(ChunkRef(
                hash=chunk_hash,
                size=len(chunk_data),
                offset=chunks[-1].offset + chunks[-1].size if chunks else 0
            ))

        # Store metadata
        return self.create_file_metadata(
            user_id=user_id,
            name=file.name,
            chunks=chunks,
            version=1
        )
```

### Sync Protocol

```
┌─────────────────────────────────────────────────────────────────┐
│                      Sync Protocol Flow                          │
│                                                                  │
│   Local Change:                                                  │
│   ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐         │
│   │ File   │───▶│ Detect │───▶│ Upload │───▶│ Update │         │
│   │ Change │    │ Change │    │ Chunks │    │ Server │         │
│   └────────┘    └────────┘    └────────┘    └────────┘         │
│                                                   │              │
│                                                   ▼              │
│   Remote Change:                           ┌──────────┐         │
│   ┌────────┐    ┌────────┐    ┌────────┐  │  Notify  │         │
│   │ Apply  │◀───│Download│◀───│ Receive│◀─│  Other   │         │
│   │ Change │    │ Chunks │    │ Update │  │  Clients │         │
│   └────────┘    └────────┘    └────────┘  └──────────┘         │
│                                                                  │
│   Conflict Resolution:                                           │
│   ┌────────────────────────────────────────────────────────┐    │
│   │  • Last-write-wins for simple edits                    │    │
│   │  • Both versions kept for conflicts                    │    │
│   │  • Operational transform for real-time docs            │    │
│   └────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Trade-offs

| Trade-off | Choice | Why |
|-----------|--------|-----|
| **Consistency vs Sync Speed** | Eventual consistency | Better UX, conflicts rare |
| **Chunk Size** | 4MB | Balance between dedup and overhead |
| **Storage Backend** | Colossus | Native integration, durability |
| **Notification** | Long polling + Push | Works across all clients |

---

## Case Study 4: Design YouTube

### Problem Statement
Design a video streaming platform at global scale.

### Scale Estimation

```
Videos watched: 1B hours/day
Videos uploaded: 500 hours of video/minute
Storage: Hundreds of petabytes
Concurrent viewers: 100M+ at peak
```

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          YouTube Architecture                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         Upload Pipeline                              │   │
│   │                                                                      │   │
│   │   ┌────────┐   ┌─────────┐   ┌─────────────┐   ┌───────────────┐   │   │
│   │   │ Upload │──▶│Validate │──▶│  Transcode  │──▶│ Store in CDN  │   │   │
│   │   │        │   │         │   │  (Multiple  │   │   (Global)    │   │   │
│   │   │        │   │         │   │  qualities) │   │               │   │   │
│   │   └────────┘   └─────────┘   └─────────────┘   └───────────────┘   │   │
│   │                                     │                               │   │
│   │                              ┌──────▼──────┐                       │   │
│   │                              │   Generate  │                       │   │
│   │                              │  Thumbnails │                       │   │
│   │                              │  + Captions │                       │   │
│   │                              └─────────────┘                       │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        Viewing Pipeline                              │   │
│   │                                                                      │   │
│   │    ┌──────────┐   ┌──────────────┐   ┌──────────────────────────┐  │   │
│   │    │  Player  │──▶│ Edge Server  │◀──│   Origin (Colossus)     │  │   │
│   │    │          │   │  (CDN PoP)   │   │                          │  │   │
│   │    │ Adaptive │   │              │   │   • Original videos      │  │   │
│   │    │ Bitrate  │   │  • Cache     │   │   • All transcodes       │  │   │
│   │    └──────────┘   │  • Popular   │   │   • Thumbnails           │  │   │
│   │                   │    videos    │   │                          │  │   │
│   │                   └──────────────┘   └──────────────────────────┘  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                     Recommendation System                            │   │
│   │                                                                      │   │
│   │    ┌────────────┐   ┌────────────┐   ┌────────────┐                │   │
│   │    │  Candidate │──▶│  Ranking   │──▶│ Diversifi- │                │   │
│   │    │ Generation │   │    Model   │   │  cation    │                │   │
│   │    │  (1000s)   │   │   (Top K)  │   │            │                │   │
│   │    └────────────┘   └────────────┘   └────────────┘                │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Video Transcoding Pipeline

```python
class TranscodingPipeline:
    QUALITY_PROFILES = [
        {"resolution": "2160p", "bitrate": "20Mbps"},
        {"resolution": "1440p", "bitrate": "10Mbps"},
        {"resolution": "1080p", "bitrate": "5Mbps"},
        {"resolution": "720p", "bitrate": "2.5Mbps"},
        {"resolution": "480p", "bitrate": "1Mbps"},
        {"resolution": "360p", "bitrate": "0.5Mbps"},
    ]

    def transcode(self, video_id: str, source: VideoFile):
        # Split video into segments for parallel processing
        segments = self.split_video(source, segment_duration=4)  # 4 second segments

        for profile in self.QUALITY_PROFILES:
            # Parallel transcode each segment
            transcoded_segments = parallel_map(
                lambda seg: self.transcode_segment(seg, profile),
                segments
            )

            # Generate manifest for adaptive streaming
            manifest = self.generate_dash_manifest(
                video_id, profile, transcoded_segments
            )

            self.store_to_cdn(video_id, profile, transcoded_segments, manifest)
```

### CDN Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                    YouTube CDN Architecture                      │
│                                                                  │
│   Tier 1: Browser/App Cache                                      │
│   ├── Next few seconds pre-buffered                              │
│                                                                  │
│   Tier 2: ISP Cache Nodes                                        │
│   ├── Most popular videos cached at ISP level                   │
│   ├── Negotiated peering arrangements                            │
│                                                                  │
│   Tier 3: Google Edge PoPs (100+ locations)                     │
│   ├── All popular videos                                         │
│   ├── Recent uploads                                             │
│   ├── 95%+ cache hit rate                                        │
│                                                                  │
│   Tier 4: Regional Data Centers                                  │
│   ├── Full video catalog                                         │
│   └── Transcode on demand for rare formats                       │
│                                                                  │
│   Tier 5: Cold Storage                                           │
│   └── Rarely accessed videos (tape/archive)                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Framework

### Time Allocation (45 min)

| Phase | Time | Activities |
|-------|------|------------|
| **Clarify** | 5 min | Scale, features, constraints |
| **Requirements** | 5 min | Functional + Non-functional |
| **High-Level** | 15 min | Components, data flow |
| **Deep Dive** | 15 min | 1-2 components in detail |
| **Wrap-up** | 5 min | Trade-offs, extensions |

### Common Follow-ups

1. "How would you handle a datacenter failure?"
2. "What if traffic increases 10x suddenly?"
3. "How do you ensure data consistency across regions?"
4. "Walk me through a request from start to finish"

---

## Related Topics

- [[00_google_overview|Google Prep Overview]]
- [[02_google_topics|Google-Specific Topics]]
- [[04_infrastructure_deep_dives|Infrastructure Deep Dives]]

---

**Tags**: #google #case-study #search #maps #drive #youtube #system-design
