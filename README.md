# System Design Prep Vault

A structured knowledge base for mastering system design interviews.

## Vault Structure

```
system-design-prep/
├── README.md                  ← you are here
├── _claude-context.md         ← paste this at the start of every session
├── _topic-roadmap.md          ← progress tracker + concepts checklist
├── designs/                   ← one file per system you've designed
│   └── url-shortener.md
├── patterns/                  ← cross-cutting concepts referenced across designs
│   ├── caching.md
│   ├── token-generation.md
│   └── database-selection.md
└── templates/
    └── design-template.md     ← blank scaffold for new topics
```

## How to Use This Vault

### Starting a session
Paste the full contents of `_claude-context.md` as your first message. Claude will load your progress, learning style, preferred depth, and where you left off — no re-explaining needed.

### After each design session
1. Copy key notes into a new file in `designs/` using `templates/design-template.md` as the scaffold.
2. Open `_claude-context.md` and move the topic from **To Cover** → **Covered**.
3. Open `_topic-roadmap.md` and check off the topic in the checklist.
4. If the session surfaced a cross-cutting insight (e.g., consistent hashing, write-through caching), add or extend the relevant file in `patterns/`.

### Cross-cutting patterns
The `patterns/` folder accumulates knowledge that spans multiple designs. When the same concept shows up in a new context (e.g., consistent hashing appears in caching AND rate limiting AND sharding), enrich the pattern file rather than duplicating notes.

### Self-assessment
`_topic-roadmap.md` contains:
- A **Concepts Mastered** checklist — work through it before an interview
- A **Numbers to Know Cold** table — latency figures, throughput benchmarks, typical sizes
