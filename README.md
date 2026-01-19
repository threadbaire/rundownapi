# RundownAPI

An endpoint that teaches AI how to use your API. No SDK. It just reads and connects.

## What this is

A pattern for making your API self-describing to AI agents. You add one endpoint — `/api/rundown` — that returns everything an AI needs to integrate: what the API does, how to authenticate, what endpoints exist, and when to use them.

The AI reads that endpoint, understands the API, and starts working with it. No pre-built connectors, no MCP servers, no SDK to install. One URL, and the AI figures out the rest.

I built this for Threadbaire Server, tested it with Claude Code. Gave it the URL, gave it a token, and it built its own integration and started querying project history without any other setup.

This document describes the pattern so you can add it to your own APIs.

## Why bother

Right now, if you want an AI agent to use your API, you have a few options:

**MCP servers** — Anthropic's protocol, now adopted by others. Works great, but you need to run a server. Infrastructure to maintain.

**Agent frameworks** — LangChain, Haystack, Google ADK. They can auto-generate tools from OpenAPI specs. But you need Python or Node wrapping your spec, and the agent needs to be built for that framework.

**Custom integration** — You write the connector yourself, or the AI writes it for you every time. Works, but repetitive.

**OpenAPI specs** — Good for documenting structure. But they tell you *how* to call an endpoint, not *when* or *why*. The AI knows the shape of the API but not when to use it.

RundownAPI adds the missing piece: behavioral instructions. Not just "here are the endpoints" but "here's when you should query this, here's what to offer the user, here's how to behave."

The AI reads one URL and knows what to do. No setup on the AI side, no infrastructure on your side beyond the endpoint itself.

## The spec

A RundownAPI endpoint returns JSON with these fields:

### Required

| Field | What it is |
|-------|-----------|
| `rundown_version` | Version of the spec (currently `"0.1"`) |
| `base_url` | Where the API lives |
| `auth` | How to authenticate (method, parameter names, header format) |
| `purpose` | What this API does, in plain language. One or two sentences. |
| `endpoints` | Array of available endpoints — path, method, description, parameters |
| `ai_instructions` | When to use this API, how to behave, what to offer the user |

### Recommended

| Field | What it is |
|-------|-----------|
| `examples` | Curl or fetch snippets the AI can use immediately |
| `error_format` | What errors look like (so the AI can handle them) |
| `capabilities` | Quick summary of what's possible: `["read", "write", "search"]` |

### Optional

Everything else — rate limits, response schemas, MCP hints, whatever's useful for your case.

---

### Full example (from Threadbaire Server)

This is the actual endpoint that worked with Claude Code:

