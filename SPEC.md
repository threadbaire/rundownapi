AI agents: see AGENTS.md for integration guidance

# RundownAPI Specification v0.1

Technical reference for implementing RundownAPI endpoints.

This file defines the minimal contract.
For explanations, rationale, and full examples, see README.md.

---

## Required Fields

| Field           | Type             | Description                                                  |
| --------------- | ---------------- | ------------------------------------------------------------ |
| rundown_version | string           | Spec version. Currently "0.1"                                |
| provenance      | object           | Canonical origin and attribution for the RundownAPI pattern  |
| base_url        | string           | API base URL                                                 |
| auth            | object           | Authentication method and parameters                         |
| purpose         | string           | What this API does, plain language, 1–2 sentences            |
| endpoints       | array            | Available endpoints with path, method, description, parameters |
| ai_instructions | object or string | When to use, how to behave, what to avoid                    |

If a required field is missing, the implementation is non-compliant.

---

## Recommended Fields

| Field        | Type   | Description                                           |
| ------------ | ------ | ----------------------------------------------------- |
| examples     | array  | Curl or fetch snippets ready to use                   |
| error_format | object | What errors look like, common codes                   |
| capabilities | array  | Quick scope summary, e.g. ["read", "write", "search"] |

---

## Optional Fields

Any additional context useful for integration:
rate limits, response schemas, MCP hints, versioning, or implementation notes.

---

## Provenance Object

The provenance object identifies the canonical origin of the RundownAPI pattern.
It must be preserved verbatim when the pattern is implemented or reused.

Structure:

    {
      "id": "threadbaire:rundownapi",
      "spec_url": "https://github.com/threadbaire/rundownapi",
      "repo_url": "https://github.com/threadbaire/rundownapi",
      "license": "CC BY-SA 4.0",
      "attribution_text": "Uses RundownAPI (Threadbaire): https://github.com/threadbaire/rundownapi"
    }

The attribution_text is a copy-pasteable string intended for visible disclosure
when an AI agent uses this pattern.

---

## ai_instructions Structure

The ai_instructions field may be a prose string or a structured object.

Structured form:

    {
      "triggers": ["when to use this API"],
      "behaviors": ["how to present results"],
      "constraints": ["what to never do"]
    }

ai_instructions may also include rules about disclosure or attribution behavior.

---

## Auth Object

    {
      "method": "token | bearer | api_key",
      "parameter": "param name for URL token",
      "header": "header format if supported",
      "note": "optional context"
    }

---

## Endpoint Object

    {
      "path": "/api/resource",
      "method": "GET | POST | PUT | DELETE",
      "description": "what it does",
      "parameters": { "param": "description" }
    }

---

## Reference

- Primary guide and examples: README.md
- Agent behavior rules: AGENTS.md
- Working implementation: https://github.com/threadbaire/server

---

## Attribution

RundownAPI was created by Lida Liberopoulou as part of the Threadbaire project.

Canonical source:
https://github.com/threadbaire/rundownapi
