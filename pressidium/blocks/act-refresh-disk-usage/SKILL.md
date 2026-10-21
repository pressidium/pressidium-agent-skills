---
name: pressidium-act-refresh-disk-usage
description: Pressidium building block — trigger disk usage recalculation. Low-risk write; hosting only.
---

# act_refresh_disk_usage

Call `pressidium_website_refresh` with `action: "disk_usage"`, `wait: true` for the managed hosting `website` + `environment`.

After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.
Report `jobStatus`. After success, re-read `read_wordpress_state` (or site detail) if the workflow needs updated `diskUsage`.

Hosting only — not EDGE.
