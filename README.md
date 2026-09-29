# Filter Coffee Way

> *Ground fresh. Think deep. Build strong.*

Source for [filtercoffeeway.com](https://filtercoffeeway.com): a personal portfolio of the things I build, the essays I write, and a software engineering notebook of system designs worked end to end.

The site is plain static HTML and CSS with no build step, hosted on GitHub Pages.

---

## What's on the site

| Section | What's there |
|---------|--------------|
| **Projects** | [Todos](https://chromewebstore.google.com/detail/todos-highlight-text-to-n/dmegmkgoomfkgenipfodoeblfjnhpjok), a Chrome extension that turns the new tab into notes and to-dos (live on the Web Store). Rangi, a voice agent that places real phone calls and texts back the result (built and demoed, personal use only). |
| **Writing** | Essays on [blog.filtercoffeeway.com](https://blog.filtercoffeeway.com) and Medium |
| **Software Engineering** | System design deep dives and the reusable patterns they produced |
| **About Me** | [`about-me.html`](about-me.html), a short personal page |
| **Filter Coffee** | [`about.html`](about.html), what South Indian filter coffee is and why the brand is named after it |

---

## Structure

```
website/
├── index.html                 ← home: projects, writing, engineering, principles
├── about-me.html              ← personal page
├── about.html                 ← what is filter coffee?
├── _topic-roadmap.html        ← system design topics and concepts reference
├── designs/                   ← one deep dive per system
│   ├── url-shortener.html
│   └── pastebin.html
├── patterns/                  ← cross-cutting concepts, reused across designs
│   ├── caching.html
│   ├── token-generation.html
│   └── database-selection.html
├── templates/
│   └── design-template.html   ← scaffold for new designs
├── todos/privacy.html         ← privacy policy for the Todos Chrome extension
├── kanakkupillai/privacy.html ← privacy policy for the KanakkuPillai Google OAuth app
├── assets/                    ← logo, favicon
└── CNAME                      ← filtercoffeeway.com
```

---

## System designs

Each design follows the same thirteen sections: requirements, capacity estimates, high-level design, key decisions, deep dives, bottlenecks by tier, hot keys, multi-region, failure modes, security and abuse, observability and SLOs, takeaways, and follow-up questions.

| System | Key concepts |
|--------|--------------|
| [URL Shortener](designs/url-shortener.html) | Base62 IDs, layered caching, the Cassandra vs. Postgres call |
| [Paste Bin](designs/pastebin.html) | Inline vs. object storage, an expiry pipeline |

### Patterns

When the same idea shows up in several designs, it gets its own pattern page, and the designs link to it instead of repeating it.

| Pattern | What it covers |
|---------|----------------|
| [Caching](patterns/caching.html) | Cache-aside, write-through, stampede, eviction, CDN |
| [Token / ID Generation](patterns/token-generation.html) | Base62, Snowflake, UUID, ZooKeeper counter ranges |
| [Database Selection](patterns/database-selection.html) | Postgres vs. Cassandra vs. Redis, a decision framework |

---

## The seven principles

1. **Grind fresh**: work from first principles before borrowing a solution
2. **Drip slow**: deep work can't be hurried
3. **Find the ratio**: there's no universal right answer; calibrate to context
4. **Pour with intention**: how you serve something matters as much as what you serve
5. **Show up every morning**: consistency is the highest form of craft
6. **Name the instant coffee**: shortcuts are fine; self-deception is not
7. **Brew the coffee, not the cup**: ship the thing itself before polishing the container

---

## Running locally

No build step. Open `index.html` in a browser, or serve the folder:

```sh
python3 -m http.server 8000
```

Pushing to `main` deploys to GitHub Pages.

---

*Brewed in Madras. Served to the world.*
