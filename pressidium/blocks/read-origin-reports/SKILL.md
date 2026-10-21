---
name: pressidium-read-origin-reports
description: Pressidium building block — monthly origin visits and bandwidth series for managed hosting. Use for Growth Monitor capacity headroom.
---

# read_origin_reports

**Permission:** read

## When to use

- Growth Monitor hosting path after team capacity
- Capacity / visit trend questions on a production website environment

## Steps

1. Require hosting target `{ website, environment }` from `read_site_identity` (not EDGE `siteId`).
2. Call `pressidium_reports` with `reportType: "origin_visits"` then again with `reportType: "origin_bandwidth"`.
3. Prefer tool **text**. If `status` is `unsupported_environment` or text says production-only, treat as **Data unavailable** — do not invent OK.
4. One pass per `reportType` — do not re-call for the same site/window.
5. Return snapshot:

```json
{
  "target": { "website": "acorn2clone", "environment": "production" },
  "originVisits": { "status": "ok", "series": [] },
  "originBandwidth": { "status": "ok", "series": [] }
}
```
