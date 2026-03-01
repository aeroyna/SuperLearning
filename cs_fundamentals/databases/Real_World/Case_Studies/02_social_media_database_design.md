# Social Media Database Design

## Requirements Analysis

```
┌─────────────────────────────────────────────────────────────────┐
│              Social Media Requirements                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FUNCTIONAL REQUIREMENTS                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • User profiles and authentication                         │ │
│  │ • Posts (text, images, videos)                             │ │
│  │ • Social graph (follow/friend relationships)               │ │
│  │ • News feed / timeline                                     │ │
│  │ • Likes, comments, shares                                  │ │
│  │ • Direct messaging                                         │ │
│  │ • Notifications                                            │ │
│  │ • Search (users, posts, hashtags)                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NON-FUNCTIONAL REQUIREMENTS                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Extreme read-heavy (100:1 read/write ratio)              │ │
│  │ • Sub-second feed loading                                  │ │
│  │ • Eventual consistency acceptable for feeds               │ │
│  │ • Handle viral content (millions of interactions)          │ │
│  │ • Global distribution                                      │ │
│  │ • Handle celebrity accounts (millions of followers)        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SCALE CHALLENGES                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Fan-out problem (celebrity posts to millions)            │ │
│  │ • Hot spots (viral posts, trending topics)                 │ │
│  │ • Graph queries (friends of friends)                       │ │
│  │ • Real-time notifications at scale                         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Data Model

```
┌─────────────────────────────────────────────────────────────────┐
│              Social Media Entity Model                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐        │
│  │  Users   │────<│   Follows    │>────│   Users      │        │
│  └──────────┘     └──────────────┘     └──────────────┘        │
│       │                                       │                  │
│       │                                       │                  │
│       ▼                                       │                  │
│  ┌──────────┐     ┌──────────────┐            │                 │
│  │  Posts   │────<│   Likes      │────────────┘                 │
│  └──────────┘     └──────────────┘                              │
│       │                                                          │
│       │           ┌──────────────┐                              │
│       └──────────<│  Comments    │                              │
│                   └──────────────┘                              │
│                                                                  │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐        │
│  │  Users   │────<│  Messages    │>────│   Users      │        │
│  └──────────┘     └──────────────┘     └──────────────┘        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Technology Stack

