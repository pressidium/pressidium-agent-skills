---
name: pressidium-act-stage-changes
description: Pressidium building block — staging create/deploy/clone. HIGH-RISK — confirm required.
---

# act_stage_changes

NEVER call with `confirm: true` unless user explicitly approved the exact action.

1. Explain action (create staging, deploy to production, clone).
2. Call staging tool once without confirm → show guidance.
3. Re-call only after user says yes.

For clone and migrate, prefer `wait: false` then `pressidium_operations` `action: await_job`. Do not omit `wait`.
After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.
