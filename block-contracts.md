# Pressidium skill block contracts (Skill library)

Parent spec: [skill library design](../superpowers/specs/2026-06-23-pressidium-skill-library-design.md)

All blocks return a JSON object in the agent's working memory before composites render prose.

**Agent consumption:** In Cursor Agent, read tool **text** first. Only rely on structured fields when your MCP client exposes `structuredContent` in the tool result visible to the model.

## read_site_identity
- **Permission:** read
- **Tools:** `pressidium_whoami`, `pressidium_list_sites` (optional), `pressidium_get_site`
- **Input:** `{ website?: string, siteId?: string|number, environment?: "production"|"staging" }`
- **Snapshot:** `{ user, target, siteSummary }`

## read_staging_state
- **Permission:** read
- **Tools:** `pressidium_get_site`
- **Input:** `{ website: string }` (hosting; from `read_site_identity`)
- **Snapshot:** `{ website, staging: "present"|"absent"|"unknown", pressidiumDomainName? }`
- **Agent note:** Prefer tool text cues `staging: present (…)` / `staging: absent` (Approach 1′). No new MCP tool.

## read_wordpress_state
- **Permission:** read
- **Tools:** `pressidium_wordpress_state`
- **Snapshot:** `{ target, snapshot }` (see `WordPressStateSnapshot` in `src/util/wordpress-snapshot.ts`)
- **Runtime:** Hosting and EDGE snapshots include `wpVersion` and `phpVersion` (`string | null`). EDGE values come from `wpCore.wpVersion` and `php.phpVersion` on `GET /site/{id}/wordpress`. EDGE `get_site` still has no WP/PHP.
- **`collectedAt`:** Hosting snapshots set this at normalization time (`GET /website/{name}` has no API timestamp). EDGE snapshots use API `collectedAt` from `GET /site/{id}/wordpress` when present, otherwise normalization time.

## read_backup_state
- **Permission:** read
- **Tools:** `pressidium_backups` action `list`
- **Snapshot:** `{ backups[], latestBackup?, hoursSinceLatest? }`
- **Timestamps:** Sort/filter by `createdOn`; report `createdOnHuman` to users.

## read_plugin_state
- **Permission:** read
- **Tools:** `pressidium_plugins` action `list` (site or account scope)
- **Snapshot:** `{ plugins, vulnerableCount, updatesCount, vulnerablePlugins, updatablePlugins }`
- **Reuse:** Skip when `read_wordpress_state` already ran for the same site — use `snapshot.pluginsSummary` instead.

## read_wp_core_state
- **Permission:** read
- **Tools:** `pressidium_wordpress_core` action `inspect` (aliases `check`, `list`)
- **Snapshot:** `{ coreVersions, updateAvailable? }`

## read_cache_state
- **Permission:** read
- **Tools:** `pressidium_cache_settings` get page+object
- **Snapshot:** `{ pageCache, objectCache, cdnEnabled: true }` on hosting (CDN included by default; do not call a CDN tool)

## read_cert_dns_state
- **Permission:** read
- **Tools:** `pressidium_domains` list (site SSL from list text: `https: on|off`, `force-https: on|off`, `cert:`); `pressidium_certificates` list (account-wide fleet expiry — no site `target`; ids + `cert: valid|expired`)
- **Snapshot:** `{ domains[], certificates[] }`
- **Timestamps:** Use `importedOnHuman` (and other `*Human` companions) when present. Certificate expiry is `expiration` (human string, not epoch).

## read_error_signals
- **Permission:** read
- **Tools:** `pressidium_logs` type `error`, format `inline`, `lastSize` (last N lines), optional `grepString` (message `|`-OR tokens), optional `includeNoise`
- **Snapshot:** `{ lines[], truncated, grepString? }`

## read_access_signals
- **Permission:** read
- **Tools:** `pressidium_logs` type `access`, format `inline`, `lastSize`, `httpStatus` (e.g. `5xx`) — do not use `grepString` for status codes
- **Snapshot:** `{ entries[], lines[], errorCount, truncated }`

## read_slow_query_signals
- **Permission:** read
- **Tools:** `pressidium_logs` type `mysqlslow`, format `inline`, `lastSize` (last N lines). Hosting and EDGE. Add `grepString` only when the user named a table or SQL snippet
- **Snapshot:** `{ lines[], truncated, grepString? }`

## read_job_history
- **Permission:** read
- **Tools:** `pressidium_operations` action `notifications` with `resource` + `since` (24h ago epoch)
- **Snapshot:** `{ items[] }`
- **Timestamps:** Report `timestampHuman`; keep `timestamp` for filtering and age math.

