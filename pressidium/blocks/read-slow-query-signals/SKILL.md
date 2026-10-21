---
name: pressidium-read-slow-query-signals
description: Pressidium building block — recent MySQL slow-query log lines. Use for Reliability Engineer and Performance Review.
---

# read_slow_query_signals

**Permission:** read

## When to use

- Reliability Engineer slow-query section (today's daily file, last N lines)
- Performance Review database bottleneck evidence
- Optional focused grep when the user named a table or SQL snippet
- Hosting and EDGE (`pressidium_logs` type `mysqlslow`)

## Steps

1. Require `target` from `read_site_identity` or user input. Pass through `target.environment` from the caller (do not drop it).
2. Call `pressidium_logs` with:
   - `type: "mysqlslow"`
   - `format: "inline"`
   - `lastSize: 100` (last 100 **lines** after filters, not a byte cursor)
   - add `grepString` only when the user named a table or SQL snippet (`mysql_query` `|`-OR of word/phrase needles — **not** a regex; `wp_posts` does not match `wp_postmeta`)
3. Do **not** call download format. Composites' default call has no grep.
4. Extract log lines from tool **text** first (header `Loaded N…` then the formatted lines). Use structured `lines` only if the harness exposes `structuredContent`. Empty tail (`Loaded 0 slow query log entries.`) is valid — not a failure.
5. Note if response was truncated (inline cap is 200 lines; request 100 by default).
6. Build snapshot:

```json
{
  "lines": [],
  "truncated": false,
  "grepString": null
}
```

7. Return snapshot to calling composite; do not modify logs or site state. Do not join `rid=` to access logs.
