---
name: pressidium-read-site-identity
description: Pressidium building block — resolve website or EDGE siteId, load identity and site summary. Use when a workflow needs a confirmed target or site metadata before other blocks run.
---

# read_site_identity

**Permission:** read (auto-run OK)

## When to use

- Start of any site-scoped composite when `website` or `siteId` is unknown
- Verify MCP session before fleet operations

## Steps

1. Call `pressidium_whoami`. If not authenticated, stop and report.
2. If target name/id unknown, call `pressidium_list_sites` and match user intent.
3. Call `pressidium_get_site` with `{ website }` or `{ siteId }` and optional `environment`.
4. Build snapshot:

```json
{
  "user": "<from whoami>",
  "target": { "website": "myblog", "environment": "production" },
  "siteSummary": {
    "kind": "hosting",
    "url": "...",
    "state": "active",
    "wpVersion": "6.8.3",
    "phpVersion": "8.2"
  }
}
```

Parse `pressidium_whoami` **text** for email and team/role (structured `user`/`loggedIn` when the client exposes it). Parse hosting `pressidium_get_site` **text** for staging, WP/PHP, disk, mapped hostnames, and inventory tokens (`N plugins (M active)`, `K themes`, `active theme {name}`) when present. Copy `wpVersion` / `phpVersion` from structured when the harness exposes it; copy `summary` inventory counts when structured is visible. Do **not** pass `detail: "full"` unless the user asked for raw plugin/theme arrays. Omit inventory on EDGE; use `null` for WP/PHP on EDGE.

5. Return snapshot to calling composite; do not mutate the site.
