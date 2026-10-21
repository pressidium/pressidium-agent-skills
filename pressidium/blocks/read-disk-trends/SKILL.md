---
name: pressidium-read-disk-trends
description: Pressidium building block — monthly disk usage series for managed hosting. Use for Growth Monitor and Maintenance Advisor disk trends.
---

# read_disk_trends

**Permission:** read

## When to use

- Growth Monitor hosting path after origin reports
- Disk growth vs plan pressure

## Steps

1. Hosting target `{ website, environment }` only.
2. Call `pressidium_reports` with `reportType: "disk"`.
3. Prefer tool text; soft-unsupported / empty → **Data unavailable**.
4. Return snapshot:

```json
{
  "target": { "website": "acorn2clone", "environment": "production" },
  "disk": { "status": "ok", "series": [] }
}
```