## read_team_capacity
- **Permission:** read
- **Tools:** `pressidium_team_usage`
- **Snapshot:** `{ usage, subscription?, availablePlans? }`

## read_analytics_overview
- **Permission:** read
- **Tools:** `pressidium_analytics_overview`
- **Input:** `{ target: { siteId } | { website, environment }, range?, domains?, mock? }` — EDGE or managed hosting; known-good ranges: `1d`, `7d`, `14d`
- **Timeout:** if the call times out, retry identical arguments once, then report data unavailable
- **Snapshot:** forward tool `structuredContent` (`{ metricGroups, range, domains, mock, data }`); parse text first — text may include staging-omit or other-env-default caveats; text summary is primary for digests

## read_analytics_traffic
- **Permission:** read
- **Tools:** `pressidium_analytics_traffic`
- **Input:** `{ target: { siteId } | { website, environment }, view?: "overview"|"analytics", range?, domains?, mock? }` — EDGE or managed hosting
- **Timeout:** if the call times out, retry identical arguments once, then report data unavailable
- **Snapshot:** forward tool `structuredContent`; parse text first — text may include staging-omit or other-env-default caveats; composite may label as `{ traffic: data }`

## read_analytics_performance
- **Permission:** read
- **Tools:** `pressidium_analytics_performance`
- **Input:** `{ target: { siteId } | { website, environment }, view?: "overview"|"analytics"|"cache", range?, domains?, mock? }` — EDGE or managed hosting; `summary` aliases `overview`
- **Timeout:** if the call times out, retry identical arguments once, then report data unavailable
- **Snapshot:** forward tool `structuredContent`; parse text first — text may include staging-omit or other-env-default caveats; composite may label as `{ performance: data }`

## read_analytics_security
- **Permission:** read
- **Tools:** `pressidium_analytics_security` (overview view only)
- **Input:** `{ target: { siteId } | { website, environment }, range?, domains?, mock? }` — EDGE or managed hosting
- **Timeout:** if the call times out, retry identical arguments once, then report data unavailable
- **Snapshot:** forward tool `structuredContent`; parse text first — text may include staging-omit or other-env-default caveats; composite may label as `{ security: data }`

## read_origin_reports
- **Permission:** read
- **Tools:** `pressidium_reports` with `reportType: "origin_visits"` and `reportType: "origin_bandwidth"`
- **Input:** `{ target: { website, environment } }` — managed hosting production only (staging returns soft-unsupported)
- **Snapshot:** `{ target, originVisits: { status, series[] }, originBandwidth: { status, series[] } }`

## read_disk_trends
- **Permission:** read
- **Tools:** `pressidium_reports` with `reportType: "disk"`
- **Input:** `{ target: { website, environment } }` — managed hosting production only
- **Snapshot:** `{ target, disk: { status, series[] } }`

## read_edge_usage
- **Permission:** read
- **Tools:** `pressidium_reports` with `reportType: "edge_usage_requests"` and `reportType: "edge_usage_data_transfer"`
- **Input:** `{ target: { siteId } }` — EDGE only
- **Snapshot:** `{ target, edgeUsageRequests: { status, series[] }, edgeUsageDataTransfer: { status, series[] } }`
- **Note:** Static variants (`edge_usage_*_static`) are optional — only when the user asks.

## read_perfaudit_history
- **Permission:** read
- **Tools:** `pressidium_perfaudit` with `action: "history"`, default `device: "mobile"`
- **Input:** `{ target: { siteId } | { website }, device?: "mobile"|"desktop", url? }` — EDGE or managed hosting (hosting paths omit `environment`)
- **Snapshot:** `{ target, action, device, history }` — forward tool `structuredContent` (`tests[]`); text summary is primary for composites
- **Note:** Empty history is success — composite writes **Data unavailable** for the section, not invented OK metrics. Optional drill-down: `action: "summary"` with `testId` from history (no separate block)

## read_audit_log
- **Permission:** read
- **Tools:** `pressidium_audit_logs`
- **Input:** `{ website?, email?, action?, page?, pageSize? }` — team-scoped; Reliability Engineer passes `website` from identity when known; default `pageSize: 25`
- **Snapshot:** `{ website?, pageSize, logs[] }` — forward tool `structuredContent`; text summary is primary for composites
- **Note:** No `since` filter (swagger has none). EDGE without identity website → skip tool call; composite writes **Data unavailable** for audit section. Empty list is success — “no recent audit entries”, not **Data unavailable**.

