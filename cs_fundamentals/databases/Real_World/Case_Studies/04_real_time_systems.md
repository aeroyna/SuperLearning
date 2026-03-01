# Real-Time Systems

## Real-Time Database Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│              Real-Time System Characteristics                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LATENCY REQUIREMENTS                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Hard real-time:    < 10ms (trading, gaming)               │ │
│  │ Soft real-time:    < 100ms (web, notifications)           │ │
│  │ Near real-time:    < 1s (dashboards, monitoring)          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMMON USE CASES                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Live dashboards and monitoring                          │ │
│  │ • Gaming leaderboards                                      │ │
│  │ • Financial trading                                        │ │
│  │ • IoT sensor processing                                    │ │
│  │ • Fraud detection                                          │ │
│  │ • Real-time recommendations                                │ │
│  │ • Chat and messaging                                       │ │
│  │ • Location tracking                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  KEY CHALLENGES                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Predictable, low-latency reads and writes               │ │
│  │ • High throughput with consistent performance             │ │
│  │ • Handling traffic spikes                                  │ │
│  │ • Data freshness vs consistency trade-offs                │ │
│  │ • Scaling without adding latency                          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Real-Time Architecture Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│              Event-Driven Architecture                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  Event Sources                           │    │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐          │    │
│  │  │ App  │ │ IoT  │ │ API  │ │ DB   │ │ User │          │    │
│  │  │Events│ │Sensor│ │Calls │ │ CDC  │ │Action│          │    │
│  │  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘          │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Event Streaming Platform                    │    │
│  │  ┌─────────────────────────────────────────────────┐    │    │
│  │  │           Apache Kafka / Pulsar / Kinesis       │    │    │
│  │  │                                                  │    │    │
│  │  │  Topic: orders    Topic: clicks   Topic: sensors│    │    │
│  │  └─────────────────────────────────────────────────┘    │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│      ┌────────────────────┼────────────────────┐                │
│      │                    │                    │                 │
│      ▼                    ▼                    ▼                 │
│  ┌───────────┐      ┌───────────┐      ┌───────────┐           │
│  │  Stream   │      │  Stream   │      │  Stream   │           │
│  │ Processor │      │ Processor │      │ Processor │           │
│  │(Flink/KSQL│      │(Flink/KSQL│      │(Flink/KSQL│           │
│  └─────┬─────┘      └─────┬─────┘      └─────┬─────┘           │
│        │                  │                  │                   │
│        ▼                  ▼                  ▼                   │
│  ┌───────────┐      ┌───────────┐      ┌───────────┐           │
│  │   Redis   │      │ ClickHouse│      │  Elastic  │           │
│  │  (State)  │      │(Analytics)│      │ (Search)  │           │
│  └───────────┘      └───────────┘      └───────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

## Redis for Real-Time

```
┌─────────────────────────────────────────────────────────────────┐
│              Redis Real-Time Patterns                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LEADERBOARD (Sorted Sets)                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Update score                                              │ │
│  │ ZADD leaderboard 1500 "player:123"                         │ │
│  │                                                             │ │
│  │ # Get top 10                                                │ │
│  │ ZREVRANGE leaderboard 0 9 WITHSCORES                       │ │
│  │                                                             │ │
│  │ # Get player rank                                           │ │
│  │ ZREVRANK leaderboard "player:123"                          │ │
│  │                                                             │ │
│  │ Performance: O(log N) operations                           │ │
│  │ Use case: Gaming, competitions, sales rankings             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RATE LIMITING (Sliding Window)                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Using sorted set with timestamps                         │ │
│  │ ZADD requests:user:123 <timestamp> <request_id>            │ │
│  │ ZREMRANGEBYSCORE requests:user:123 0 <old_timestamp>       │ │
│  │ ZCARD requests:user:123                                     │ │
│  │                                                             │ │
│  │ # Or use Redis Cell module (GCRA algorithm)                │ │
│  │ CL.THROTTLE user:123 100 60 1                              │ │
│  │ # 100 requests per 60 seconds                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PUB/SUB for Real-Time Updates                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Publisher                                                 │ │
│  │ PUBLISH price:BTC '{"price": 42000, "time": 1703721600}'  │ │
│  │                                                             │ │
│  │ # Subscriber                                                │ │
│  │ SUBSCRIBE price:BTC price:ETH                              │ │
│  │                                                             │ │
│  │ Note: Fire-and-forget, no persistence                      │ │
│  │ For durable: Use Redis Streams                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  REDIS STREAMS (Kafka-like)                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Add event                                                 │ │
│  │ XADD events * type click user_id 123 page /home            │ │
│  │                                                             │ │
│  │ # Read new events                                           │ │
│  │ XREAD BLOCK 1000 STREAMS events $                          │ │
│  │                                                             │ │
│  │ # Consumer groups for scaling                              │ │
│  │ XGROUP CREATE events mygroup $                             │ │
│  │ XREADGROUP GROUP mygroup consumer1 STREAMS events >        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Time-Series Databases

```
┌─────────────────────────────────────────────────────────────────┐
│              Time-Series Database Patterns                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  USE CASES                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • IoT sensor data                                          │ │
│  │ • Application metrics                                       │ │
│  │ • Financial tick data                                       │ │
│  │ • Log analytics                                             │ │
│  │ • Network monitoring                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TIMESCALEDB (PostgreSQL extension)                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE metrics (                                      │ │
│  │   time TIMESTAMPTZ NOT NULL,                               │ │
│  │   device_id TEXT,                                           │ │
│  │   temperature DOUBLE PRECISION,                            │ │
│  │   humidity DOUBLE PRECISION                                │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ SELECT create_hypertable('metrics', 'time');               │ │
│  │                                                             │ │
│  │ -- Automatic partitioning by time                          │ │
│  │ -- Compression for old data                                │ │
│  │ -- Continuous aggregates (materialized views)              │ │
│  │                                                             │ │
│  │ SELECT time_bucket('1 hour', time) AS hour,                │ │
│  │        AVG(temperature)                                     │ │
│  │ FROM metrics                                                 │ │
│  │ WHERE time > NOW() - INTERVAL '1 day'                      │ │
│  │ GROUP BY hour;                                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  INFLUXDB                                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ // Write (Line Protocol)                                   │ │
│  │ weather,location=NYC temp=72.5,humidity=65 1703721600      │ │
│  │                                                             │ │
│  │ // Query (Flux)                                             │ │
│  │ from(bucket: "sensors")                                     │ │
│  │   |> range(start: -1h)                                     │ │
│  │   |> filter(fn: (r) => r._measurement == "weather")        │ │
│  │   |> mean()                                                 │ │
│  │                                                             │ │
│  │ Features:                                                   │ │
│  │ • Purpose-built for time-series                            │ │
│  │ • High write throughput                                    │ │
│  │ • Automatic downsampling                                   │ │
│  │ • Retention policies                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Real-Time Analytics

```
┌─────────────────────────────────────────────────────────────────┐
│              Real-Time Analytics Stack                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CLICKHOUSE (Column store)                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE events (                                       │ │
│  │   timestamp DateTime,                                       │ │
│  │   user_id UInt64,                                          │ │
│  │   event_type String,                                        │ │
│  │   properties String  -- JSON                               │ │
│  │ ) ENGINE = MergeTree()                                      │ │
│  │ PARTITION BY toYYYYMM(timestamp)                           │ │
│  │ ORDER BY (event_type, user_id, timestamp);                 │ │
│  │                                                             │ │
│  │ -- Billions of rows, sub-second queries                    │ │
│  │ SELECT                                                      │ │
│  │   event_type,                                               │ │
│  │   count() AS cnt,                                           │ │
│  │   uniq(user_id) AS users                                   │ │
│  │ FROM events                                                  │ │
│  │ WHERE timestamp > now() - INTERVAL 1 HOUR                  │ │
│  │ GROUP BY event_type;                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  APACHE DRUID                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Sub-second OLAP queries                                  │ │
│  │ • Native Kafka ingestion                                   │ │
│  │ • High concurrency (1000s of queries/sec)                  │ │
│  │ • Automatic data tiering                                   │ │
│  │                                                             │ │
│  │ Use cases:                                                  │ │
│  │ • User-facing analytics                                    │ │
│  │ • Real-time dashboards                                     │ │
│  │ • Network/security monitoring                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  APACHE PINOT (LinkedIn)                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Designed for user-facing analytics                       │ │
│  │ • Low latency on large datasets                            │ │
│  │ • Real-time + batch hybrid                                 │ │
│  │ • Used by LinkedIn, Uber, Stripe                           │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Stream Processing

```
┌─────────────────────────────────────────────────────────────────┐
│              Stream Processing Patterns                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  APACHE FLINK                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ DataStream<Event> events = env                             │ │
│  │   .addSource(new KafkaSource<>(...));                      │ │
│  │                                                             │ │
│  │ events                                                       │ │
│  │   .keyBy(event -> event.getUserId())                       │ │
│  │   .window(TumblingEventTimeWindows.of(Time.minutes(5)))    │ │
│  │   .aggregate(new CountAggregator())                        │ │
│  │   .addSink(new RedisSink<>(...));                          │ │
│  │                                                             │ │
│  │ Capabilities:                                               │ │
│  │ • Exactly-once semantics                                   │ │
│  │ • Event time processing                                    │ │
│  │ • Stateful computations                                    │ │
│  │ • Complex event processing (CEP)                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  KSQLDB (Kafka Streams)                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Create stream from Kafka topic                          │ │
│  │ CREATE STREAM clicks (                                      │ │
│  │   user_id VARCHAR,                                          │ │
│  │   page VARCHAR,                                              │ │
│  │   timestamp BIGINT                                          │ │
│  │ ) WITH (                                                     │ │
│  │   KAFKA_TOPIC='clicks',                                     │ │
│  │   VALUE_FORMAT='JSON'                                       │ │
│  │ );                                                           │ │
│  │                                                             │ │
│  │ -- Real-time aggregation                                   │ │
│  │ CREATE TABLE clicks_per_page AS                            │ │
│  │ SELECT page, COUNT(*) AS click_count                       │ │
│  │ FROM clicks                                                  │ │
│  │ WINDOW TUMBLING (SIZE 1 MINUTE)                            │ │
│  │ GROUP BY page;                                              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Best Practices

```
┌─────────────────────────────────────────────────────────────────┐
│              Real-Time System Best Practices                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LATENCY OPTIMIZATION                                           │
│  • Keep data close to compute (co-location)                    │
│  • Use in-memory stores for hot data                           │
│  • Pre-compute when possible                                    │
│  • Avoid cross-datacenter calls in hot path                    │
│                                                                  │
│  RELIABILITY                                                    │
│  • Design for failure (circuit breakers, retries)              │
│  • Use async processing for non-critical paths                 │
│  • Implement backpressure handling                             │
│  • Monitor end-to-end latency percentiles                      │
│                                                                  │
│  SCALING                                                        │
│  • Partition data for parallel processing                      │
│  • Use consumer groups for horizontal scaling                  │
│  • Implement graceful degradation                              │
│  • Cache aggressively with smart invalidation                  │
│                                                                  │
│  DATA FRESHNESS VS CONSISTENCY                                  │
│  • Accept eventual consistency where possible                  │
│  • Use event sourcing for audit trails                         │
│  • Implement idempotent operations                             │
│  • Track data lag metrics                                      │
└─────────────────────────────────────────────────────────────────┘
```
