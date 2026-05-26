# System Design: URL Shortener (TinyURL / bit.ly)

**Date**: 2026-05-25
**Difficulty**: Medium
**Tags**: read-heavy, caching, token generation, NoSQL, hashing

---

## 1. Requirements

### Functional
- Given a long URL, generate a short URL (alias)
- Given a short URL, redirect to the original long URL
- Short URLs should be unique and not collide
- Optional: custom aliases, expiration dates, analytics

### Non-Functional
- **High availability** — redirects must always work
- **Low latency** — redirect < 10 ms p99
- **Durability** — short URLs should not disappear
- Read-heavy: ~100:1 read/write ratio

### Out of Scope (for this session)
- User authentication
- Full analytics dashboard
- Link preview / safety scanning

---

## 2. Capacity Estimates

| Parameter | Value |
|-----------|-------|
| New URLs per day | 100 million |
| New URLs per second (write QPS) | ~1,200 |
| Redirect QPS (100× reads) | ~120,000 |
| URL record size | ~500 bytes (long URL + metadata) |
| Storage for 10 years | 100M × 365 × 10 × 500B ≈ **180 TB** |
| Cache (top 20% of URLs = 80% traffic) | 120K QPS × 86400s × 20% × 500B ≈ **~1 TB/day** → hot set fits in Redis |

**Key insight**: This is overwhelmingly read-heavy. Design around the read path.

---

## 3. High-Level Design

```
Client
  │
  ▼
[Load Balancer]
  │              │
  ▼              ▼
[Write Service]  [Redirect Service]
  │                    │
  ▼                    ▼
[Token Generator]   [Redis Cache]  ──miss──▶  [DB Read Replica]
  │                                                │
  ▼                                                │
[DB Primary] ◀───────────────────────────────────┘
```

### Write path
1. Client POSTs long URL
2. Write service calls token generator → gets a unique short code
3. Writes `{short_code → long_url, created_at, expiry}` to DB primary
4. Optionally pre-warms cache
5. Returns short URL to client

### Read path (redirect)
1. Client GETs `tinyurl.com/abc123`
2. Redirect service checks Redis cache first
3. Cache hit → 301/302 redirect immediately
4. Cache miss → query DB read replica → cache the result → redirect

---

## 4. Key Design Decisions

### 4.1 Short Code Length

- Alphabet: base62 = `[a-z A-Z 0-9]` = 62 characters
- 7-character code: 62^7 ≈ **3.5 trillion** unique codes
- At 100M/day: covers **~95 years** before exhaustion
- 6 characters = 56 billion → only ~1.5 years. Use 7.

### 4.2 Token Generation Strategy

See `patterns/token-generation.md` for all approaches. For URL shortener:

**Chosen approach: Zookeeper-coordinated counter ranges**

- Each write server claims a range of counter values from Zookeeper (e.g., server A gets 1–1M, server B gets 1M–2M)
- Convert counter value to base62 → short code
- Servers generate tokens locally within their range — zero DB coordination overhead
- When range exhausted, claim new range from Zookeeper

**Why not MD5/SHA hash?**
- Hash of URL → take first 7 chars → collision risk + not human-stable
- Same URL hashed twice = same code (good for dedup but complicates custom aliases)

**Why not UUID?**
- 128 bits → need to shorten anyway; introduces randomness we don't need

### 4.3 301 vs 302 Redirect

| | 301 Permanent | 302 Temporary |
|--|---|---|
| Browser caches? | Yes | No |
| Analytics possible? | ❌ (browser skips server) | ✅ (every redirect hits server) |
| Server load | Lower | Higher |

**Decision**: 302 if you need click analytics; 301 if you want to minimize server load.

### 4.4 Database Choice

See `patterns/database-selection.md` for full framework.

**Schema**:
```
short_code  VARCHAR(7)   PRIMARY KEY
long_url    TEXT
user_id     BIGINT
created_at  TIMESTAMP
expires_at  TIMESTAMP    NULLABLE
```

**Postgres** works for < ~500M rows with proper indexing.

**Cassandra** for scale:
- Partition key = `short_code` → O(1) lookup
- No joins needed, pure key-value access pattern
- Multi-region replication built-in
- Eventual consistency acceptable here (stale redirect → still valid URL)

**Decision**: Start with Postgres. Migrate to Cassandra if write QPS or storage forces it.

### 4.5 Caching

See `patterns/caching.md` for full strategies.

- **Strategy**: Cache-aside on read path
- **TTL**: Match URL expiry, or 24h default for non-expiring URLs
- **Eviction**: LRU — naturally keeps hot links warm
- **Cache stampede**: Mutex lock on cache miss for the same key; or probabilistic early recompute
- **Cache key**: `short_code` → `long_url`
- **Cache size**: Top 20% of daily active short codes covers ~80% of traffic

---

## 5. Deep Dives

### Custom Aliases
- User specifies `tinyurl.com/my-brand`
- Write service checks availability first (`SELECT` on short_code)
- Reserve these in same table; mark `is_custom = true`
- Risk: squatting → rate-limit custom alias creation

### URL Expiration
- Store `expires_at` in DB
- Redirect service checks expiry before returning URL
- Background job (cron) deletes/tombstones expired records
- Cache TTL should be `min(24h, expires_at - now)`

### Analytics (if in scope)
- Write click events asynchronously to Kafka
- Downstream consumer aggregates into ClickHouse or similar OLAP store
- Never block the redirect on analytics writes

### Rate Limiting
- Limit writes per IP/user to prevent abuse
- Token bucket at API gateway level
- See rate limiter design (future topic)

---

## 6. Failure Modes & Mitigations

| Failure | Impact | Mitigation |
|---------|--------|-----------|
| DB primary down | Writes fail | Leader election / failover (Postgres streaming replication) |
| Cache down | All reads hit DB | DB read replicas; cache is optimization not dependency |
| Token range server (Zookeeper) down | New URL creation fails | Pre-fetched ranges allow ~1M URLs before blocking |
| Hash collision (if hash-based) | Wrong redirect | Check-on-write; retry with different hash seed |

---

## 7. Key Takeaways

1. **Read-heavy → cache everything on the read path.** Redis in front of DB is the single biggest win.
2. **Base62 + counter is cleaner than hashing** for uniqueness guarantees at scale.
3. **Zookeeper range pre-fetch** lets write servers operate independently — no lock contention.
4. **301 vs 302** is a real trade-off interviewers notice. Know when you'd choose each.
5. **Cassandra's partition-key model** is a perfect fit for pure key-value access (short_code → long_url).
6. **Separate write and redirect services** — they have completely different scaling profiles.

---

## 8. Follow-Up Questions (for self-practice)

- How would you handle a viral URL that suddenly gets 10M hits/min?
- How would you implement URL preview (unfurl og:image) without slowing down the write path?
- How would you shard the DB if short_code is the partition key but you also need queries by user_id?
- Design the analytics system — how do you count unique visitors?
