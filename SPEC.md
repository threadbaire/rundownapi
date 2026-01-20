# RundownAPI Specification v0.1

Technical reference for implementing RundownAPI endpoints.

---

## Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `rundown_version` | string | Spec version. Currently `"0.1"` |
| `base_url` | string | API base URL |
| `auth` | object | Authentication method and parameters |
| `purpose` | string | What this API does, plain language, 1-2 sentences |
| `endpoints` | array | Available endpoints with path, method, description, parameters |
| `ai_instructions` | object or string | When to use, how to behave, what to avoid |

---

## Recommended Fields

| Field | Type | Description |
|-------|------|-------------|
| `examples` | array | Curl/fetch snippets ready to use |
| `error_format` | object | What errors look like, common codes |
| `capabilities` | array | Quick scope summary: `["read", "write", "search"]` |

---

## Optional Fields

Any additional context useful for integration: rate limits, response schemas, MCP hints, versioning.

---

## ai_instructions Structure

Can be a string (prose) or object:
```json
{
  "triggers": ["when to use this API"],
  "behaviors": ["how to present results"],
  "constraints": ["what to never do"]
}
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
  "parameters": { "param": "description" }
}
```

---

## Reference

- Full examples: [README.md](README.md)
- Agent instructions: [AGENTS.md](AGENTS.md)
- Working implementation: [github.com/threadbaire/server](https://github.com/threadbaire/server)
