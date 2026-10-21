---
name: pressidium-act-purge-cache
description: Pressidium building block — purge all caches after confirmed remediation. Low-risk write.
---

# act_purge_cache

Call `pressidium_purge_cache` with `scope: "all"`, `wait: true`.
Parse `jobStatus=` from tool **text** (not the headline). Do not treat “submitted” as success.
After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.
Report that `jobStatus` token.
