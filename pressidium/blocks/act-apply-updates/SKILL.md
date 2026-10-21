---
name: pressidium-act-apply-updates
description: Pressidium building block — plugin or core updates. HIGH-RISK — confirm required; run act_create_backup first.
---

# act_apply_updates

**Permission:** confirm-gated

1. Recommend `act_create_backup` first.
2. Never pass `confirm` on destructive paths; plugin/core updates use `wait: true`.
3. For plugins: `pressidium_plugins` `action: "update"` with explicit plugin list (not `all` without approval).
4. After `wait: true` (plugin or core), follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.
