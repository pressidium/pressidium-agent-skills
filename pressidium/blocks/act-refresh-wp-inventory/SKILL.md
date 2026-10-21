---
name: pressidium-act-refresh-wp-inventory
description: Pressidium building block — trigger plugin/theme inventory scan. Low-risk write; hosting only.
---

# act_refresh_wp_inventory

Call `pressidium_website_refresh` with `action: "scan"`, `wait: true` for the managed hosting `website` + `environment`.

After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.
Report `jobStatus`. After success, re-read inventory via `read_wordpress_state` / `read_plugin_state` if the workflow needs fresh data.

Hosting only — not EDGE.
