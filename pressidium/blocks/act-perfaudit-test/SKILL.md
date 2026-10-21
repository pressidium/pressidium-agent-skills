---
name: pressidium-act-perfaudit-test
description: Pressidium building block — start an on-demand PerfAudit test for managed hosting websites or EDGE sites. Low-risk write; use only after Performance Review when the user opts in.
---

# act_perfaudit_test

Call `pressidium_perfaudit` with `{ target, action: "start", device: "mobile", wait: true }` (EDGE `{ siteId }` or hosting `{ website }`).

Use `device: "desktop"` only when the user explicitly requests a desktop test. Report `jobStatus` / wait outcome from the tool text.
After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.

**Low-risk write** — do not run in default Performance Review runs. Invoke only after the user opts in (e.g. from Priority actions).

After success, re-read history via `read_perfaudit_history` if the workflow needs fresh metrics.
