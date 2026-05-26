# System Design: [TOPIC NAME]

**Date**: YYYY-MM-DD
**Difficulty**: Easy / Medium / Hard
**Tags**: [e.g., read-heavy, caching, streaming, pub-sub, geolocation]

---

## 1. Requirements

### Functional
- 
- 
- 

### Non-Functional
- **Availability**: 
- **Latency**: 
- **Consistency**: 
- **Durability**: 
- Read/Write ratio: ~X:1

### Out of Scope
- 
- 

---

## 2. Capacity Estimates

| Parameter | Value |
|-----------|-------|
| Daily active users | |
| Requests per day | |
| Write QPS | |
| Read QPS | |
| Data per record | |
| Storage (5 years) | |
| Bandwidth | |

**Key constraint to design around**: [read-heavy / write-heavy / storage / latency]

---

## 3. High-Level Design

```
[Draw ASCII diagram here]

Client → Load Balancer → Service → Cache → DB
```

### Write path
1. 
2. 
3. 

### Read path
1. 
2. 
3. 

---

## 4. Key Design Decisions

### 4.1 [Decision Name]

**Options considered**:
- Option A: pros / cons
- Option B: pros / cons

**Decision**: [chosen approach and why]

### 4.2 [Decision Name]

**Options considered**:
- Option A: pros / cons
- Option B: pros / cons

**Decision**: [chosen approach and why]

### 4.3 Database Choice

See `patterns/database-selection.md`.

- **Schema**: 
- **Access pattern**: 
- **Decision**: 
- **Sharding key** (if applicable): 

### 4.4 Caching

See `patterns/caching.md`.

- **Strategy**: 
- **TTL**: 
- **Eviction**: 
- **Stampede mitigation**: 

---

## 5. Deep Dives

### [Topic 1 — e.g., Handling hot spots]


### [Topic 2 — e.g., Consistency trade-offs]


### [Topic 3 — e.g., Failure scenarios]


---

## 6. Failure Modes & Mitigations

| Failure | Impact | Mitigation |
|---------|--------|-----------|
| | | |
| | | |
| | | |

---

## 7. Key Takeaways

1. 
2. 
3. 

---

## 8. Patterns to Update

After this session, add/update:
- [ ] `patterns/caching.md` — [what to add]
- [ ] `patterns/database-selection.md` — [what to add]
- [ ] `patterns/[new-pattern].md` — [if a new cross-cutting pattern emerged]

---

## 9. Follow-Up Questions (for self-practice)

- 
- 
- 