## read_firewall_activity
- **Permission:** read
- **Tools:** `pressidium_firewall_activity` — **two calls** per target: `group: "platform"` then `group: "user"`
- **Input:** `{ target: { siteId } | { website, environment: "production" } }` — EDGE or managed hosting. Hosting: always production (or omit env); do not pass `staging`.
- **Snapshot:** `{ target, platform: { rules[] }, user: { rules[] } }` — forward tool `structuredContent` per group; text summary is primary for composites
- **Note:** Empty list for a group is success. Does not manage firewall rules (CRUD remains backlog priority 11). When the report is hosting staging, the firewall section must say **production**. Do not treat those counts as staging-environment activity without corroboration. If the locked sentence appears, quote it; do not invent rule counts.

## read_ssh_state
- **Permission:** read
- **Tools:** `pressidium_website_ssh` `action: status`
- **Input:** `{ website: string, environment?: "production"|"staging" }` — hosting only
- **Snapshot:** `{ target, sshService }` — prefer tool **text** (`enabled=`, `host=`, `port=`, `username=`, `ssh -p` line)
- **Note:** EDGE → stop. Missing `sshService` is `enabled=unknown`, not an error.

## Composite: site_guardian

Parent skill: `.cursor/skills/pressidium/composites/site-guardian/SKILL.md`

**Permission:** read-only (default run)

**Block order:**

1. `read_site_identity`
2. `read_backup_state`
3. `read_plugin_state` (site scope)
4. `read_cert_dns_state`
5. `read_error_signals`
6. Analytics (EDGE and hosting): `read_analytics_traffic` (`view: overview`, `range: 1d`) → `read_analytics_security` (`range: 1d`, reuse `domains`)
7. `read_audit_log` — `website` from identity; EDGE without website → **Data unavailable**
8. `read_firewall_activity` — both `platform` and `user` groups; hosting `{ website, environment: "production" }` (or omit env) or EDGE `{ siteId }`
9. `read_team_capacity`

**Analytics branch:** Run step 6 for both `siteSummary.kind === 'edge'` (`{ siteId }`) and `hosting` (`{ website, environment }` from identity).

**Analytics inputs:** Never `mock: true`. Known-good ranges: `1d`, `7d`, `14d` (Site Guardian uses `1d`).

**Digest:** Includes **Audit log** and **Firewall activity** sections. Elevated firewall block counts in tool text may also tip **ATTENTION NEEDED** — cite **production** firewall tool text, not staging. Do not treat those counts as staging-environment activity without corroboration. Audit role denial alone does not escalate.

## Composite: performance_review

Parent skill: `.cursor/skills/pressidium/composites/performance-review/SKILL.md`

**Permission:** read-only (default run)

**Block order:**

1. `read_site_identity`
2. `read_cache_state`
3. `read_analytics_performance` (`view: overview`, `range: 7d`; optional `view: cache`)
4. `read_perfaudit_history` — `{ device: "mobile" }` (EDGE `{ siteId }` or hosting `{ website }`)
5. `read_slow_query_signals` (`type: mysqlslow`, `format: inline`, `lastSize: 100`)
6. `read_analytics_traffic` (`view: overview`, `range: 7d`, reuse `domains`)
7. `read_wordpress_state` **or** `read_plugin_state` (one only)
8. `read_wp_core_state` (skip if redundant)

**Analytics branch:** EDGE and hosting (same as Site Guardian kind table).

**Report:** **PerfAudit (mobile)** section after Performance analytics; **Slow queries** section after PerfAudit (mobile), before Traffic context. Default run stays read-only — `act_perfaudit_test` only after explicit user opt-in.

## Composite: reliability_engineer

Parent skill: `.cursor/skills/pressidium/composites/reliability-engineer/SKILL.md`

**Permission:** read-only (default run)

**Block order:**

1. `read_site_identity`
2. `read_error_signals`
3. `read_access_signals`
4. `read_slow_query_signals` (`type: mysqlslow`, `format: inline`, `lastSize: 100`)
5. `read_job_history`
6. `read_analytics_traffic` (prefer `view: analytics`, `range: 1d`)
7. `read_analytics_security` (`range: 1d`, reuse `domains`)
8. `read_audit_log` — `website` from identity; EDGE without website → **Data unavailable** for audit section
9. `read_firewall_activity` — both `platform` and `user` groups; hosting `{ website, environment: "production" }` (or omit env) or EDGE `{ siteId }`

