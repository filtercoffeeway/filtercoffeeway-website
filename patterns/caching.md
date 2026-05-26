# Pattern: Caching

**Appears in**: URL Shortener, Feed systems, CDN design, any read-heavy system

---

## Caching Strategies

### 1. Cache-Aside (Lazy Loading)
```
App → check cache → hit: return
                  → miss: read DB → write to cache → return
```
- **Pros**: Only caches what's actually read; cache failure doesn't break reads
- **Cons**: First request always misses (cold start); stale data possible
- **Use when**: Read-heavy, unpredictable access patterns

### 2. Write-Through
```
App → write to cache AND DB simultaneously
```
- **Pros**: Cache always consistent with DB; no cold start on reads
- **Cons**: Write latency higher; cache fills with data that may never be read
- **Use when**: Write-heavy but every written item will likely be read soon

### 3. Write-Behind (Write-Back)
```
App → write to cache → return immediately
Cache → async flush to DB (batched)
```
- **Pros**: Very fast writes; good for burst workloads
- **Cons**: Data loss risk if cache crashes before flush; complex to implement
- **Use when**: High write throughput, tolerable eventual persistence (shopping cart, counters)

### 4. Read-Through
```
App → check cache → miss: cache itself fetches from DB → returns to app
```
- **Pros**: Cache manages DB fallback; simpler app code
- **Cons**: First read always misses; cache must know DB schema
- **Use when**: You want caching logic centralized (Redis + modules, Memcached + proxy)

---

## Cache Stampede (Thundering Herd)

**Problem**: A popular cached item expires. 10,000 simultaneous requests all miss → all hit DB simultaneously.

### Solutions

| Solution | How | Trade-off |
|----------|-----|-----------|
| **Mutex / lock** | First thread to miss acquires lock, fetches DB, populates cache; others wait | Serializes misses; adds latency on expiry |
| **Probabilistic early recompute** | Before TTL expires, randomly recompute based on proximity to expiry | Small chance of unnecessary recompute; no lock needed |
| **Stale-while-revalidate** | Return stale value immediately; recompute in background | Brief stale data acceptable; zero extra latency |
| **Jitter on TTL** | Randomize TTL (`base_ttl ± jitter`) to spread expiry | Prevents synchronized mass expiry |

**Recommended default**: Mutex for correctness-sensitive data; stale-while-revalidate for best UX.

---

## Eviction Policies

| Policy | Behavior | Best for |
|--------|----------|---------|
| **LRU** (Least Recently Used) | Evict item not accessed longest | General-purpose; temporal locality |
| **LFU** (Least Frequently Used) | Evict item accessed fewest times | Stable hot-set (celebrity profiles, trending pages) |
| **TTL** | Evict after fixed time regardless of access | Data freshness requirements (prices, inventory) |
| **FIFO** | Evict oldest inserted item | Simple; rarely best choice |
| **Random** | Evict random item | Surprisingly effective at scale; avoids LRU's worst cases |

**Redis default**: LRU (with approximate algorithm for performance). Can configure LFU in Redis 4+.

---

## Redis vs Memcached

| | Redis | Memcached |
|--|-------|-----------|
| Data structures | Strings, Lists, Sets, Sorted Sets, Hashes, Streams | Strings only |
| Persistence | RDB snapshots + AOF | None |
| Replication | Built-in leader-follower | External (e.g., mcrouter) |
| Clustering | Redis Cluster (built-in) | Client-side sharding only |
| Pub/Sub | Yes | No |
| Lua scripting | Yes | No |
| Memory efficiency | Slightly less than Memcached | Slightly better for pure key-value |
| Multi-threading | Single-threaded (I/O multiplexed) | Multi-threaded |

**Choose Redis when**: You need persistence, complex data structures, pub/sub, or built-in clustering.
**Choose Memcached when**: Pure string caching at very high throughput with many cores and memory efficiency matters most.

**In practice**: Redis is almost always the right default.

---

## CDN Caching

### Push vs Pull

| | Push CDN | Pull CDN |
|--|----------|---------|
| How | You upload content to CDN proactively | CDN fetches from origin on first miss |
| Best for | Known-static content (JS, CSS, videos) | Dynamic-ish content (user avatars, thumbnails) |
| Stale risk | Low (you control upload) | Higher (TTL-based expiry) |
| Complexity | Higher (must manage uploads) | Lower |

### Cache Invalidation
- **TTL-based**: Set `Cache-Control: max-age=N` — simple but lag before update reflects
- **Versioned URLs**: `/static/bundle.v3.js` — change URL = instant cache busting, no invalidation needed
- **Purge API**: CloudFront/Fastly APIs to invalidate by URL or tag — use for urgent content fixes

---

## Key Numbers

| | Latency |
|--|---------|
| Redis GET (same DC) | ~0.1 ms |
| Memcached GET (same DC) | ~0.05 ms |
| DB read (Postgres, indexed) | ~1–5 ms |
| DB read (Cassandra) | ~1–3 ms |
| CDN edge cache hit | ~5–20 ms |
| CDN miss (origin fetch) | ~50–200 ms |

---

## Interview Tips

- Always state **why** you're caching — what the bottleneck is
- Specify the **strategy** (cache-aside is most common; say it explicitly)
- Discuss **TTL** and **invalidation** — interviewers love seeing you think about staleness
- Mention **stampede** for any viral/high-traffic scenario
- For write-heavy systems, consider whether caching even helps (often it doesn't — look at write-through or message queues instead)
