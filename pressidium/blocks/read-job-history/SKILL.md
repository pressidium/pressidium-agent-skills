---
name: pressidium-read-job-history
description: Pressidium building block — recent operation notifications for a site. Use for Site Guardian and post-action verification.
---

# read_job_history

**Permission:** read

## When to use

- Verify recent backup, cache purge, or update jobs completed
- Site Guardian activity summary (last 24h)
- Post-action confirmation after write blocks

## Steps

1. Require `target` from `read_site_identity` or user input (`website` name).
2. Compute `since` as Unix epoch seconds for 24 hours ago.
3. Call `pressidium_operations` with:
   - `action: "notifications"`
   - `since`: epoch from step 2
   - `resource`: website name from target
4. Parse notifications **text** for type, status, job id, and title. Use `structured.items` when the client exposes it.
5. Filter items whose title/resource matches the target website if the API returns broader results.
6. When reporting notification times to users, prefer `timestampHuman` over raw `timestamp` (Unix seconds).
7. Build snapshot:

```json
{
  "items": []
}
```

8. Return snapshot to calling composite; do not submit or await jobs.