**Report:** **Slow queries** section between Access / HTTP errors and Recent jobs; **Audit log** and **Firewall activity** sections feed probable-cause ranking. Phase 3+ limitations remain (uptime, login attempts). Firewall rule CRUD is still out of scope (backlog priority 11).

## Composite: growth_monitor

Parent skill: `.cursor/skills/pressidium/composites/growth-monitor/SKILL.md`

**Permission:** read-only (default run)

**Block order:**

1. `read_site_identity`
2. `read_analytics_traffic` (`view: overview`, default `range: 7d`; `14d` if user requests longer)
3. `read_analytics_overview` (same range, reuse `domains`)
4. `read_team_capacity`
5. By `kind`: **hosting** → `read_origin_reports` then `read_disk_trends`; **edge** → `read_edge_usage`

**v1 gap:** GSC/GA4 (Phase 4). CDN and hosting-edge `reportType`s on `pressidium_reports` are tool-optional (not wired into Growth Monitor by default). For **Plan** (upgrade/stay/downgrade) and **Fleet** (room/tight/full) advice, use [Capacity Advisor](../superpowers/specs/2026-07-28-capacity-advisor-design.md) — Growth Monitor keeps traffic + origin/disk headroom context only.

## Composite: staging_validator

Parent skill: `.cursor/skills/pressidium/composites/staging-validator/SKILL.md`

**Permission:** read-only (default run); confirm-gated deploy after report + opt-in

**Block order:**

1. `read_site_identity` — hosting only; EDGE unsupported
2. `read_staging_state` — hard-stop if absent → NO-GO (+ create hint, opt-in only)
3. `read_error_signals` (`environment: staging`)
4. `read_job_history`
5. `read_wordpress_state` **or** `read_plugin_state` (staging, one only)
6. Same inventory block for production
7. `read_wp_core_state` (staging, then production)
8. `read_perfaudit_history` (`device: mobile`)

**Verdict:** GO | CAUTION | NO-GO (agent heuristics from tool text). Inventory skew → at least CAUTION; never auto-NO-GO from skew alone.

**Post-report promote (Phase 2):**
- Preconditions: Validator report in conversation + explicit user opt-in to promote.
- **GO** or **CAUTION** → `act_stage_changes` / `pressidium_staging` `deploy_to_production` (explain → unconfirmed call → confirm only after user yes). On CAUTION, restate caution reasons (incl. skew) first. Prefer `deploydb: false` unless user asks otherwise.
- **NO-GO** → refuse deploy; no confirm walk.
- Never auto-confirm; never delete staging / auto-create from this composite.

**Remaining gaps:** No themes in the diff; EDGE unsupported; no NO-GO deploy override.

## Composite: capacity_advisor

Parent skill: `.cursor/skills/pressidium/composites/capacity-advisor/SKILL.md`

**Permission:** read-only (default run)

**Block order:**

**Account-only:**

1. `read_team_capacity`

**With target:**

1. `read_site_identity`
2. `read_team_capacity`
3. By `kind`: **hosting** → `read_origin_reports` then `read_disk_trends`; **edge** → `read_edge_usage`

**Verdicts:** Plan `UPGRADE` | `STAY` | `DOWNGRADE`; Fleet `ROOM` | `TIGHT` | `FULL` (agent heuristics from tool text)

**v1 gap:** No billing/subscription writes; no analytics (Growth Monitor); GSC/GA4 (Phase 4). Optional `act_refresh_disk_usage` after user opt-in only.

## await_job loop (wait: true)

If wait text has `jobStatus=running`, call `pressidium_operations` `action: await_job` with that `jobId`.
The first `wait: true` does **not** count. Ceiling: **10** `await_job` calls.
Stop on `success` / `error`. Stop on `unknown` unless the text says the result could not be confirmed (`probeFailed` / unconfirmed) — then **one** extra `await_job`.
After 10 `running` results: the job is still running server-side, **not failed**. Tell the user `jobId` and `pressidium_operations action=await_job jobId=<id>`.
If those 10 were unconfirmed live-job probes, say the Dashboard API may be unhealthy and status is unconfirmed.

