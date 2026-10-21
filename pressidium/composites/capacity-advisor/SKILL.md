---
name: pressidium-capacity-advisor
description: Pressidium Operations composite — plan fit (upgrade/stay/downgrade) and fleet headroom. Invoke with "Run Capacity Advisor" or "Run Capacity Advisor on {website}" / EDGE siteId.
---

# Capacity Advisor

**Schedule intent:** on-demand / monthly  
**Tier:** Operations / host-boundary  
**Orchestrates:** team capacity (+ optional site origin/disk or EDGE usage)

## Block order

### With target (`Run Capacity Advisor on {website}` or EDGE siteId)

1. `read_site_identity`
2. `read_team_capacity`
3. By `kind` from identity:
   - **hosting:** `read_origin_reports` then `read_disk_trends` (pass `{ website, environment }`)
   - **edge:** `read_edge_usage` (pass `{ siteId }`)
4. Emit report — site series support Plan/Fleet rationale only

### Account-only (`Run Capacity Advisor` — no site target)

1. `read_team_capacity`
2. Emit dual **Plan** + **Fleet** verdicts and Priority actions (omit Site pressure sections)

Do **not** call `read_analytics_traffic` or `read_analytics_overview` from this composite (those belong to Growth Monitor). Never set `confirm: true`.

## Agent rules (Cursor / text-first clients)

1. Use the **tool text response** as the primary data source.
2. **One pass only** — do not re-call the same tool for the same account/site in one run.
3. **Never** search the codebase or skill templates for live site facts.
4. If a section lacks concrete facts in tool text, write **"Data unavailable"** — do not invent OK.
5. **Never** set `mock: true`. If any tool text starts with `[MOCK DATA]`, flag the report as synthetic.
6. Default run is **read-only** — no subscription changes, no confirmed write actions, no `act_refresh_disk_usage` unless the user explicitly opts in **after** the report.
7. Prefer normalized `pressidium_team_usage` `GB` + `%` text (and `(high)` at ~80%). Treat `units may differ` as an incomparable fallback, not the happy path. Use available plan names when present.
8. Ownership boundary: plan economics + fleet add-capacity only. No SEO/content/marketing. Point traffic anomaly deep-dives to Growth Monitor.
9. To rank hosting sites by disk, call `pressidium_list_sites` with `sortBy: currentDiskUsage` (typically `includeEdge: false`) and parse `disk:` from **text**. Do not assume the Dashboard API accepted the query param.

## Verdict heuristics

Agent-applied from tool text (not an MCP scorer). Align warning threshold with ~80% / `CAPACITY_WARNING_RATIO` when ratios are comparable.

| Axis | Label | When |
|------|-------|------|
| **Plan** | **UPGRADE** | Any paired limit at/above warning (~80%) **or** Fleet **FULL** with clear pressure; mention relevant available plan names when present |
| **Plan** | **DOWNGRADE** | Usage clearly well below limits on comparable metrics **and** no site-pressure growth signals; never on thin/incomparable data |
| **Plan** | **STAY** | Default when neither UPGRADE nor DOWNGRADE is justified |
| **Fleet** | **FULL** | Sites/staging (when limits exist) at/near plan limit, or comparable disk exhausted |
| **Fleet** | **TIGHT** | Approaching limits (near warning) but not exhausted |
| **Fleet** | **ROOM** | Clear headroom on fleet-relevant limits |
| Either | **Data unavailable** | Thin subscription/usage or incomparable units — do not invent OK |

## Report template

```
# Capacity Advisor — {account | website or siteId} — {date}

## Verdicts
- Plan: {UPGRADE | STAY | DOWNGRADE} — {one-line rationale}
- Fleet: {ROOM | TIGHT | FULL} — {one-line rationale}

## Capacity vs plan
{from read_team_capacity / pressidium_team_usage text}

## Available plans
{brief catalog from tool text, or Data unavailable / omitted when absent}

## Site pressure (only if target given)
### Origin visits / bandwidth (hosting)
{from read_origin_reports — Data unavailable if non-production or empty}

### Disk trends (hosting)
{from read_disk_trends}

### EDGE usage (EDGE)
{from read_edge_usage}

## Priority actions
1. ...
(Plan/fleet only — e.g. consider upgrade, avoid adding sites, optional act_refresh_disk_usage after opt-in)

## v1 limitations
- No subscription change or billing actions.
- Google Search Console and GA4 connectors are not included (Phase 4).
- Disk or other metrics may remain incomparable if API units cannot be normalized safely.
- Capacity Advisor does not replace Growth Monitor (traffic/ops) or Site Guardian.
```

Do not run write blocks in the default Capacity Advisor run.
