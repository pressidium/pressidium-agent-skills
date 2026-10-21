---
name: pressidium-read-analytics-traffic
description: Pressidium building block — traffic overview and request analytics (volume, status classes, trends) for EDGE or managed hosting. Use for Reliability Engineer 5xx checks and Growth Monitor.
---

# read_analytics_traffic

**Permission:** read (auto-run OK)

## When to use

- Incident triage: request volume and error-class trends over the incident window
- Growth/traffic anomaly checks in scheduled digests

## Steps

1. If target unknown, run the `read_site_identity` block first.
2. Call `pressidium_analytics_traffic` with `{ target: { siteId }, view, range }` (EDGE) or `{ target: { website, environment }, view, range }` (managed hosting):
   - `view: "overview"` for headline traffic numbers (default)
   - `view: "analytics"` for detailed request analytics
3. If the call times out, retry identical arguments once, then report data unavailable.
4. Known-good ranges: `1d`, `7d`, `14d` (see `docs/research/analytics-edge-mock/range-matrix.md`). Use `1d` for incident windows and daily digests; `7d` or `14d` for trend comparisons.
5. Reuse `domains` from earlier blocks in the same composite when available.
6. Never set `mock: true` in production composites. If the text summary starts with `[MOCK DATA]`, flag the entire report as synthetic.
7. Parse the tool **text** summary first; forward `structuredContent` when the composite needs payload detail. Honor Caveat A/B if present; do not treat `No headline metrics in payload` or Caveat B as visits=0; parse header `domains:`.
8. Return the snapshot to the calling composite; do not mutate the site:

```json
{
  "target": { "siteId": "env_123" },
  "view": "analytics",
  "range": "1d",
  "domains": ["www.example.com"],
  "mock": false,
  "traffic": { "<merged dashboard-metrics payload>": "..." }
}
```

Hosting target shape example:

```json
{
  "target": { "website": "example", "environment": "production" },
  "view": "overview",
  "range": "1d",
  "domains": ["www.example.com"],
  "mock": false,
  "traffic": { "<merged dashboard-metrics payload>": "..." }
}
```