```json
{
  "rundown_version": "0.1",
  "base_url": "https://your-server.vercel.app",
  "purpose": "Project decision log and context memory. Store and retrieve dated entries about what you've done, decided, and learned across projects.",
  
  "auth": {
    "method": "token",
    "parameter": "token",
    "header_alternative": "Authorization: Bearer <token>",
    "note": "Most AI chat interfaces can't set headers. Use ?token= in the URL."
  },
  
  "capabilities": ["read", "write", "search"],
  
  "endpoints": [
    {
      "path": "/api/entries",
      "method": "GET",
      "description": "List entries with optional filters",
      "parameters": {
        "project": "Filter by project name",
        "document_type": "Filter by type: 'addendum' or 'dev_log'",
        "after": "Entries on or after this date (YYYY-MM-DD)",
        "before": "Entries on or before this date (YYYY-MM-DD)",
        "q": "Keyword search across title, summary, details",
        "limit": "Max entries to return (default 20)",
        "offset": "Skip this many entries (for pagination)"
      }
    },
    {
      "path": "/api/entries",
      "method": "POST",
      "description": "Create a new entry",
      "body": {
        "project": "(required) Project name",
        "document_type": "(required) 'addendum' or 'dev_log'",
        "entry_date": "(required) Date in YYYY-MM-DD format",
        "title": "(required) Short title for the entry",
        "entry_type": "Category like 'Feature', 'Decision', 'Fix'",
        "status": "Status like 'complete', 'in_progress', 'blocked'",
        "summary": "Brief description",
        "details": "Full details (markdown supported)",
        "next_steps": "What follows from this"
      }
    },
    {
      "path": "/api/entries/:id",
      "method": "GET",
      "description": "Get a single entry by ID"
    },
    {
      "path": "/api/entries/:id",
      "method": "PUT",
      "description": "Update an existing entry"
    },
    {
      "path": "/api/entries/:id",
      "method": "DELETE",
      "description": "Soft delete an entry"
    }
  ],
  
  "ai_instructions": {
    "triggers": [
      "User asks about project history or past decisions",
      "User asks what was done on a specific date",
      "User wants to know why something was decided",
      "User completes work and might want to log it"
    ],
    "behaviors": [
      "Query entries when user asks about history — don't guess from memory",
      "Offer to create entries when user completes significant work",
      "Summarize results in natural language, don't dump raw JSON",
      "Use keyword search (q parameter) for finding specific topics"
    ],
    "constraints": [
      "Ask user for API token if not provided",
      "Never fabricate entries or dates",
      "Don't modify or delete entries without explicit user request"
    ]
  },
  
  "examples": [
    {
      "description": "Get recent entries for a project",
      "request": "curl 'https://your-server.vercel.app/api/entries?project=myproject&limit=10&token=YOUR_TOKEN'"
    },
    {
      "description": "Search for entries about authentication",
      "request": "curl 'https://your-server.vercel.app/api/entries?q=authentication&token=YOUR_TOKEN'"
    },
    {
      "description": "Create a new entry",
      "request": "curl -X POST 'https://your-server.vercel.app/api/entries?token=YOUR_TOKEN' -H 'Content-Type: application/json' -d '{\"project\": \"myproject\", \"document_type\": \"addendum\", \"entry_date\": \"2025-01-19\", \"title\": \"Added user login\", \"status\": \"complete\", \"summary\": \"Implemented OAuth flow\"}'"
    }
  ],
  
  "error_format": {
    "structure": "{ \"error\": \"message\" }",
    "codes": {
      "401": "Missing or invalid token",
      "400": "Missing required fields",
      "404": "Entry not found"
    }
  }
}
```

---

### Minimal example (read-only API)

Not every API needs all of this. Here's a simpler version for a read-only service:

```json
{
  "rundown_version": "0.1",
  "base_url": "https://api.example.com",
  "purpose": "Weather data for any city. Returns current conditions and 5-day forecast.",
  
  "auth": {
    "method": "api_key",
    "header": "X-API-Key"
  },
  
  "capabilities": ["read"],
  
  "endpoints": [
    {
      "path": "/weather/:city",
      "method": "GET",
      "description": "Get current weather and forecast",
      "parameters": {
        "units": "'metric' or 'imperial' (default: metric)"
      }
    }
  ],
  
  "ai_instructions": {
    "triggers": [
      "User asks about weather",
      "User is planning travel or outdoor activities"
    ],
    "behaviors": [
      "Include both current conditions and relevant forecast days",
      "Mention precipitation probability if above 30%"
    ]
  },
  
  "examples": [
    {
      "description": "Get weather for London",
      "request": "curl -H 'X-API-Key: KEY' 'https://api.example.com/weather/london'"
    }
  ]
}
```

---

### The `ai_instructions` field

This is what makes RundownAPI different from OpenAPI. It tells the AI *when* to act, not just *how*.

**Triggers** — What should prompt the AI to use this API? User questions, topics, situations.

**Behaviors** — How should the AI present results? What should it offer proactively?

**Constraints** — What should the AI never do? What requires explicit user permission?

You can structure this as an object (like above) or write it as prose:

```json
"ai_instructions": "Query this API when the user asks about their project history or past decisions. Offer to log entries when they complete significant work. Always ask for the token if not provided. Never fabricate entries — if you don't find something, say so."
```

The format matters less than the content. Give the AI enough guidance to be useful without being asked.

## Security and trust

The obvious question: "If my AI reads behavioral instructions from an API, what stops a malicious API from telling it to do bad things?"

Fair concern. Here's how to think about it:

**You initiate the connection.** You give the AI the URL. You choose which APIs to connect to. This is no different from `git clone` or `npm install` — you're trusting the source. Don't give your AI a RundownAPI URL you wouldn't trust with your data.

