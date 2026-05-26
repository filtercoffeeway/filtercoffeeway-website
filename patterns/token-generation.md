# Pattern: Token / ID Generation

**Appears in**: URL Shortener, Order IDs, Session tokens, Distributed primary keys, Idempotency keys

---

## The Problem

Distributed systems need unique IDs generated across multiple nodes with no coordination overhead. The requirements usually include:
- **Uniqueness** — no collisions, ever
- **Speed** — millions per second without DB round-trips
- **Optionally sortable** — time-ordered IDs enable range scans
- **Short / URL-safe** — for human-facing tokens

---

## Approach 1: Database Auto-Increment

```sql
INSERT INTO urls (...) → returns BIGINT auto_increment id
```

- **Pros**: Simple, unique, sortable
- **Cons**: Single point of failure; write bottleneck; exposes record count
- **Use when**: Single-node, low write volume

---

## Approach 2: UUID v4 (Random)

```
550e8400-e29b-41d4-a716-446655440000
```

- **Pros**: Trivially unique; no coordination; well-supported
- **Cons**: 128 bits = long; not sortable; random = poor index locality (B-tree fragmentation)
- **Use when**: Session tokens, API keys where length doesn't matter

---

## Approach 3: UUID v7 (Time-ordered)

```
018e-xxxx-xxxx-xxxx-xxxxxxxxxxxx  (48-bit timestamp prefix)
```

- **Pros**: Unique + time-sortable; good for DB indexes; no coordination
- **Cons**: Still 128 bits; UUID v7 is a newer spec (check library support)
- **Use when**: DB primary keys where you want insert locality without custom infra

---

## Approach 4: Snowflake ID (Twitter)

```
[41-bit timestamp ms] [10-bit machine ID] [12-bit sequence]
= 64-bit integer
```

- **Pros**: Globally unique; sortable by time; compact (64-bit); ~4096 IDs/ms/node
- **Cons**: Requires machine ID assignment; clock skew can cause issues
- **Use when**: High-throughput distributed systems (tweets, orders, events)

### Variants
- **Sonyflake** (Sony): 39-bit timestamp (10ms resolution) + 16-bit sequence + 8-bit machine
- **Instagram's approach**: Postgres sequence per shard + timestamp + shard ID

---

## Approach 5: MD5 / SHA Hash of Content

```
short_code = base62(SHA256(long_url))[:7]
```

- **Pros**: Deterministic — same URL always gets same code (natural dedup)
- **Cons**: Collision risk for first N chars; can't guarantee uniqueness without check; not sequential
- **Use when**: Content-addressable storage, dedup is a primary requirement

### Collision handling
- On write: `INSERT ... ON CONFLICT` or check-then-insert
- On collision: append salt, re-hash, retry

---

## Approach 6: Counter + Base62 Encoding ✅ (Recommended for URL Shortener)

```
counter = 1000000
base62(1000000) = "4c92"  →  7-char short code
```

**Base62 alphabet**: `a-z A-Z 0-9` (62 chars)

| Chars | Max values | Coverage at 100M/day |
|-------|-----------|----------------------|
| 6 | 56 billion | ~1.5 years |
| 7 | 3.5 trillion | ~95 years |
| 8 | 218 trillion | ~5,800 years |

Use **7 characters**.

### Counter Sources

| Method | How | Trade-off |
|--------|-----|-----------|
| Single DB sequence | `SELECT nextval('url_seq')` | Simple; DB bottleneck |
| Redis INCR | Atomic increment | Fast; single Redis = SPOF without clustering |
| Zookeeper ranges | Each server claims range (e.g., 1M–2M) | No coordination per ID; Zookeeper is only needed for range claims |

**Best choice**: Zookeeper (or etcd) range pre-fetch. Servers generate IDs locally within their claimed range. Claim a new range only when exhausted.

---

## Approach 7: Pre-generated Token Pool

```
Background job generates N random tokens
Stores in "available" table
Write service pops one atomically
```

- **Pros**: Write service has zero generation logic; easy to scale
- **Cons**: Complex background job; token pool size to manage; "used" vs "available" state
- **Use when**: Tokens need to be guaranteed-random (not sequential) and pre-validation is useful

---

## Base62 Implementation Reference

```python
ALPHABET = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
BASE = 62

def encode(num: int) -> str:
    if num == 0:
        return ALPHABET[0]
    result = []
    while num:
        result.append(ALPHABET[num % BASE])
        num //= BASE
    return ''.join(reversed(result))

def decode(s: str) -> int:
    result = 0
    for char in s:
        result = result * BASE + ALPHABET.index(char)
    return result
```

---

## Decision Framework

| Requirement | Best approach |
|-------------|--------------|
| Human-readable + short | Base62 counter |
| Time-sortable + compact | Snowflake / UUID v7 |
| No coordination at all | UUID v4 |
| Deterministic from content | Hash (SHA/MD5) + truncate |
| Pre-validated randomness | Token pool |
| Simplest possible | DB auto-increment |

---

## Interview Tips

- For URL Shortener: always go Counter + Base62 — explain why hashing risks collisions
- Mention Zookeeper/etcd for range coordination — shows distributed systems fluency
- Know 62^7 ≈ 3.5 trillion and that 7 chars covers decades at scale
- Snowflake is worth knowing for any "generate unique IDs at scale" question
- Always discuss: will sequential IDs expose business metrics? (order count, user count) — sometimes security consideration to use random instead
