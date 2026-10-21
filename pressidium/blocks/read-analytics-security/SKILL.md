---
name: pressidium-read-analytics-security
description: Pressidium building block — security overview (blocked requests, bots, top IPs/countries) for EDGE or managed hosting. Use for Site Guardian security signals and incident triage.
---

# read_analytics_security

**Permission:** read (auto-run OK)

## When to use

- Site Guardian security posture checks in daily digests
- Incident triage when attack traffic or blocking activity is suspected

## Steps

1. If target unknown, run the `read_site_identity` block first.
2. Call `pressidium_analytics_security` with `{ target: { siteId }, range }` (EDGE) or `{ target: { website, environment }, range }` (managed hosting) — overview view only (blocked requests, bots, top IPs/countries).
3. If the call times out, retry identical arguments once, then report data unavailable.
4. Known-good ranges: `1d`, `7d`, `14d` (see `docs/research/analytics-edge-mock/range-matrix.md`). Use `1d` for suspected attacks; `7d` for digest rollups.
5. Reuse `domains` from earlier blocks in the same composite when available.
6. Never set `mock: true` in production composites. If the text summary starts with `[MOCK DATA]`, flag the entire report as synthetic.
7. Parse the tool **text** summary first; forward `structuredContent` when the composite needs payload detail. Honor Caveat A/B if present; do not treat `No headline metrics in payload` or Caveat B as visits=0; parse header `domains:`.
8. Return the snapshot to the calling composite; do not mutate the site:

```json
{
  "target": { "siteId": "env_123" },
  "range": "1d",
  "domains": ["www.example.com"],
  "mock": false,
  "security": { "<merged dashboard-metrics payload>": "..." }
}
```

Hosting target shape example:

```json
{
  "target": { "website": "example", "environment": "production" },
  "range": "1d",
  "domains": ["www.example.com"],
  "mock": false,
  "security": { "<merged dashboard-metrics payload>": "..." }
}
```
