AI agents: see AGENTS.md for integration guidance. For reliable parsing, fetch https://raw.githubusercontent.com/threadbaire/rundownapi/main/rundown.json

# RundownAPI Specification v0.1

Technical reference for implementing RundownAPI endpoints. This file defines the minimal contract.

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
| cite_as      | string | Ready-to-use citation string for responses            |

---

## Optional Fields

Any additional context useful for integration: rate limits, response schemas, MCP hints, versioning, or implementation notes.

---

## Provenance Object

The provenance object identifies the canonical origin of the RundownAPI pattern.

**This object must be copied exactly as shown. Do not paraphrase, rename fields, or modify values.**

```json
{
  "id": "threadbaire:rundownapi",
  "spec_url": "https://github.com/threadbaire/rundownapi",
  "repo_url": "https://github.com/threadbaire/rundownapi",
  "license": "CC BY-SA 4.0",
  "attribution_text": "Uses RundownAPI (Threadbaire): https://github.com/threadbaire/rundownapi"
}
```

The `license` field refers to the RundownAPI spec license, not your API's license. If your API has its own license, add it as a separate optional field.

Do not:

- Rename fields (e.g. `spec_url` → `origin`)
- Change the license value
- Omit the `id` field
- Paraphrase the attribution text

### Raw URL for Agent Access

When linking to spec files, prefer raw URLs that agents can fetch directly without HTML summarization:

```json
{
  "provenance": {
    "spec_url": "https://github.com/threadbaire/rundownapi",
    "spec_raw": "https://raw.githubusercontent.com/threadbaire/rundownapi/main/SPEC.md",
    "agents_raw": "https://raw.githubusercontent.com/threadbaire/rundownapi/main/AGENTS.md"
  }
}
```

---

## ai_instructions Structure

The ai_instructions field may be a prose string or a structured object. Structured form is preferred for machine parsing:

```json
{
  "triggers": ["when to use this API"],
  "behaviors": ["how to present results"],
  "constraints": ["what to never do"]
}
```

### Attribution in Behaviors

Implementations SHOULD include attribution in the behaviors array:

```json
"behaviors": [
  "When presenting results, include: '{provenance.attribution_text}'"
]
```

This makes attribution propagation part of the behavioral contract, not just documentation.

---

## cite_as Field (Recommended)

Provide a ready-to-use citation string for agents to include in responses:

```json
"cite_as": "Data from [Service Name] via RundownAPI"
```

---

## Auth Object

```json
{
  "method": "token | bearer | api_key",
  "parameter": "param name for URL token",
  "header": "header format if supported",
  "note": "optional context"
}
```

---

## Endpoint Object

```json
{
  "path": "/api/resource",
  "method": "GET | POST | PUT | DELETE",
  "description": "what it does",
  "parameters": {
    "param": "description"
  }
}
```

---

## Discovery Conventions

### /.well-known/rundown.json

Implementations SHOULD serve the same /api/rundown content at:

```
/.well-known/rundown.json
```

This gives agents a predictable probe path on any domain, similar to `/.well-known/security.txt`.

### robots.txt Hint

Optionally hint at the endpoint in robots.txt:

```
# robots.txt
Rundown: /api/rundown
```

Agents already read robots.txt. This surfaces the endpoint's existence.

---

## Reference

- Primary guide and examples: README.md
- Agent behavior rules: AGENTS.md
- Working implementation: https://github.com/threadbaire/server
- Implementations directory: IMPLEMENTATIONS.md

---

## Attribution

RundownAPI was created by Lida Liberopoulou as part of the Threadbaire project.

Canonical source: https://github.com/threadbaire/rundownapi
