---
name: pressidium-read-analytics-overview
description: Pressidium building block — website analytics overview (visits, bandwidth, headline metrics) for a relative time window on EDGE or managed hosting. Use for Site Guardian digests and Performance Review.
---

# read_analytics_overview

**Permission:** read (auto-run OK)

## When to use

- Daily digest composites needing headline traffic/bandwidth numbers
- Before/after comparisons around deployments or incidents

## Steps

1. If target unknown, run the `read_site_identity` block first.
2. Call `pressidium_analytics_overview` with `{ target: { siteId }, range }` (EDGE) or `{ target: { website, environment }, range }` (managed hosting). Default range `1d`. Known-good ranges: `1d`, `7d`, `14d` (see `docs/research/analytics-edge-mock/range-matrix.md`).
3. If the call times out, retry identical arguments once, then report data unavailable.
4. If this composite already resolved the site's mapped domains, pass them as `domains` to skip the repeated site-detail fetch.
5. Never set `mock: true` in production composites. If the text summary starts with `[MOCK DATA]`, flag the entire report as synthetic.
6. Parse the tool **text** summary first; forward `structuredContent` (`metricGroups`, `range`, `domains`, `mock`, `data`) when the composite needs payload detail. Headline numbers are flattened scalars in text (not "Fields: Page 1"). If text says "No headline metrics in payload," treat overview metrics as unavailable. Honor Caveat A (`Website staging analytics uses the platform hostname only; custom mapped hosts were omitted: `) and Caveat B (`matches another environment's platform default`) if present; do not treat `No headline metrics in payload` or Caveat B as visits=0. Parse header `domains:` for the hostnames that were sent.
7. Return the snapshot to the calling composite; do not mutate the site:

```json
{
  "target": { "siteId": "env_123" },
  "range": "1d",
  "domains": ["www.example.com"],
  "mock": false,
  "overview": { "<merged dashboard-metrics payload>": "..." }
}
```

Hosting target shape example:

```json
{
  "target": { "website": "example", "environment": "production" },
  "range": "1d",
  "domains": ["www.example.com"],
  "mock": false,
  "overview": { "<merged dashboard-metrics payload>": "..." }
}
```
