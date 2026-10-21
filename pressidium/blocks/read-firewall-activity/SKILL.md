---
name: pressidium-read-firewall-activity
description: Pressidium building block — firewall access-rule activity aggregates (platform + user groups) for EDGE or managed hosting. Use for Reliability Engineer block-count RCA; read-only; dual-product.
---

# read_firewall_activity

**Permission:** read (auto-run OK)

## When to use

- Reliability Engineer — identify active firewall rules and recent block/pass counts
- Incident triage when access blocks or WAF activity may explain HTTP errors or traffic drops

## Steps

1. If target unknown, run the `read_site_identity` block first.
2. Build the **dual-product** target (same pattern as analytics blocks):
   - **EDGE:** `{ target: { siteId } }`
   - **Managed hosting:** `{ target: { website, environment: "production" } }` (or omit `environment`). Do not pass `staging`, even when site identity is staging.
3. Call `pressidium_firewall_activity` **twice** with the same target — required `group` per call:
   - First: `{ target, group: "platform" }`
   - Second: `{ target, group: "user" }`
4. Parse the tool **text** summary first (rule `id`, `action`, `ip`/`country`, `active`, `createdAt`, `1h`/`24h` event counts when present); forward `structuredContent` when the composite needs payload detail.
5. Empty list for a group is a successful response — write **no firewall activity** for that group, not **Data unavailable**. When identity or the user ask is hosting **staging**, the firewall section must say **production** (data source). Do **not** treat those production counts as evidence of staging-environment activity without corroboration from another staging-scoped signal (for example staging logs). If the tool returns `Firewall activity is not available on the staging environment. Use production.`, quote it and do not invent rule counts.
6. Return the snapshot to the calling composite; do not mutate the site:

```json
{
  "target": { "siteId": "env_123" },
  "platform": { "<platform rule activity>": "..." },
  "user": { "<user rule activity>": "..." }
}
```

Hosting target shape example:

```json
{
  "target": { "website": "acorn2clone", "environment": "production" },
  "platform": { "<platform rule activity>": "..." },
  "user": { "<user rule activity>": "..." }
}
```

**Read-only** — does not create or modify firewall rules (rule CRUD remains out of scope).
