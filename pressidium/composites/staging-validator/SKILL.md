---
name: pressidium-staging-validator
description: Pressidium Operations composite — pre-deploy go/no-go checklist for managed hosting staging. Invoke with "Run Staging Validator on {website}". Hosting only — EDGE unsupported.
---

# Staging Validator

**Schedule intent:** pre-deploy / on-demand  
**Tier:** Operations (v1 + Phase 1–2 deepenings)  
**Orchestrates:** site identity + staging state + staging error signals + job history + dual-env WordPress/plugin + WP core diff + mobile PerfAudit history

## Block order

**Invoke:** `Run Staging Validator on {website}`

1. `read_site_identity` — require managed hosting (`siteSummary.kind === hosting`). If `kind === edge`, stop: Staging Validator is **unsupported** for EDGE (no create/deploy staging model).
2. `read_staging_state` — if staging **absent**, stop with **NO-GO**. Do **not** call further staging-targeted blocks. Priority actions may recommend `act_stage_changes` (`create`) only after **explicit user opt-in** — this composite **never creates** staging.
3. If staging **present**, continue with `read_error_signals` — `{ website, environment: "staging" }`.
4. `read_job_history` — recent jobs for the website; highlight failed staging/create/deploy jobs when visible in tool text.
5. Staging inventory: `read_wordpress_state` **or** `read_plugin_state` (site scope, `environment: "staging"`) — **one pass only**, not both.
6. Production inventory: the **same** chosen block with `environment: "production"` (second call allowed — different environment; do not re-call the same site+environment).
7. `read_wp_core_state` — staging target; then production target.
8. `read_perfaudit_history` — `{ device: "mobile", website }` (website-level). If unavailable, write **Data unavailable**.
9. Emit report with Verdict + Checklist + **Prod ↔ staging diff**. Optional after the report: user may opt in to `act_perfaudit_test`, then re-read history. **Gated deploy** only under Post-report promote (below) — never during the default read-only checklist pass.

### Hosting only — EDGE unsupported

After `read_site_identity`, inspect `siteSummary.kind`:

| `kind` | Action |
|--------|--------|
| `hosting` | Continue with `read_staging_state` |
| `edge` | Stop with unsupported message — Staging Validator does not apply to EDGE sites |

Identity line: hosting `{website} · {environment} · hosting`.

## Agent rules (Cursor / text-first clients)

1. Use the **tool text response** as the primary data source.
2. **One pass only** per site+environment — do not re-call the same tool for the same site/environment in one run. Dual-environment inventory/core reads are two passes by design.
3. **Never** search the codebase or skill templates for live site facts.
4. If a section lacks concrete facts in tool text, write **"Data unavailable"** — do not invent OK or invent inventory parity.
5. Prefer `read_wordpress_state` **or** `read_plugin_state` — not both; use the same choice for staging and production.
6. **Never set `mock: true`.** If tool text starts with `[MOCK DATA]`, mark the report synthetic.
7. **Read-only default:** Do **not** run write blocks (`act_stage_changes`, `act_perfaudit_test`, …) during the checklist pass. Writes only after the report when the user explicitly opts in.
8. **Post-report promote (Phase 2):** After a Staging Validator report in this conversation, if the user explicitly opts in to promote / deploy to production:
   - **GO** or **CAUTION** → walk `act_stage_changes` → `pressidium_staging` `action: "deploy_to_production"` using the confirm pattern: explain → call **once without** `confirm` → show guidance → re-call with confirm **only after** the user says yes. Never auto-confirm.
   - On **CAUTION**, **restate** why (including any prod↔staging skew or other caution reasons) **before** starting the confirm walk.
   - **NO-GO** → **refuse** deploy; point at Priority actions; do **not** start the confirm walk and do **not** pass confirm for deploy.
9. **Deploy defaults:** Prefer API-safe defaults — `deploydb: false` unless the user explicitly asks to deploy the database; `deployfiles` per tool defaults. Optional `wait` / job follow-up via existing patterns. Do **not** delete staging or auto-create from this composite.
10. **Create / PerfAudit opt-in (unchanged):** Staging absent → Priority actions may recommend `act_stage_changes` (`create`) only after explicit opt-in. `act_perfaudit_test` only after explicit opt-in. Create/PerfAudit confirm rules still follow their blocks.

## Verdict heuristics

Agent-applied from tool text (not a hard-coded MCP scoring service):

| Verdict | When |
|---------|------|
| **NO-GO** | Staging absent; or clear fatal/blocking errors on staging; or failed recent staging/deploy jobs that leave staging unhealthy for promote. Do **not** auto-NO-GO from inventory skew alone. |
| **CAUTION** | Non-fatal errors, aging/missing PerfAudit, pending WP/plugin updates, thin/unavailable signals, **or any prod↔staging plugins/core skew** (version, active, or present-only-on-one-side), or missing inventory/core text that gaps the promote story |
| **GO** | Staging present; no blocking errors/failed jobs; **no** plugins/core skew; remaining signals clear or explicitly "Data unavailable" without inventing OK. |

## Report template

```
# Staging Validator — {website} — {date}

## Verdict
{GO | CAUTION | NO-GO} — {one-line rationale}

## Staging identity
{present/absent + domain from read_staging_state; or EDGE unsupported}

## Error signals (staging)
{lines or Data unavailable.}

## Recent jobs
...

## WordPress / plugin risk (staging)
...

## Prod ↔ staging diff
{core versions; plugin skew (version/active/side-only); or Data unavailable}

## PerfAudit (mobile)
{history or Data unavailable}

## Checklist
- [ ] Staging environment present
- [ ] No blocking staging errors
- [ ] No failed staging/deploy jobs blocking promote
- [ ] WP/plugin risk reviewed
- [ ] Prod ↔ staging inventory (plugins + core) reviewed
- [ ] PerfAudit reviewed (or unavailable noted)

## Priority actions
1. ...
(Recommend act_stage_changes create only if staging absent — after user opt-in.
 Recommend act_perfaudit_test only after opt-in.
 After GO or CAUTION + explicit promote opt-in: walk act_stage_changes deploy_to_production (confirm pattern; restate CAUTION reasons first).
 On NO-GO: refuse deploy; do not start confirm walk.)

## Limitations
- Confirm-gated deploy only after report + explicit opt-in on GO/CAUTION; never on NO-GO; no themes in the diff; EDGE unsupported.
- PerfAudit may be unavailable on some Dashboard environments.
- Staging Validator does not replace Performance Review or Reliability Engineer.
```

**Still out of scope:** Auto-create staging, EDGE, themes in the diff, MCP-computed diff, deploy override on NO-GO, fleet PerfAudit.

Default Staging Validator run is **read-only**. After the report, create/PerfAudit only on explicit opt-in; **deploy** only on **GO** or **CAUTION** + explicit promote opt-in via `act_stage_changes` confirm pattern; **refuse** deploy on **NO-GO**.
