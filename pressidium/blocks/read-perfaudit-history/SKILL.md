---
name: pressidium-read-perfaudit-history
description: Pressidium building block — mobile PerfAudit test history (FCP, LCP, CLS, TTFB, scores) for EDGE or managed hosting. Use for Performance Review CWV history; read-only.
---

# read_perfaudit_history

**Permission:** read (auto-run OK)

## When to use

- Weekly Performance Review — mobile Core Web Vitals history alongside analytics
- Comparing recent on-demand PerfAudit runs without starting a new test

## Steps

1. If target unknown, run the `read_site_identity` block first.
2. Call `pressidium_perfaudit` with `{ target, action: "history", device: "mobile" }`:
   - EDGE: `{ target: { siteId } }`
   - Managed hosting: `{ target: { website } }` (no `environment` segment for PerfAudit paths)
3. Default `device: "mobile"`. Use `device: "desktop"` only when the caller explicitly requests desktop history.
4. Parse the tool **text** summary first (timestamp, url, device, FCP/LCP/CLS/TTFB/TBT/SpeedIndex/PerformanceScore when present); forward `structuredContent` when the composite needs payload detail.
5. Empty history is a successful response — the composite should write **Data unavailable** for the section, not invent OK metrics.
6. Return the snapshot to the calling composite; do not mutate the site:

```json
{
  "target": { "siteId": "env_123" },
  "action": "history",
  "device": "mobile",
  "history": { "<PerfAudit test summaries>": "..." }
}
```

Hosting target shape example:

```json
{
  "target": { "website": "acorn2clone" },
  "action": "history",
  "device": "mobile",
  "history": { "<PerfAudit test summaries>": "..." }
}
```

Optional drill-down: call `pressidium_perfaudit` with `{ action: "summary", testId }` when the user asks for one test from the history list — no separate block required.
