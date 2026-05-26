# System Design Topic Roadmap

## Progress Tracker

### ✅ Completed
- [x] URL Shortener — `designs/url-shortener.md`

### 🔲 In Queue
- [ ] Paste Bin / text sharing
- [ ] Instagram / photo sharing feed
- [ ] Twitter / social media timeline
- [ ] YouTube / video streaming
- [ ] WhatsApp / chat system
- [ ] Uber / ride-sharing with geolocation
- [ ] Rate limiter
- [ ] Web crawler
- [ ] Notification system
- [ ] Search autocomplete / typeahead
- [ ] Distributed key-value store
- [ ] Distributed cache (Redis-style)
- [ ] Consistent hashing deep dive
- [ ] Distributed message queue (Kafka-style)
- [ ] Payment system

---

## Concepts Mastered Checklist

Use this for self-assessment before an interview.

### Fundamentals
- [ ] CAP theorem — what each letter means and real-world examples of each trade-off
- [ ] BASE vs ACID — when to choose which
- [ ] Horizontal vs vertical scaling
- [ ] Stateless vs stateful services
- [ ] Synchronous vs asynchronous communication

### Data Storage
- [ ] SQL vs NoSQL — decision framework, not just "SQL = relational"
- [ ] Sharding strategies — range, hash, directory-based
- [ ] Replication — leader-follower, multi-leader, leaderless
- [ ] Indexes — B-tree, LSM-tree, when each is better
- [ ] Consistent hashing — how it works, virtual nodes, use cases

### Caching
- [ ] Cache-aside, write-through, write-behind, read-through
- [ ] Cache stampede / thundering herd and solutions
- [ ] Eviction policies — LRU, LFU, TTL
- [ ] Redis vs Memcached — when to pick which
- [ ] CDN — push vs pull, cache invalidation

### Networking & APIs
- [ ] REST vs GraphQL vs gRPC — trade-offs
- [ ] Long polling vs WebSockets vs Server-Sent Events
- [ ] Load balancing — round-robin, least connections, consistent hashing
- [ ] Rate limiting — token bucket, leaky bucket, sliding window

### Distributed Systems
- [ ] Leader election
- [ ] Distributed transactions — 2PC, saga pattern
- [ ] Message queues — Kafka, RabbitMQ, trade-offs
- [ ] Idempotency — why it matters, how to implement
- [ ] Exactly-once vs at-least-once delivery
- [ ] Bloom filters — use cases, false positive rate
- [ ] Distributed locks

### Estimation
- [ ] Back-of-envelope math — QPS, storage, bandwidth
- [ ] Read/write ratio reasoning
- [ ] When to reach for what tier (cache vs DB vs CDN)

---

## Numbers to Know Cold

| Metric | Value | Notes |
|--------|-------|-------|
| L1 cache hit | ~1 ns | |
| L2 cache hit | ~10 ns | |
| RAM access | ~100 ns | 100× slower than L1 |
| SSD random read | ~100 µs | 1000× slower than RAM |
| HDD seek | ~10 ms | 100× slower than SSD |
| Network (same DC) | ~0.5 ms | |
| Network (cross-region) | ~150 ms | |
| 1 Gbps NIC | ~125 MB/s | |
| MySQL/Postgres write | ~1–5 ms | single row, no contention |
| Redis get | ~0.1 ms | in-memory |
| Typical DB row | ~1 KB | |
| Typical image (compressed) | ~300 KB | |
| Typical video (1 min, 720p) | ~50 MB | |
| 1 million seconds | ~11.5 days | handy for TTL math |
| 1 billion seconds | ~31.7 years | handy for ID space sizing |

---

## Pattern Cross-Reference

| Pattern | Appears in |
|---------|-----------|
| Base62 encoding | URL Shortener, Order IDs, Session tokens |
| Zookeeper coordination | URL Shortener (counter), Leader election |
| Cassandra (wide-column) | URL Shortener, Chat, Timeline |
| Write-through cache | URL Shortener, Feed, any read-heavy system |
| Cache stampede mitigation | URL Shortener, Feed, any hot-key scenario |
| Consistent hashing | Cache clusters, Sharding, Load balancing |
