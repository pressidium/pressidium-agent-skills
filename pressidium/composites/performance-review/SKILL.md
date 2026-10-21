---
name: pressidium-performance-review
description: Pressidium Operations composite — weekly performance bottleneck report. Invoke with "Run Performance Review on {website}" or an EDGE siteId.
---

# Performance Review

**Schedule intent:** weekly  
**Tier:** Operations (v1)  
**Orchestrates:** cache state + performance/cache analytics + mobile PerfAudit history + MySQL slow-query logs + traffic overview + plugin/WordPress context

## Block order

1. `read_site_identity`
2. `read_cache_state`
3. `read_analytics_performance` — `{ view: "overview", range: "7d" }` (EDGE `{ siteId }` or hosting `{ website, environment }`)
4. Optionally repeat `read_analytics_performance` with `{ view: "cache", range: "7d" }` if overview is thin or cache is the suspected bottleneck — reuse `domains`
5. `read_perfaudit_history` — `{ device: "mobile" }` (EDGE `{ siteId }` or hosting `{ website }`)
6. `read_slow_query_signals` — `{ type: "mysqlslow", format: "inline", lastSize: 100 }`; no `grepString` unless the user named a table or SQL snippet
7. `read_analytics_traffic` — `{ view: "overview", range: "7d" }`, reuse `domains`
8. `read_wordpress_state` **or** `read_plugin_state` (site scope) — **one pass only**, not both
9. `read_wp_core_state` — only if core/PHP context is not already clear from step 8 tool text; otherwise skip

### EDGE vs managed hosting

After `read_site_identity`, inspect `siteSummary.kind`:

| `kind` | Analytics / cache targets | PerfAudit target |
|--------|---------------------------|------------------|
| `edge` | `{ siteId }` (cache block as documented for EDGE) | `{ siteId }` |
| `hosting` | `{ website, environment }` from identity | `{ website }` (no environment segment) |

Identity line: hosting `{website} · {environment} · hosting` / EDGE `{siteId or website} · edge`.

## Agent rules (Cursor / text-first clients)

1. Use the **tool text response** as the primary data source.
2. **One pass only** — do not re-call the same tool for the same site. **Exception:** if an analytics tool call times out or returns the retry hint, retry **identical** arguments once, then treat a second failure as data unavailable. Do not change `view` or `range` on that retry. `view=cache` with `range=7d` can time out; that exception applies.
3. **Never** search the codebase or skill templates for live site facts.
4. If a section lacks concrete facts in tool text, write **"Data unavailable"** — do not invent OK.
5. Prefer `pressidium_wordpress_state` **or** `pressidium_plugins` — not both.
6. **Analytics:** Never set `mock: true`. If any analytics tool text starts with `[MOCK DATA]`, flag the report as synthetic.
7. **Domain reuse:** After the first analytics block that returns `domains`, pass them into later analytics calls.
8. **Read-only default:** Do **not** run write blocks (`act_purge_cache`, `act_apply_updates`, `act_perfaudit_test`, …) unless the user explicitly opts in after the report.
9. **Slow queries:** rank from text; do not re-call to sort. An empty tail is not a global "database is fine" claim — it only means today's slow-query file has no entries.

## Report template

```
# Performance Review — {website or siteId} — {date}

## Executive summary
{one paragraph: primary bottlenecks}

## Cache / CDN
{from read_cache_state}

## Performance analytics (7d)
{from read_analytics_performance}

## PerfAudit (mobile)
{from read_perfaudit_history — FCP, LCP, CLS, TTFB, scores when present}

## Slow queries
{from read_slow_query_signals — top slow queries from today's file, or none if the tail is empty}

## Traffic context
{from read_analytics_traffic overview}

## Plugin / update risk
{from wordpress_state or plugin_state — perf-relevant only}

## Priority actions
1. ...
(Recommend purge/exclusion review or a fresh PerfAudit test via `act_perfaudit_test` only after user opt-in; do not auto-run writes)

## v1 limitations
- Origin/capacity report tools are not included (roadmap 2b).
- GSC / field CWV data and automatic cache purge remain out of scope.
```

**Still out of scope:** GSC/CWV field data, automatic cache purge, fleet PerfAudit, full HTML PerfAudit reports.

Default Performance Review run is **read-only** — never call `act_perfaudit_test` unless the user opts in after reviewing the report.
