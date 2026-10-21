---
name: pressidium-read-analytics-performance
description: Pressidium building block — performance and cache analytics (timings, cache hit behaviour) for EDGE or managed hosting. Use for Performance Review and cache-tuning diagnostics.
---

# read_analytics_performance

**Permission:** read (auto-run OK)

## When to use

- Weekly Performance Review bottleneck reports
- Cache effectiveness checks after purges or exclusion changes

## Steps

1. If target unknown, run the `read_site_identity` block first.
2. Call `pressidium_analytics_performance` with `{ target: { siteId }, view, range }` (EDGE) or `{ target: { website, environment }, view, range }` (managed hosting):
   - `view: "overview"` (alias `summary`) for headline performance metrics (default)
   - `view: "analytics"` for detailed performance analytics
   - `view: "cache"` for cache analytics
   Do not pass other view names.
3. If the call times out, retry identical arguments once, then report data unavailable. `view=cache` with `range=7d` can time out; retry identical arguments.
4. Known-good ranges: `1d`, `7d`, `14d` (see `docs/research/analytics-edge-mock/range-matrix.md`). Default `1d` for post-change checks; `7d` for weekly reviews.
5. Reuse `domains` from earlier blocks in the same composite when available.
6. Never set `mock: true` in production composites. If the text summary starts with `[MOCK DATA]`, flag the entire report as synthetic.
7. Parse the tool **text** summary first; forward `structuredContent` when the composite needs payload detail. Headline numbers are flattened scalars in text (not "Fields: Page 1"). If text says "No headline metrics in payload," treat performance/cache-hit/TTFB scalars as unavailable. Honor Caveat A (`Website staging analytics uses the platform hostname only; custom mapped hosts were omitted: `) and Caveat B (`matches another environment's platform default`) if present; do not treat `No headline metrics in payload` or Caveat B as visits=0. Parse header `domains:` for the hostnames that were sent.
8. Return the snapshot to the calling composite; do not mutate the site:

```json
{
  "target": { "siteId": "env_123" },
  "view": "cache",
  "range": "1d",
  "domains": ["www.example.com"],
  "mock": false,
  "performance": { "<merged dashboard-metrics payload>": "..." }
}
```

Hosting target shape example:

```json
{
  "target": { "website": "example", "environment": "production" },
  "view": "cache",
  "range": "1d",
  "domains": ["www.example.com"],
  "mock": false,
  "performance": { "<merged dashboard-metrics payload>": "..." }
}
```
