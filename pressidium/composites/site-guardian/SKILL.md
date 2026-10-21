---
name: pressidium-site-guardian
description: Pressidium Starter composite — daily digest combining backup, plugins, certs, errors, EDGE and hosting traffic/security analytics, audit log, firewall activity, and capacity. Invoke with "Run Site Guardian on {website}" or an EDGE siteId.
---

# Site Guardian

**Schedule intent:** daily  
**Orchestrates:** Backup Guardian + Plugin Auditor (site scope) + cert/DNS + errors + traffic/security analytics (EDGE + hosting) + audit log + firewall activity + team capacity

## Block order

1. `read_site_identity`
2. `read_backup_state` (or invoke Backup Guardian logic)
3. `read_plugin_state` (site) — flag vulns
4. `read_cert_dns_state`
5. `read_error_signals`
6. **Analytics branch** (EDGE and managed hosting — see below):
   - `read_analytics_traffic` — `{ target: { siteId }, view: "overview", range: "1d" }` (EDGE) or `{ target: { website, environment }, view: "overview", range: "1d" }` (hosting)
   - `read_analytics_security` — same target shape, `{ range: "1d", domains: <domains from traffic block> }`
7. `read_audit_log` — `website` from identity; EDGE without website → **Data unavailable**; role must be team owner/admin (tool text may say not allowed — do not invent audit rows; do not treat remapped 503 as outage)
8. `read_firewall_activity` — both `platform` and `user` groups; hosting `{ website, environment: "production" }` (or omit env) or EDGE `{ siteId }`
9. `read_team_capacity`

### Target shape by kind

After `read_site_identity`, inspect `siteSummary.kind`:

| `kind` | Analytics targets | Firewall targets | Audit |
|--------|-------------------|------------------|-------|
| `edge` | `{ siteId }` | `{ siteId }` | `website` from identity when known; else **Data unavailable** |
| `hosting` | `{ website, environment }` from identity | `{ website, environment: "production" }` (or omit env; never staging) | `{ website }` from identity |

Do not copy analytics identity `environment` into firewall hosting targets.

Digest identity line:
- Hosting: `{website} · {environment} · hosting`
- EDGE: `{siteId or website} · edge`

## Agent rules (Cursor / text-first clients)

1. Use the **tool text response** as the primary data source. Cursor Agent often does not surface `structuredContent`.
2. **One pass only** — do not re-call the same tool or call `pressidium_wordpress_state` after `pressidium_plugins` for the same site. **Exception:** if an analytics tool call times out or returns the retry hint, retry **identical** arguments once, then treat a second failure as data unavailable. Do not change `view` or `range` on that retry.
3. **Never** search the codebase or read skill templates for live site facts (domain names, plugin slugs, usage numbers).
4. If a section lacks concrete facts in tool text, write **"Data unavailable"** — do not mark OK from the digest template.
5. Include site identity line per EDGE vs hosting table above.
6. Prefer `pressidium_wordpress_state` **or** `pressidium_plugins` for plugins — not both.
7. **Analytics:** Never set `mock: true`. If any analytics tool text starts with `[MOCK DATA]`, flag the report as synthetic and do not treat metrics as production truth.
8. **Domain reuse:** After `read_analytics_traffic`, pass returned `domains` into `read_analytics_security` to avoid a second site-detail fetch.
9. **Audit + firewall:** One pass only. Call `read_firewall_activity` for **both** `platform` and `user` groups. If audit is unavailable due to role denial, permission, or remapped 503, note that limitation — do not invent audit rows and do not treat remapped 503 as an outage. Do not treat audit role denial alone as **ATTENTION NEEDED**. Elevated firewall block counts in tool text may tip **ATTENTION NEEDED** — cite **production** firewall tool text, not staging.

## Digest template

```
# Site Guardian — {website or siteId} — {date}

## Overall: {ALL CLEAR | ATTENTION NEEDED}

### Backups
{one line from backup block}

### Security (plugins)
{vuln count or OK}

### SSL / domains
{OK or issues}

### Errors (recent log tail)
{line count / sample or none}

### Traffic & security (analytics)
{one-liners from read_analytics_traffic overview + read_analytics_security}

### Audit log
{one-liner from read_audit_log — recent actions or Data unavailable / permission note}

### Firewall activity
{one-liner from read_firewall_activity — platform + user. If identity is staging, the section must say **production**.}

### Capacity
{team usage one-liner if relevant}

## Priority actions
1. ...
```

**Escalation rule:** When the log tail is empty or inconclusive but analytics traffic overview suggests elevated 5xx or blocked traffic, set **ATTENTION NEEDED** and cite analytics tool text. Applies to both EDGE and managed hosting. Elevated firewall block counts in tool text may also tip **ATTENTION NEEDED** — cite **production** firewall tool text, not staging. Do not treat those counts as staging-environment activity without corroboration. Audit role denial or permission limitation alone does **not** escalate; note the limitation and continue.

**Still out of scope:** uptime probes, login-attempt monitoring — note in the report when signals remain inconclusive.

Do not run write blocks in default Site Guardian run.