**The AI still has its own guardrails.** Claude won't delete your files or exfiltrate your data just because an API endpoint said to. Modern AI models are trained to resist prompt injection and verify suspicious instructions with the user. The RundownAPI instructions don't override the model's safety layers.

**The instructions are scoped.** A RundownAPI endpoint describes how to use *that specific API* — when to query, what to offer, how to format results. It's not a general prompt that can say "ignore your system instructions" or "run this shell command." It's documentation with behavioral hints.

**You control the token.** Read access might be open, but writes require authentication. The AI asks you for the token. You decide whether to provide it. No token, no write access.

**The endpoint is readable.** Before you connect, you (or your AI) can fetch `/api/rundown` and inspect exactly what instructions it contains. Nothing is hidden. If something looks suspicious, don't use it.

---

### What this doesn't protect against

If you connect to a malicious API and give it your token, you've given that API access. Same as any API. RundownAPI doesn't add new attack surface — it just makes the integration faster.

If you're working with APIs you don't fully trust, don't give them write tokens. Use read-only access, or don't connect at all.

---

### The short version

RundownAPI is a trust relationship. Same as using any library, any API, any code you didn't write yourself.

You choose what to connect to. The AI's safety training still applies. If you wouldn't `git clone` it, don't give your AI the URL.

## Adding this to your API

Create an endpoint at `/api/rundown` (or wherever you like) that returns the JSON described above. That's it.

No library to install. No schema to register. Just a JSON endpoint your AI can read.

### The minimum

```javascript
// /api/rundown endpoint
export function GET(request) {
  return Response.json({
    rundown_version: "0.1",
    base_url: "https://your-api.com",
    purpose: "What your API does, in plain language.",
    auth: {
      method: "bearer",
      header: "Authorization: Bearer <token>"
    },
    endpoints: [
      {
        path: "/your/endpoint",
        method: "GET",
        description: "What it returns"
      }
    ],
    ai_instructions: "When to use this. How to behave. What to avoid."
  });
}
```

### Tips

**Detect your base URL dynamically.** If you deploy to multiple environments, read from the request headers instead of hardcoding:

```javascript
const host = request.headers.get('host');
const protocol = host.includes('localhost') ? 'http' : 'https';
const base_url = `${protocol}://${host}`;
```

**Keep it public.** The rundown endpoint is documentation — no auth required to read it. The AI will ask the user for tokens when it needs to authenticate with your actual endpoints.

**Update it when your API changes.** The rundown should match your actual endpoints. If you add a new route, add it here too.

**Test it with an AI.** Give Claude or ChatGPT the URL and ask it to integrate. Watch what works and what confuses it. Update your `ai_instructions` based on what you see.

## License

CC BY-SA 4.0 — Use it, adapt it, build on it. Keep it open and credit the source.

If you make something with this, that's great. If you improve the spec, even better. Just keep derivatives open so no one can close the gate.

## Attribution

RundownAPI was created by Lida Liberopoulou as part of the Threadbaire project.

- Threadbaire method: [github.com/threadbaire/method](https://github.com/threadbaire/method)
- Threadbaire server: [github.com/threadbaire/server](https://github.com/threadbaire/server)

Questions, suggestions, improvements: [GitHub issues](https://github.com/threadbaire/rundownapi/issues) or find me on [LinkedIn](https://linkedin.com/in/lliberopoulou).

## Questions

**Do I need Threadbaire to use this?**

No. RundownAPI is a standalone pattern. It came out of building Threadbaire, but works with any API.

**Is this a standard?**

It's v0.1. A pattern that worked for me, documented so others can try it. If it's useful, it'll spread. If people improve it, great.

**What AI models does this work with?**

Tested with Claude Code. Should work with anything that can fetch a URL and read JSON — ChatGPT, Gemini, Cursor, whatever. Some models have quirks (ChatGPT can't modify URL parameters, Gemini gets blocked by some bot protection). Your mileage may vary.

**What about MCP?**

MCP is great if you want to run servers and build that infrastructure. RundownAPI is for when you don't. They're not competing — use whichever fits your situation. Use both if you want.

**Can I use a different endpoint path?**

Yes. `/api/rundown` is a convention, not a requirement. Put it wherever makes sense. Just make sure your AI knows where to find it.

**What if I want to contribute to the spec?**

Open an issue. I'm not running a standards body, but if you've found something that works better, I'm interested.
