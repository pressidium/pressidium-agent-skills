---
name: pressidium-read-access-signals
description: Pressidium building block — recent access log lines with HTTP error filtering. Use for incident workflows and 5xx detection.
---

# read_access_signals

**Permission:** read

## When to use

- Incident workflows after user reports outages or slow responses
- Detect HTTP 500/502 patterns in access logs
- Complement `read_error_signals` when PHP logs are inconclusive

## Steps

1. Require `target` from `read_site_identity` or user input.
2. Call `pressidium_logs` with:
   - `type: "access"`
   - `format: "inline"`
   - `lastSize: 100` (last 100 **lines** after filters)
   - `httpStatus: "5xx"` (or `"500,502"` when the user asked for those codes)
   - Do **not** use `grepString` for status codes — on access logs it filters `request_path`.
3. Prefer tool **text**. The tool already filtered by `httpStatus`. Optionally include 4xx by calling with `httpStatus: "4xx,5xx"` instead of post-filtering. If the harness exposes `structuredContent`, `structured.entries` are the same filtered set.
4. Build snapshot from tool text / filtered entries (`lines`, and `entries` when using structured). Do **not** require structured output — text-only harnesses are enough.
5. Note if response was truncated.
6. Build snapshot:

```json
{
  "entries": [],
  "lines": [],
  "errorCount": 0,
  "truncated": false
}
```

7. Return snapshot to calling composite; do not modify logs or site state.
