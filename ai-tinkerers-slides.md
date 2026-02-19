# What if your AI assistant remembered everything you did on your computer?

---

# Let me show you.

> *"What have I been working on in the last few hours?"*

Live demo — Claude Desktop + MemoryLane MCP

---

# What just happened?

```
Periodic screenshots (1/sec)
    → Vision model extraction
        → Local vector storage (SQLite + LanceDB)
            → MCP server
                → Claude answers your question
```

Everything runs locally. Nothing leaves your machine.

---

# The hard parts of building this

1. Token efficiency — getting useful context without burning money
2. MCP tool design — making the tools actually pleasant for an LLM to use

---

# Part 1: Token efficiency

---

# The problem

We're processing hundreds of screenshots per hour through vision models.

Two things drive cost:
- **Input tokens** — how you encode what happened on screen
- **Output tokens** — what you ask the model to produce

---

# Input tokens: video turns out to be the best encoding

| | 10 individual images | 10s video (1fps) |
|---|---|---|
| Tokens | ~2,580 | ~2,580 |
| Payload | ~500KB–1MB | ~100–300KB |
| Temporal context | none | model sees transitions |

Same token cost. Smaller payload. And the model actually understands what's happening over time.

---

# Not all models are equal

| Model | Cost / 5 min video |
|---|---|
| Gemini 2.5 Flash | ~$0.001 |
| Gemini 3 Flash | ~$0.0008 (variable seq length) |
| GPT-4.1 | ~$0.46 |
| GPT-4.1 mini | ~$0.002 |

Gemini's native video tokenization makes it about 400x cheaper than GPT-4.1 for this use case.

---

# Output tokens: this is where it gets expensive

Output tokens cost 4–8x more than input tokens.

Our approach: don't ask the LLM to do what a free API can do.

---

# Split the work

| Task | Who does it | Cost |
|---|---|---|
| Activity summary (40–60 words) | Vision LLM | ~$0.001 |
| Raw text extraction | On-device OCR | Free |

The LLM handles *understanding*. OCR handles *raw recall*.

---

# The architecture

```
Screenshot
    ├── → Vision LLM → 50-word summary → vector DB
    │                                     (semantic search)
    └── → Native OCR → raw text → SQLite
                                  (exact recall, on-demand)
```

At query time, we search summaries first (fast, cheap), then pull OCR text on demand.

---

# The point

Optimize where the cost actually is.

Video gives you cheap, dense input. Short summaries keep output costs down. Free local OCR covers the rest.

---

# Part 2: MCP tool design

---

# How do you make screen context useful to an LLM?

---

# Three tools, not one

```typescript
search_context(query, startTime?, endTime?, limit?)
// semantic search over activity summaries
// "when did I review that PR?"

browse_timeline(startTime, endTime, limit?, sampling?)
// chronological listing, uniform sampling
// "what did I do this morning?"

get_activity_details(ids[])
// full OCR text, on-demand
// "show me the exact error message"
```

---

# Why three?

LLMs are bad at constructing complex queries.

But they're good at picking the right tool for what you're asking.

```
"find my work on auth module"  →  search_context
"what did I do today?"         →  browse_timeline
"what exactly did it say?"     →  get_activity_details
```

Design tools around how LLMs think, not how your database works.

---

# Live demo

> *"When did I last work on the notarization pipeline?"*

Watch the MCP tool calls in Claude Desktop.

---

# Things we learned building MCP tools

**Tool descriptions are basically prompts** — they matter as much as the implementation behind them.

**Summaries first, details on demand** — keeps responses fast and cheap.

**Add guardrails** — we use anti-overreach rules to prevent over-fetching.

**The happy path should be one tool call** — not three.

---

# The point

MCP tool design is really prompt engineering.

Name your tools clearly. Split by intent, not by data model. Make the common case a single call.

---

# It's open source — scan to star

![QR Code](memorylane-repo-qr.png)

**[github.com/deusXmachina-dev/memorylane](https://github.com/deusXmachina-dev/memorylane)**

Electron · TypeScript · MCP · SQLite · LanceDB

Works with Claude Desktop, Cursor, and Claude Code today.

---

# Let's connect

🐙 [github.com/FilipKubis](https://github.com/FilipKubis)
🔗 [linkedin.com/in/filip-kubis](https://linkedin.com/in/filip-kubis)

Come chat if you're building MCP tools or local-first AI.
