---
name: pressidium-read-error-signals
description: Pressidium building block — recent PHP/error log lines. Use for Site Guardian and incident triage.
---

# read_error_signals

**Permission:** read

## When to use

- Site Guardian errors section (recent log tail — today's daily file, last N lines)
- Incident triage after user reports site issues
- Optional focused grep for fatal PHP errors
- EDGE has no error logs (`pressidium_logs` type `error` is hosting-only).

## Steps

1. Require `target` from `read_site_identity` or user input. Pass through `target.environment` from the caller (do not drop it).
2. Call `pressidium_logs` with:
   - `type: "error"`
   - `format: "inline"`
   - `lastSize: 100` (last 100 **lines** after filters, not a byte cursor)
   - optional `grepString`: `Fatal error|PHP` (message `|`-OR of word/phrase needles — **not** a regex; `PHP` does not match `index.php`)
   - default omits AH01276 / directory-index probe noise; pass `includeNoise: true` only when the user needs those lines
3. **One call is enough for composites** — do not make duplicate unfiltered + filtered calls for the same site.
4. Extract log lines from tool **text** first (header `Loaded N…` then the formatted lines). Use structured `lines` only if the harness exposes `structuredContent`. Do not treat a count-only summary as empty logs when lines follow the header.
5. Note if response was truncated (inline cap is 200 lines; request 100 by default).
6. Build snapshot:

```json
{
  "lines": [],
  "truncated": false,
  "grepString": "Fatal error|PHP"
}
```

7. Return snapshot to calling composite; do not modify logs or site state.
