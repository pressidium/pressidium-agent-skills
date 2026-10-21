---
name: pressidium-read-cache-state
description: Pressidium building block — page and object cache settings plus CDN status. Use for Site Guardian and infrastructure diagnostics on hosting sites.
---

# read_cache_state

**Permission:** read

## When to use

- Site Guardian infrastructure section
- Before recommending cache purge after remediation
- Hosting sites only — EDGE uses a different cache model; skip or note limitation

## Steps

1. Require `target` from `read_site_identity` or user input (`website`, `environment`).
2. If `siteSummary.kind` is `edge`, set snapshot `{ pageCache: null, objectCache: null, cdnEnabled: null, note: "EDGE cache model differs; use EDGE-specific tools" }` and return.
3. Call `pressidium_cache_settings` with `action: "get"`, `cacheType: "page"`, and `target`.
4. Call `pressidium_cache_settings` with `action: "get"`, `cacheType: "object"`, and `target`.
5. Call `pressidium_cdn` (read) for hosting sites to determine CDN enabled state.
6. Parse each cache get **text** for `page cache: on|off` / `object cache: on|off`. Use `structuredContent.data` when the client exposes it. Build snapshot:

```json
{
  "pageCache": { "enabled": true },
  "objectCache": { "enabled": true },
  "cdnEnabled": true
}
```

7. Return snapshot to calling composite; do not change cache settings.
