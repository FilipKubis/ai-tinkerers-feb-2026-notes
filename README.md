# What if your AI assistant remembered everything you did on your computer?

---

# Let me show you.

> *"What have I been working on in the last few hours?"*

🖥️ **LIVE DEMO — Claude Desktop + MemoryLane MCP**

---

# What just happened?

```
Periodic screenshots (1/sec)
    → Vision model extraction
        → Local vector storage (SQLite + LanceDB)
            → MCP server
                → Claude answers your question
```

**All local. All private. No cloud required.**

---

# Hard tradeoffs in building this

1. **Token efficiency** — maximize context, minimize cost
2. **MCP tool design** — make the tech actually useful

---

# PART 1: TOKEN EFFICIENCY

---

# The problem

Processing **hundreds of screenshots/hour** through vision models

Two cost levers:
- **Input tokens** — how you encode what happened on screen
- **Output tokens** — what you ask the model to produce

---

# Input tokens: video is the best encoding

| | 10 individual images | 10s video (1fps) |
|---|---|---|
| **Tokens** | ~2,580 | ~2,580 |
| **Payload** | ~500KB–1MB | ~100–300KB |
| **Temporal context** | ❌ none | ✅ model sees transitions |

Same token cost. Smaller payload. Richer understanding.

---

# Not all models are equal

| Model | Cost / 5 min video |
|---|---|
| Gemini 2.5 Flash | **~$0.001** |
| Gemini 3 Flash | **~$0.0008** (variable seq length) |
| GPT-4.1 | ~$0.46 |
| GPT-4.1 mini | ~$0.002 |

Gemini's native video tokenization = **400x cheaper** than GPT-4.1

---

# Output tokens: the expensive part

Output tokens cost **4–8x more** than input tokens

Our approach: **don't ask the LLM to do what a free API can do**

---

# Split the work

| Task | Who does it | Cost |
|---|---|---|
| **Activity summary** (40–60 words) | Vision LLM | ~$0.001 |
| **Raw text extraction** | On-device OCR | **Free** |

LLM → *understanding*
OCR → *raw recall*

---

# The architecture

```
Screenshot
    ├── → Vision LLM → 50-word summary → vector DB
    │                                     (semantic search)
    └── → Native OCR → raw text → SQLite
                                  (exact recall, on-demand)
```

Query time: summaries first (fast, cheap), OCR on-demand.

---

# Takeaway #1

> **Optimize where the cost actually is.**
>
> Video for cheap dense input.
> Short summaries for expensive output.
> Free local OCR for everything else.

---

# PART 2: MCP TOOL DESIGN

---

# How do you make screen context useful to an LLM?

---

# Three tools, not one

```typescript
search_context(query, startTime?, endTime?, limit?)
// → semantic search over activity summaries
// "when did I review that PR?"

browse_timeline(startTime, endTime, limit?, sampling?)
// → chronological listing, uniform sampling
// "what did I do this morning?"

get_activity_details(ids[])
// → full OCR text, on-demand
// "show me the exact error message"
```

---

# Why three tools?

LLMs are **bad at** constructing complex queries

LLMs are **great at** choosing the right tool for the intent

```
"find my work on auth module"  →  search_context
"what did I do today?"         →  browse_timeline
"what exactly did it say?"     →  get_activity_details
```

**Design tools around how LLMs think, not how your DB works.**

---

# Live demo

> *"When did I last work on the notarization pipeline?"*

🖥️ **Watch the MCP tool calls in Claude Desktop**

---

# Lessons for MCP builders

**Tool descriptions are prompts** — they matter as much as implementation

**Summaries first, details on-demand** — keep responses fast and cheap

**Add guardrails** — we use anti-overreach rules to prevent over-fetching

**Happy path = one tool call** — not three

---

# Takeaway #2

> **MCP tool design is prompt engineering.**
>
> Name your tools clearly.
> Split by intent, not by data model.
> Make the happy path one call.

---

# Open source — scan to star ⭐

![QR Code](memorylane-repo-qr.png)

**[github.com/deusXmachina-dev/memorylane](https://github.com/deusXmachina-dev/memorylane)**

Electron · TypeScript · MCP · SQLite · LanceDB

Works with Claude Desktop, Cursor, Claude Code — today.

---

# Let's connect

🐙 **[github.com/FilipKubis](https://github.com/FilipKubis)**
🔗 **[linkedin.com/in/filip-kubis](https://linkedin.com/in/filip-kubis)**

Come chat if you're building MCP tools or local-first AI.
