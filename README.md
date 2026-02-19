# What if your AI assistants remembered everything you did on your computer?


## How does it work?

```
Periodic screenshots (smart activity based capture)
    → Vision model extracts information about your activity
        → Local storage with vector embeddings (SQLite)
            → MCP server
                → Claude can access information about your activity
```

Everything runs locally - besides the vision model where you can point to any endponit (including local private model).

---

## The hard parts of building this

1. Efficient use of tokens
2. Making the extracted data useful for the user

Other challenges which I won't focus on but happy to discuss in person
- Capturing activity costs CPU (but users like their all day battery life)
- Data privacy guarantees
- Cross platform distribution
- App integrations
- Closing the loop for coding agents (without verification, it produces slop usually, providing verification criteria is tougher for electron apps than for most SW)

---

## Part 1: Token efficiency

### The goal

Extract meaningful information about the users activity.

### The problem

We're processing hundreds of screenshots per hour through vision models - this can get expensive.

We want to provide as **much information** as possible in as **few input tokens** as possible.

Output tokens are quite expensive.

---

### Lessons with Input tokens:

| Model | Tokens / large image | Tokens / small image | Tokens / sec of video | Cost / 1M input tokens | Cost / 1h video input | Cost / 250h video input |
|---|---|---|---|---|---|---|
| Gemini 2.5 Flash Lite | 3,360 | 1,812 | ~258 | $0.10 | ~$0.09 | ~$23 |
| Gemini 2.5 Flash | 3,360 | 1,812 | ~258 | $0.30 | ~$0.28 | ~$70 |
| Gemini 3 Flash Preview | 1,072 | 1,110 | ~64 | $0.50 | ~$0.12 | ~$29 |
| Gemini 3 Pro Preview | 1,072 | 1,110 | ~64 | $2.00 | ~$0.46 | ~$115 |
| Mistral Small 3.2 24B | 2,025 | 1,633 | N/A | $0.10 | N/A | N/A |


Understand your models' tokenizers!

Video is an incredibly token-efficient representation of "activity" (LLMs usualy sample 1FPS and understand temporal dependencies incredibly well).

*Large image: 3024x1964, small image: 1698x894.*

---

### Lessons with Output tokens:

Usually cost 3–8x more than input tokens.

Our approach: don't ask the LLM to do what a free API can do. The LLM handles *understanding*. OCR handles *raw recall*.

**High Level:**
LLMs output activity summary - the narrative of what happened at ~$0.0001

**Detail:**
Native OCR extracts text which can be used for perfect recall of the LLM.


## Part 2: Making the data useful


### Providing different levels of detail

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

**Good tool docs** prevent the LLM from making wrong conclusions (for example based on the OCR data).

### Skills 

Context is not valuable if not used. 

Providing skills that help guide the LLM to use the tools correctly helps the user gain value from the app.

**Examples:**
- context summarization from recent activity
- finding topics of interest (use it to automate my news ingestion)
- analyzing how you work - looking for improvement opportunities
- time reports

### Feel free to contribute - try it out

<img src="memorylane-repo-qr.png" alt="QR Code" width="200">

**[github.com/deusXmachina-dev/memorylane](https://github.com/deusXmachina-dev/memorylane)**

Electron · TypeScript · MCP · SQLite

Works with Claude Desktop, Cursor, and Claude Code today.

---

# Let's connect

🐙 [github.com/FilipKubis](https://github.com/FilipKubis)
🔗 [linkedin.com/in/filip-kubis](https://linkedin.com/in/filip-kubis)

Please come chat if the context problem is interesting for you.
