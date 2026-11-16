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
2. If target name/id unknown, call `pressidium_list_sites` and match user intent to a row:
   - **Hosting** (`- [hosting] myblog …`): note the website name.
   - **EDGE** (`- [edge] mystore #env_123456789 …` or `- [edge] #env_123456789 …`): note display name (if present) and the `#` token as `siteId`.
3. Call `pressidium_get_site`:
   - From a **hosting** list row: `{ website, environment, kind: "hosting" }` (default `environment` to `production` when omitted) so a later name clash cannot recur.
   - From an **EDGE** list row: `{ website: "mystore" }` when the user chose by name, or `{ siteId: "env_123456789" }` from the `#` token.
4. On clash, duplicate EDGE name, or cap errors in tool text, retry exactly as the error says (`siteId` for EDGE, or `website` with `kind: "hosting"` for managed hosting).
5. Build snapshot — `target` is always XOR-valid; display metadata stays on `siteSummary`, not on `target`.

**Hosting snapshot:**

```json
{
  "user": "<from whoami>",
  "target": { "website": "myblog", "environment": "production", "kind": "hosting" },
  "siteSummary": {
    "kind": "hosting",
    "url": "...",
    "state": "active",
    "wpVersion": "6.8.3",
    "phpVersion": "8.2"
  }
}
```

**EDGE snapshot** — callable target is `siteId` only; name lives on `siteSummary`:

```json
{
  "user": "<from whoami>",
  "target": { "siteId": "env_123456789" },
  "siteSummary": {
    "kind": "edge",
    "name": "mystore",
    "url": "...",
    "state": "live",
    "wpVersion": null,
    "phpVersion": null
  }
}
```

Do **not** put both `website` and `siteId` on `target`. Do **not** put EDGE display name on `target` (downstream blocks would re-enter name resolve and can hit a clash).

Parse `pressidium_whoami` **text** for email and team/role (structured `user`/`loggedIn` when the client exposes it). Parse hosting `pressidium_get_site` **text** for staging, WP/PHP, disk, mapped hostnames, and inventory tokens (`N plugins (M active)`, `K themes`, `active theme {name}`) when present. Copy `wpVersion` / `phpVersion` from structured when the harness exposes it; copy `summary` inventory counts when structured is visible. Do **not** pass `detail: "full"` unless the user asked for raw plugin/theme arrays. Omit inventory on EDGE; use `null` for WP/PHP on EDGE.

6. Return snapshot to calling composite; do not mutate the site.
