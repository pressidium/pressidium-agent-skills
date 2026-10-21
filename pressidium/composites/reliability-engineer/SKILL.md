---
name: pressidium-reliability-engineer
description: Pressidium Operations composite — incident RCA from logs, jobs, analytics, audit, and firewall activity. Invoke with "Run Reliability Engineer on {website}" or an EDGE siteId.
---

# Reliability Engineer

**Schedule intent:** on incident  
**Tier:** Operations (v1)  
**Orchestrates:** error + access logs + slow-query logs + job history + traffic/security analytics + audit log + firewall activity

## Block order

1. `read_site_identity`
2. `read_error_signals`
3. `read_access_signals`
4. `read_slow_query_signals` — `{ type: "mysqlslow", format: "inline", lastSize: 100 }`; no `grepString` unless the user named a table or SQL snippet
5. `read_job_history`
6. `read_analytics_traffic` — prefer `{ view: "analytics", range: "1d" }` when diagnosing 5xx; otherwise `{ view: "overview", range: "1d" }` (EDGE `{ siteId }` or hosting `{ website, environment }`)
7. `read_analytics_security` — `{ range: "1d" }`, reuse `domains` from traffic
8. `read_audit_log` — `website` from identity; EDGE without website → **Data unavailable**; role must be team owner/admin (tool text may say not allowed — do not invent audit rows; do not treat remapped 503 as outage)
9. `read_firewall_activity` — both `platform` and `user` groups; hosting `{ website, environment: "production" }` (or omit env) or EDGE `{ siteId }`

### EDGE vs managed hosting

| `kind` | Analytics targets | Firewall targets | Audit |
|--------|-------------------|------------------|-------|
| `edge` | `{ siteId }` | `{ siteId }` | `website` from identity when known; else **Data unavailable** |
| `hosting` | `{ website, environment }` from identity | `{ website, environment: "production" }` (or omit env; never staging) | `{ website }` from identity |

Do not copy analytics identity `environment` into firewall hosting targets.

## Agent rules (Cursor / text-first clients)

1. Use the **tool text response** as the primary data source.
2. **One pass only** — do not re-call the same tool for the same site. **Exception:** if an analytics tool call times out or returns the retry hint, retry **identical** arguments once, then treat a second failure as data unavailable. Do not change `view` or `range` on that retry.
3. **Never** search the codebase or skill templates for live site facts.
4. If a section lacks concrete facts in tool text, write **"Data unavailable"** — do not invent OK.
5. **Analytics:** Never set `mock: true`. If any analytics tool text starts with `[MOCK DATA]`, flag the report as synthetic.
6. **Domain reuse:** After `read_analytics_traffic`, pass returned `domains` into `read_analytics_security`.
7. **Escalation:** If log tails are empty or inconclusive but analytics show elevated 5xx or blocked traffic, rank analytics-backed causes higher.
8. **Audit + firewall in ranking:** When ranking probable causes, weigh recent audit actions (deploys, cache purges, firewall changes, plugin updates) and high `1h`/`24h` firewall block counts alongside log and analytics evidence. If audit is unavailable due to role, note that limitation and rank without inventing actor history. Do not treat production firewall counts as staging-environment activity without corroboration.
9. Do **not** run write blocks unless the user explicitly opts in after the report.
10. Access 5xx: `read_access_signals` uses `httpStatus` (for example `5xx`). Do not use access `grepString` for status codes.
11. **Slow queries are parallel evidence:** weigh `read_slow_query_signals` lines alongside HTTP/PHP errors, but do not join on `pressidium_rid` — rids stay on slow-query text for manual operator use; this slice does not change access format. EDGE still runs `read_slow_query_signals` (same `mysqlslow` inline call as hosting). An empty tail means no slow queries in today's file — report that as-is; it is not an error and not proof the database is healthy.

## Report template

```
# Reliability Engineer — {website or siteId} — {date}

## Incident summary
{one paragraph}

## Errors (PHP / error log)
{from read_error_signals}

## Access / HTTP errors
{from read_access_signals — snapshot from the httpStatus-filtered call}

## Slow queries
{from read_slow_query_signals — or no slow queries in today's file if Loaded 0}

## Recent jobs
{from read_job_history}

## Traffic & security (analytics, 1d)
{from read_analytics_traffic + read_analytics_security}

## Audit log
{from read_audit_log — recent actions, actors, timestamps; or Data unavailable if role cannot read}

## Firewall activity
{from read_firewall_activity — platform + user groups, rule ids, block counts. If identity is staging, the section must say **production**. Do not treat production counts as staging-environment activity without corroboration from another staging-scoped signal.}

## Probable-cause ranking
1. ...
2. ...

## Priority actions
1. ...

## v1 limitations
- Uptime probes and login-attempt monitoring remain out of scope (Phase 3+).
- Firewall rule CRUD (IP/country allow/deny management) is not available via MCP (backlog priority 11).
```

**Still out of scope in v1:** Firewall rule CRUD, automatic remediation writes.

Do not run write blocks in default Reliability Engineer run.