```
┌─────────────────────────────────────────────────────────────────┐
│              Social Media Database Stack                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                       Application                          │ │
│  └────────────────────────────┬───────────────────────────────┘ │
│                               │                                  │
│      ┌─────────┬─────────┬────┴────┬─────────┬─────────┐       │
│      │         │         │         │         │         │        │
│      ▼         ▼         ▼         ▼         ▼         ▼        │
│                                                                  │
│  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐   │
│  │MySQL/ │ │Redis  │ │Cassan-│ │Neo4j/ │ │Elast- │ │S3/CDN │   │
│  │Postgr │ │       │ │dra    │ │TAO    │ │search │ │       │   │
│  └───────┘ └───────┘ └───────┘ └───────┘ └───────┘ └───────┘   │
│     │         │          │         │         │         │        │
│  Users     Sessions   Timeline   Social    Search    Media     │
│  Posts     Cache      Feed      Graph     Users              │
│  Auth      Counters   Activity  Friends   Posts              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Feed Generation Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│              News Feed Architecture                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PULL MODEL (Fan-in on read)                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  User requests feed:                                        │ │
│  │  1. Get list of followed users                             │ │
│  │  2. Fetch recent posts from each                           │ │
│  │  3. Merge and rank                                          │ │
│  │  4. Return to user                                          │ │
│  │                                                             │ │
│  │  ✓ Storage efficient (posts stored once)                  │ │
│  │  ✓ Fresh content                                           │ │
│  │  ✗ Slow for users following many accounts                 │ │
│  │  ✗ High read load                                          │ │
│  │                                                             │ │
│  │  Best for: Users following few accounts                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PUSH MODEL (Fan-out on write)                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  When user posts:                                           │ │
│  │  1. Write post to posts table                              │ │
│  │  2. Get all followers                                       │ │
│  │  3. Push post ID to each follower's feed cache             │ │
│  │                                                             │ │
│  │  User requests feed:                                        │ │
│  │  1. Read pre-computed feed from cache                      │ │
│  │                                                             │ │
│  │  ✓ Fast reads                                               │ │
│  │  ✗ High write amplification                                │ │
│  │  ✗ Celebrity problem (millions of followers)               │ │
│  │  ✗ Stale for inactive users                                │ │
│  │                                                             │ │
│  │  Best for: Regular users with normal follower counts       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  HYBRID MODEL (Twitter/Instagram approach)                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Regular users (< 10K followers): Push model               │ │
│  │  Celebrities (> 10K followers): Pull model                 │ │
│  │                                                             │ │
│  │  Feed generation:                                           │ │
│  │  1. Get pre-computed feed (pushed posts)                   │ │
│  │  2. Fetch posts from followed celebrities                  │ │
│  │  3. Merge both sets                                         │ │
│  │  4. Apply ranking algorithm                                │ │
│  │                                                             │ │
│  │  ┌─────────────────────────────────────────────────┐       │ │
│  │  │ Follower Count │  Strategy   │  Reason         │       │ │
│  │  │────────────────┼─────────────┼─────────────────│       │ │
│  │  │ < 1,000        │ Push        │ Low fan-out     │       │ │
│  │  │ 1K - 100K      │ Push + Delay│ Async fan-out   │       │ │
│  │  │ > 100K         │ Pull        │ Too many writes │       │ │
│  │  └─────────────────────────────────────────────────┘       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Social Graph Storage

```
┌─────────────────────────────────────────────────────────────────┐
│              Social Graph Options                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  OPTION 1: Relational Table                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE follows (                                     │ │
│  │     follower_id BIGINT,                                    │ │
│  │     followee_id BIGINT,                                    │ │
│  │     created_at TIMESTAMP,                                  │ │
│  │     PRIMARY KEY (follower_id, followee_id)                │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ CREATE INDEX idx_followee ON follows(followee_id);         │ │
│  │                                                             │ │
│  │ Queries:                                                    │ │
│  │ -- Who do I follow?                                        │ │
│  │ SELECT followee_id FROM follows WHERE follower_id = ?;    │ │
│  │                                                             │ │
│  │ -- Who follows me?                                         │ │
│  │ SELECT follower_id FROM follows WHERE followee_id = ?;    │ │
│  │                                                             │ │
│  │ ✓ Simple, well-understood                                  │ │
│  │ ✗ Friends-of-friends queries expensive                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  OPTION 2: Graph Database (Neo4j)                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ (User)-[:FOLLOWS]->(User)                                  │ │
│  │                                                             │ │
│  │ // Who do I follow?                                        │ │
│  │ MATCH (me:User {id: $userId})-[:FOLLOWS]->(them)          │ │
│  │ RETURN them                                                 │ │
│  │                                                             │ │
│  │ // Friends of friends                                      │ │
│  │ MATCH (me:User {id: $userId})-[:FOLLOWS]->()              │ │
│  │       -[:FOLLOWS]->(suggestion)                            │ │
│  │ WHERE NOT (me)-[:FOLLOWS]->(suggestion)                   │ │
│  │ RETURN suggestion, count(*) as mutualFriends              │ │
│  │                                                             │ │
│  │ ✓ Graph queries natural and fast                          │ │
│  │ ✗ Additional system to maintain                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  OPTION 3: Facebook's TAO (Custom)                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Specialized graph store                                  │ │
│  │ • Objects (nodes) and Associations (edges)                │ │
│  │ • Optimized for social graph patterns                     │ │
│  │ • Caching layer over MySQL                                 │ │
│  │ • Eventually consistent                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Counters and Hot Data

```
┌─────────────────────────────────────────────────────────────────┐
│              Handling Counters at Scale                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PROBLEM: Viral post with millions of likes                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Naive approach:                                             │ │
│  │ UPDATE posts SET like_count = like_count + 1               │ │
│  │ WHERE id = ?;                                               │ │
│  │                                                             │ │
│  │ Issues:                                                     │ │
│  │ • Row lock contention                                       │ │
│  │ • Write amplification                                       │ │
│  │ • Database hot spot                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SOLUTION: Buffered counters with Redis                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  1. Increment in Redis (atomic, fast)                      │ │
│  │     INCR post:123:likes                                    │ │
│  │                                                             │ │
│  │  2. Periodic flush to database (every N seconds)           │ │
│  │     - Read all dirty counters                              │ │
│  │     - Batch update database                                │ │
│  │     - Reset Redis counters                                 │ │
│  │                                                             │ │
│  │  3. Read combines DB + Redis                               │ │
│  │     db_count + redis_delta                                 │ │
│  │                                                             │ │
│  │  Trade-off: Eventual consistency (seconds)                 │ │
│  │  Acceptable for like counts                                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ALTERNATIVE: Counter sharding                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Instead of one counter, use N shards:                      │ │
│  │                                                             │ │
│  │ post_likes_0, post_likes_1, ... post_likes_99             │ │
│  │                                                             │ │
│  │ Increment: Random shard                                    │ │
│  │ Read: Sum all shards                                       │ │
│  │                                                             │ │
│  │ Reduces contention by factor of N                          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Schema Example

```sql
-- Users (MySQL/PostgreSQL)
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    display_name VARCHAR(100),
    bio TEXT,
    avatar_url VARCHAR(500),
    follower_count INT DEFAULT 0,
    following_count INT DEFAULT 0,
    is_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Posts (MySQL/PostgreSQL)
CREATE TABLE posts (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    content TEXT,
    media_urls JSON,
    like_count INT DEFAULT 0,
    comment_count INT DEFAULT 0,
    share_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_created (user_id, created_at DESC)
);

-- Feed cache (Redis)
-- Key: feed:{user_id}
-- Value: Sorted set of post IDs with timestamp as score
-- ZADD feed:123 1703721600 "post:456"
-- ZREVRANGE feed:123 0 19  -- Get latest 20

-- Follows (Cassandra for scale)
-- Optimized for "get followers" and "get following"
CREATE TABLE followers (
    user_id BIGINT,
    follower_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, created_at, follower_id)
) WITH CLUSTERING ORDER BY (created_at DESC);

CREATE TABLE following (
    user_id BIGINT,
    following_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, created_at, following_id)
) WITH CLUSTERING ORDER BY (created_at DESC);
```