## act_create_backup
- **Permission:** low-risk write
- **Tools:** `pressidium_backups` action `create`, `wait: true`
- **Snapshot:** `{ submitted, jobId, jobStatus, backupId? }`
- **Agent note:** Parse `jobStatus=` from tool **text** (not the headline). After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`. Proceed to updates only if `jobStatus` is `success` after the wait loop. On success parse `backupId=` / `createdOn=`. If `isError` and text contains `jobStatus=success`, do **not** create again.

## act_purge_cache
- **Permission:** low-risk write
- **Tools:** `pressidium_purge_cache` scope `all`, `wait: true`
- **Snapshot:** `{ submitted, jobId, jobStatus }`
- **Agent note:** Parse `jobStatus=` from tool **text** (not the headline). Do not treat “submitted” as success. After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.

## act_enable_ssh
- **Permission:** low-risk write
- **Tools:** `pressidium_ssh_keys` `list`/`add`, `pressidium_website_ssh` `status`/`enable` with `wait: true`
- **Rule:** List keys first at `page: 1` (default `page_size` 25). Parse **text** only. Confirmed empty is entire text exactly `Loaded 0 SSH key(s).` on initial page 1 → `ssh-keygen -t rsa` (Linux/macOS/Windows OpenSSH), `.pub` only, **do not enable**. `Loaded 0 SSH key(s). (N skipped)` is uncertainty — do not treat as empty; skipped is sticky across pages. Follow `nextPage=` while displayed count is 0. `SSH key list unavailable (unrecognized response).` or `isError` → stop, no keygen-as-empty. After keys exist, skip enable when already `enabled=on` with an `ssh -p` line. Parse `jobStatus=` from text. If `jobStatus=success` (including `job=none` / no job id), print status tokens and the `ssh` command (`ssh -p` when present), then stop without polling — do not call `await_job`. If `jobStatus=running` **and** a `jobId` is present, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`. After the loop, if still `running`, tell the user `jobId` / `await_job`; on `success`, print the same status tokens and the `ssh` command then stop. If `running` with no `jobId`, do not call `await_job`; report the wait status; suggest `read-ssh-state` if tokens are missing. `jobStatus=success` includes a finished wait with no job id (`job=none`).
- **Snapshot:** `{ submitted?, jobId?, jobStatus?, sshService? }`

## act_disable_ssh
- **Permission:** low-risk write
- **Tools:** `pressidium_website_ssh` `action: disable`, `wait: true`
- **Snapshot:** `{ submitted, jobId, jobStatus, sshService? }`
- **Agent note:** Parse `jobStatus=` from tool **text**. If `jobStatus=success` (including `job=none` / no job id), report any `enabled=` / host / port / username tokens, then stop without polling — do not call `await_job`. If `jobStatus=running` **and** a `jobId` is present, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`. After the loop, if still `running`, tell the user `jobId` / `await_job`; on `success`, report the same SSH-off tokens then stop. If `running` with no `jobId`, do not call `await_job`; report the wait status; suggest `read-ssh-state` if tokens are missing. `jobStatus=success` includes a finished wait with no job id (`job=none`). No `confirm`. Hosting only.

## act_stage_changes
- **Permission:** confirm-gated
- **Tools:** `pressidium_staging`, `pressidium_clone_website`
- **Rule:** Never pass `confirm: true` unless the user explicitly approved. Clone and migrate: prefer `wait: false` then `pressidium_operations` `await_job`; `wait` remains. After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.

## act_apply_updates
- **Permission:** confirm-gated
- **Tools:** `pressidium_plugins` update, `pressidium_wordpress_core` update
- **Rule:** Recommend `act_create_backup` first; never auto-update without approval. After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.

## act_refresh_wp_inventory
- **Permission:** low-risk write
- **Tools:** `pressidium_website_refresh` `action: "scan"`, `wait: true`
- **Snapshot:** `{ submitted, jobId, jobStatus, action: "scan" }`
- **Notes:** Hosting only. Optional Maintenance Advisor pre-step. After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.

## act_refresh_disk_usage
- **Permission:** low-risk write
- **Tools:** `pressidium_website_refresh` `action: "disk_usage"`, `wait: true`
- **Snapshot:** `{ submitted, jobId, jobStatus, action: "disk_usage" }`
- **Notes:** Hosting only. Optional Maintenance Advisor pre-step. After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.

## act_perfaudit_test
- **Permission:** low-risk write
- **Tools:** `pressidium_perfaudit` with `action: "start"`, default `device: "mobile"`, `wait: true`
- **Snapshot:** `{ submitted, jobId, jobStatus, action: "start", device }`
- **Notes:** EDGE or hosting. Do **not** run in default Performance Review — opt-in only after the report. Re-read history via `read_perfaudit_history` when fresh metrics are needed. After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.
