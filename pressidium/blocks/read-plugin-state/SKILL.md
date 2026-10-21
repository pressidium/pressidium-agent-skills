---
name: pressidium-read-plugin-state
description: Pressidium building block — plugin vulnerabilities and updates (site or fleet). Use for Plugin Auditor and Maintenance Advisor.
---

# read_plugin_state

**Permission:** read

## Steps

1. **Fleet:** `pressidium_plugins` `scope: "account"`, `action: "list"`.
2. **Site:** `scope: "site"`, `target`, `action: "list"`.
3. Parse plugin names and counts from the tool **text**. **Fleet:** parse `Sites with issues:` then named findings (`plugin @ website`). **Site:** lists `Vulnerable plugins:` and `Updatable:` lines. Use `structuredContent` when your client exposes it.
4. For fleet, fill site names from the rollup line in **text**. Do not require grouping `installations[].website` from structured-only.
5. **Skip this block** when a composite already ran `read_wordpress_state` for the same site — reuse `snapshot.pluginsSummary` and `snapshot.plugins` instead (avoids a duplicate `GET /website/{name}`).
