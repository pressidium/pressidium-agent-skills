---
name: pressidium-read-edge-usage
description: Pressidium building block — EDGE site monthly usage (requests and data-transfer). Use for Growth Monitor on EDGE sites.
---

# read_edge_usage

**Permission:** read

## When to use

- Growth Monitor EDGE path after team capacity

## Steps

1. EDGE target `{ siteId }` only (not hosting website reports).
2. Call `pressidium_reports` with `reportType: "edge_usage_requests"` then `reportType: "edge_usage_data_transfer"`.
3. Prefer tool text; empty series → **Data unavailable**.
4. Static variants (`edge_usage_*_static`) only if the user asks.
5. Return snapshot:

```json
{
  "target": { "siteId": "abc123" },
  "edgeUsageRequests": { "status": "ok", "series": [] },
  "edgeUsageDataTransfer": { "status": "ok", "series": [] }
}
```
