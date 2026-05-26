# System Design

> *Ground fresh. Think deep. Build strong.*

A structured reference for understanding how large-scale systems are designed — not just what the answers are, but why the trade-offs exist.

Each design goes deep: requirements, capacity estimates, component decisions, failure modes, and alternate approaches. Patterns are extracted across designs so the same insight compounds over time.

---

## Structure

```
system-design/
├── index.html                 ← home page (browse at GitHub Pages)
├── _topic-roadmap.html        ← all topics and concepts reference
├── designs/                   ← one deep dive per system
│   └── url-shortener.html
├── patterns/                  ← cross-cutting concepts, reused across designs
│   ├── caching.html
│   ├── token-generation.html
│   └── database-selection.html
└── templates/
    └── design-template.html   ← scaffold for new designs
```

---

## Designs

| System | Key Concepts |
|--------|-------------|
| [URL Shortener](designs/url-shortener.html) | Read-heavy systems, caching, base62 encoding, token generation |

More coming — see the [topic roadmap](_topic-roadmap.html).

---

## Patterns

Patterns are the through-line. The same caching strategy appears in URL shorteners, feed systems, and CDNs. The same database trade-off surfaces in every design. Rather than repeat the reasoning, each pattern file goes deep once — and every design links back to it.

| Pattern | What it covers |
|---------|---------------|
| [Caching](patterns/caching.html) | Cache-aside, write-through, stampede, eviction, CDN |
| [Token / ID Generation](patterns/token-generation.html) | Base62, Snowflake, UUID, Zookeeper counter ranges |
| [Database Selection](patterns/database-selection.html) | Postgres vs Cassandra vs Redis — a decision framework |

---

## Philosophy

Good system design is not about memorizing architectures. It's about developing judgment — knowing *why* you'd pick Cassandra over Postgres, *when* a cache makes things worse, *what* breaks first under load.

The filter coffee way: **find the ratio**. There is no universal right answer. Every decision is a trade-off calibrated to context — read/write ratio, consistency requirements, scale, team capability. The goal is to understand the trade-offs well enough to reason about any system, not just the ones you've seen before.

---

## Live Site

Hosted on GitHub Pages — [view here](https://filtercoffeeway.github.io/system-design/)
