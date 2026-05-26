# Pattern: Database Selection

**Appears in**: Every system design — always discuss storage trade-offs

---

## The Decision Framework

Ask these questions in order:

1. **What is the access pattern?** Key-value lookup? Range scan? Full-text search? Graph traversal?
2. **What are the consistency requirements?** Strong consistency or eventual OK?
3. **What is the scale?** Single node? Horizontal? Multi-region?
4. **What is the read/write ratio?** Read-heavy → indexes + replicas. Write-heavy → LSM, partitioning.
5. **Do you need transactions?** Multi-row ACID? Cross-shard?

---

## Postgres (SQL / Relational)

**Best for**: Structured data with relationships, ACID transactions, complex queries, moderate scale

| Strength | Detail |
|----------|--------|
| ACID transactions | Safe for financial data, inventory, anything requiring atomicity |
| Flexible querying | JOINs, aggregations, subqueries — no schema pre-planning needed |
| Mature ecosystem | pgvector, PostGIS, JSON support, rich extension library |
| Vertical scale | Single powerful node handles millions of rows easily |
| Read replicas | Streaming replication for read scale-out |

**Weaknesses**:
- Horizontal write scaling requires sharding (adds complexity)
- Schema migrations at scale are painful
- Not ideal for time-series or pure key-value at very high throughput

**Rule of thumb**: Start here unless you have a clear reason not to. < 500M rows + moderate QPS = Postgres is fine.

---

## Cassandra (Wide-Column / NoSQL)

**Best for**: Write-heavy, globally distributed, time-series, pure key-value at massive scale

| Strength | Detail |
|----------|--------|
| Horizontal scaling | Add nodes linearly; no primary bottleneck |
| Write throughput | LSM-tree writes to memory (MemTable) first → very fast |
| Multi-region | Built-in multi-master replication across DCs |
| Tunable consistency | Per-query: ONE, QUORUM, ALL |
| Time-series native | Wide rows model fits `(user_id, timestamp)` patterns |

**Weaknesses**:
- No JOINs — must denormalize; one table per query pattern
- No multi-row transactions (use lightweight transactions sparingly — slow)
- Data model design is query-driven — harder to evolve
- Eventual consistency by default — stale reads possible

**When to choose Cassandra**:
- Write QPS > 10K/s sustained
- Data naturally fits partition key → clustering key → value (e.g., user_id → timestamp → message)
- Multi-region replication is a hard requirement
- You can live with eventual consistency

**URL Shortener fit**: `short_code` as partition key, pure key-value → perfect Cassandra use case.

---

## Redis (In-Memory / Cache + Data Structure Store)

**Best for**: Caching, sessions, pub/sub, leaderboards, counters, queues

| Strength | Detail |
|----------|--------|
| Sub-millisecond latency | All operations in-memory |
| Rich data structures | Sorted sets, streams, hyperloglogs, bloom filters |
| Pub/Sub | Lightweight message passing |
| Atomic operations | INCR, LPUSH, etc. — race-condition-free counters |
| TTL support | Native expiry on any key |

**Weaknesses**:
- Memory-limited (not for primary storage of large datasets)
- Persistence optional — RDB/AOF adds latency; not a true DB replacement
- Single-threaded command execution

**Use as**: Cache layer in front of any DB, session store, rate-limit counter, distributed lock (Redlock), real-time leaderboard.

---

## DynamoDB (Managed Key-Value / Document)

**Best for**: Serverless, AWS-native, unpredictable traffic (auto-scaling), simple access patterns

| Strength | Detail |
|----------|--------|
| Fully managed | No ops burden |
| Auto-scaling | Handles traffic spikes automatically |
| Single-digit ms latency | At scale |
| DynamoDB Streams | CDC for downstream consumers |
| Global Tables | Multi-region replication |

**Weaknesses**:
- Expensive at high sustained throughput
- Query flexibility limited to partition + sort key
- Scan operations are expensive and slow
- Vendor lock-in

**Use when**: AWS shop, variable workload, don't want to manage infrastructure.

---

## Elasticsearch / OpenSearch

**Best for**: Full-text search, log analytics, faceted search

- Inverted index → fast text search across millions of documents
- Not a primary DB — typically used alongside Postgres/Cassandra as a search index
- Eventual consistency; not ACID
- Use for: product search, log/event analytics, autocomplete

---

## ClickHouse / OLAP Stores

**Best for**: Analytics queries over billions of rows (aggregations, GROUP BY, time-series analytics)

- Columnar storage → reads only needed columns
- Excellent for: "count clicks per URL per day", "top 10 URLs by country"
- Not for transactional workloads
- Write in bulk (batch insert); real-time inserts degrade performance

---

## Sharding Strategies (for any DB)

### Range Sharding
```
Shard 1: user_id 0–1M
Shard 2: user_id 1M–2M
...
```
- **Pros**: Simple; range queries stay on one shard
- **Cons**: Hot spots (newest users get all traffic)

### Hash Sharding
```
shard = hash(user_id) % num_shards
```
- **Pros**: Even distribution
- **Cons**: Range queries span all shards; resharding requires data migration

### Consistent Hashing
- Virtual nodes on a ring → each node owns a range
- Add/remove nodes: only adjacent ranges affected
- **Use when**: Cache clusters, distributed KV stores, Cassandra (uses this internally)

### Directory-Based Sharding
- Lookup table maps entity → shard
- **Pros**: Flexible; can move data without re-hashing
- **Cons**: Lookup table is a bottleneck/SPOF if not cached

---

## Replication Patterns

| Pattern | How | Use case |
|---------|-----|---------|
| **Leader-Follower** | Writes to leader, replicas sync async | Most common; reads scale-out; leader is write bottleneck |
| **Multi-Leader** | Multiple write nodes | Multi-region writes; conflict resolution complexity |
| **Leaderless (Quorum)** | Write to W nodes, read from R nodes; W+R > N | Cassandra, DynamoDB; no SPOF; tunable consistency |

---

## Quick Decision Table

| Scenario | Pick |
|----------|------|
| Financial transactions, complex queries | Postgres |
| High-write, time-series, multi-region | Cassandra |
| Caching, sessions, real-time counters | Redis |
| Full-text search | Elasticsearch |
| Analytics over billions of rows | ClickHouse |
| Serverless / AWS / variable traffic | DynamoDB |
| Graph relationships | Neo4j |

---

## Interview Tips

- Never just say "I'll use MySQL" or "I'll use NoSQL" without justification
- Walk through the framework: access pattern → consistency → scale → transactions
- Mention that you'd start with Postgres and only move to Cassandra when scale demands it
- Discuss sharding only when the data outgrows a single node — don't prematurely optimize
- Interviewers love hearing: "I'd add a Redis cache in front of the DB for the read path"
