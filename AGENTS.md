AI agents: start at AGENTS.md

# RundownAPI

An endpoint that teaches AI how to use your API.  
No SDK. It just reads and connects.

---

## What this is

A pattern for making your API self-describing to AI agents.

You add one endpoint — /api/rundown — that returns everything an AI needs to integrate:
what the API does, how to authenticate, what endpoints exist, and when to use them.

The AI reads that endpoint, understands the API, and starts working with it.
No pre-built connectors, no MCP servers, no SDK to install.
One URL, and the AI figures out the rest.

This pattern was built and tested while developing Threadbaire Server.
Claude Code was given only the rundown URL and a token, and it built its own
integration and queried project history without any other setup.

This document describes the pattern so you can add it to your own APIs.

---

## Why bother

Right now, if you want an AI agent to use your API, you have a few options:

MCP servers — Anthropic’s protocol, now adopted by others. Works well, but you need
to run and maintain a server.

Agent frameworks — LangChain, Haystack, Google ADK. They can generate tools from
OpenAPI specs, but require Python or Node wrappers and framework-specific setup.

Custom integration — You write the connector yourself, or the AI re-derives it
every time. It works, but it’s repetitive.

OpenAPI specs — Good for documenting structure. They describe how to call an API,
but not when or why. The AI knows the shape of the API, not when to use it.

RundownAPI adds the missing piece: behavioral instructions.

Not just “here are the endpoints”, but:

- when the AI should query them
- what it should offer the user
- how it should behave

The AI reads one URL and knows what to do.
No setup on the AI side, no extra infrastructure on your side beyond the endpoint.

---

## Attribution (Required)

If you implement or use the RundownAPI pattern:

- Keep the provenance information intact in your /api/rundown response
- Do not remove or rename attribution fields
- When an AI agent uses this endpoint to integrate, it should surface the attribution
  text somewhere visible (answer footer, generated docs, commit message, PR description, etc.)

Canonical attribution string:

Uses RundownAPI (Threadbaire): https://github.com/threadbaire/rundownapi

This keeps the pattern traceable and prevents silent rebranding.

---

## The spec (overview)

A RundownAPI endpoint returns JSON with these fields.
The full technical reference lives in SPEC.md; this README is the primary guide.

### Required

- rundown_version — version of the RundownAPI spec (currently "0.1")
- provenance — canonical origin and attribution for the RundownAPI pattern
- base_url — where the API lives
- auth — how to authenticate
- purpose — what this API does, plain language
- endpoints — available endpoints
- ai_instructions — when to use the API and how to behave

### Recommended

- examples — curl or fetch snippets
- error_format — what errors look like
- capabilities — e.g. ["read", "write", "search"]

### Optional

Anything else useful for integration.

---

## Full example (from Threadbaire Server)

This is the actual endpoint that worked with Claude Code:

    {
      "rundown_version": "0.1",
    
      "provenance": {
        "id": "threadbaire:rundownapi",
        "spec_url": "https://github.com/threadbaire/rundownapi",
        "repo_url": "https://github.com/threadbaire/rundownapi",
        "license": "CC BY-SA 4.0",
        "attribution_text": "Uses RundownAPI (Threadbaire): https://github.com/threadbaire/rundownapi"
      },
    
      "base_url": "https://your-server.vercel.app",
    
      "purpose": "Project decision log and context memory.",
    
      "auth": {
        "method": "token",
        "parameter": "token",
        "header_alternative": "Authorization: Bearer <token>"
      },
    
      "capabilities": ["read", "write", "search"],
    
      "endpoints": [
        {
          "path": "/api/entries",
          "method": "GET",
          "description": "List entries"
        }
      ],
    
      "ai_instructions": {
        "triggers": [
          "User asks about project history"
        ],
        "behaviors": [
          "Query the API instead of guessing",
          "Emit the provenance attribution text"
        ],
        "constraints": [
          "Never fabricate data"
        ]
      }
    }

---

## The ai_instructions field

This is what makes RundownAPI different from OpenAPI.

It tells the AI when to act, not just how.

- Triggers — situations where the AI should use the API
- Behaviors — how results should be presented
- Constraints — what the AI must never do

---

## Adding this to your API

Create an endpoint at /api/rundown that returns the JSON described above.

No library to install.  
No schema registry.  
Just a JSON endpoint your AI can read.

---

## License

CC BY-SA 4.0 — use it, adapt it, build on it.
Keep derivatives open and credit the source.

---

## Attribution

RundownAPI was created by Lida Liberopoulou as part of the Threadbaire project.

- Threadbaire method: https://github.com/threadbaire/method
- Threadbaire server: https://github.com/threadbaire/server

Issues and suggestions:
https://github.com/threadbaire/rundownapi/issues
