---
name: pressidium-act-create-backup
description: Pressidium building block — create instant backup (low-risk write). Use only when user or Maintenance Advisor requested a safety backup.
---

# act_create_backup

**Permission:** low-risk write

## Steps

1. Call `pressidium_backups` `action: "create"`, `comment` describing reason, `wait: true`.
2. Parse `jobStatus=` from tool **text** (not the headline).
   After `wait: true`, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`.
3. If `isError` and the text contains `jobStatus=success`, do **not** create again; report the list failure and use `job=` / a later list.
4. After the loop, proceed to updates **only** if `jobStatus` is `success`. If `jobStatus` is not `success` (`error`, `unknown`, or still `running`), do not proceed to updates. `running` is not a failure, but it is not a completed backup; tell the user `jobId` and to continue with `await_job` when it is still running.
5. On success, parse `backupId=` / `createdOn=` from text when present.
6. Return `{ submitted, jobId, jobStatus, backupId? }`.
