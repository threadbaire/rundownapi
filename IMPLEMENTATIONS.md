# RundownAPI Implementations

Known implementations of the RundownAPI pattern. If you've added `/api/rundown` to your project, submit a PR to be listed.

---

## Reference Implementation

### Threadbaire Server

- **Repository:** [github.com/threadbaire/server](https://github.com/threadbaire/server)
- **Endpoint:** `/app/api/rundown/route.ts`
- **Status:** Reference implementation, tested with Claude Code
- **Notes:** Project decision log and context memory. First successful AI self-integration test (January 2026).

---

## Community Implementations

*None yet. Be the first!*

---

## Adding Your Implementation

To be listed, your implementation must:

1. Expose a `/api/rundown` endpoint (or document your alternative path)
2. Include all REQUIRED fields from [SPEC.md](SPEC.md)
3. Preserve the provenance object exactly as specified
4. Be publicly accessible (open source preferred)

Submit a PR with:

- Project name
- Repository URL
- Endpoint path
- Brief description

---

## Discovery Convention

Implementations SHOULD also serve the same content at:

- `/.well-known/rundown.json`

This gives agents a predictable probe path on any domain.
