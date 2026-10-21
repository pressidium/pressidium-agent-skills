---
name: pressidium-read-team-capacity
description: Pressidium building block — team usage, subscription limits, and available plans. Use for Site Guardian capacity checks and fleet planning.
---

# read_team_capacity

**Permission:** read

## When to use

- Site Guardian capacity one-liner
- Before recommending new sites, staging environments, or fleet expansions
- Compare current usage against plan limits

## Steps

1. Call `pressidium_team_usage` (no site target required — account scope).
2. Parse tool **text** (`Team capacity. Plan: … Usage: disk 14.52 GB / 30 GB (48%)` — cdn/visits similar, e.g. `cdn 1.20 GB / 10 GB (12%); visits 4200 / 10000 (42%)`. When a limit is missing: `(no quota on this plan)`. `(units may differ)` only if still present after conversion; may append `Available plans: …`). Prefer those text `GB` + `%` figures. Do not compute % from raw structured usage vs quota (`diskUsage` is bytes, `diskQuota` is GB). Structured output is optional:
   - `usage` — current consumption metrics
   - `subscription` — active plan and limits (if present)
   - `availablePlans` — upgrade catalog (if present)
3. Compare using **text** percents (or the same converted pairs). Flag when usage exceeds 80% of any limit as `status: "warning"`.
4. Build snapshot:

```json
{
  "usage": {},
  "subscription": {},
  "availablePlans": []
}
```

5. Return snapshot to calling composite; do not change subscription or team settings.
