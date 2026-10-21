---
name: pressidium-growth-monitor
description: Pressidium Operations composite — traffic and capacity headroom report. Invoke with "Run Growth Monitor on {website}" or an EDGE siteId.
---

# Growth Monitor

**Schedule intent:** on-demand / weekly  
**Tier:** Operations / host-boundary (v1)  
**Orchestrates:** traffic overview + analytics overview + team capacity + origin/disk (hosting) or EDGE usage

## Block order

1. `read_site_identity`
2. `read_analytics_traffic` — `{ view: "overview", range: "7d" }` by default; use `range: "14d"` only when the user asks for a longer window (EDGE `{ siteId }` or hosting `{ website, environment }`)
3. `read_analytics_overview` — same `range` as step 2, reuse `domains`
4. `read_team_capacity`
5. By `kind` from identity:
   - **hosting:** `read_origin_reports` then `read_disk_trends` (pass `{ website, environment }`)
   - **edge:** `read_edge_usage` (pass `{ siteId }`)

### EDGE vs managed hosting

| `kind` | Analytics targets | Reports / usage |
|--------|-------------------|-----------------|
| `edge` | `{ siteId }` | `read_edge_usage` |
| `hosting` | `{ website, environment }` from identity | `read_origin_reports`, `read_disk_trends` |

## Agent rules (Cursor / text-first clients)

1. Use the **tool text response** as the primary data source.
2. **One pass only** — do not re-call the same tool for the same site. **Exception:** if an analytics tool call times out or returns the retry hint, retry **identical** arguments once, then treat a second failure as data unavailable. Do not change `view` or `range` on that retry. `range=7d` / `14d` traffic and overview can time out; same exception.
3. **Never** search the codebase or skill templates for live site facts.
4. If a section lacks concrete facts in tool text, write **"Data unavailable"** — do not invent OK.
5. **Analytics:** Never set `mock: true`. If any analytics tool text starts with `[MOCK DATA]`, flag the report as synthetic.
6. **Domain reuse:** After `read_analytics_traffic`, pass returned `domains` into `read_analytics_overview`.
7. **Ownership boundary:** Host-ops framing only (“Would the customer blame the host?”). No SEO strategy, content advice, or marketing copy. Do not claim GSC/GA4 insights.
8. Do **not** run write blocks unless the user explicitly opts in after the report.
9. Do **not** emit dual Plan/Fleet verdicts (UPGRADE/STAY/DOWNGRADE or ROOM/TIGHT/FULL) — defer that advice to **Capacity Advisor** (`Run Capacity Advisor` or `Run Capacity Advisor on {website}` / EDGE siteId).

## Report template

```
# Growth Monitor — {website or siteId} — {date}

## Traffic / visits ({range})
{from read_analytics_traffic overview}

## Bandwidth / headline metrics
{from read_analytics_overview}

## Capacity vs plan
{from read_team_capacity}

Do **not** emit dual Plan/Fleet verdict labels here — defer upgrade/stay/downgrade and fleet room/tight/full advice to **Capacity Advisor**.

## Origin visits / bandwidth (hosting only)
{from read_origin_reports — monthly series; Data unavailable if non-production or empty}

## Disk trends (hosting only)
{from read_disk_trends — monthly series; Data unavailable if non-production or empty}

## EDGE usage (EDGE only)
{from read_edge_usage — requests and data-transfer series}

## Anomalies / headroom
{warnings or OK}

## Priority actions
1. ...
(Hosting/ops only — e.g. capacity headroom, traffic spikes needing infra attention)

## v1 limitations
- Google Search Console and GA4 connectors are not included (Phase 4).
```

**Still out of scope in v1:** SEO/content analysis, GSC/GA4.

For **Plan** (upgrade/stay/downgrade) and **Fleet** (room/tight/full) advice, run **Capacity Advisor** (`Run Capacity Advisor` or `Run Capacity Advisor on {website}` / EDGE siteId). Growth Monitor keeps traffic + origin/disk headroom context only.

Do not run write blocks in default Growth Monitor run.
