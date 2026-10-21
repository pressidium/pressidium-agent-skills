---
name: pressidium-read-backup-state
description: Pressidium building block — backup inventory and recency. Use for Backup Guardian and Site Guardian.
---

# read_backup_state

**Permission:** read

## Steps

1. Call `pressidium_backups` with `action: "list"`, `website`, `environment`. Do not pass `detail: "full"` for Guardian.
2. Parse **text** first (Cursor does not show structuredContent).
3. If text is exactly `Backup list unavailable (unrecognized response).` OR `Loaded 0 backup(s) for {website}. (N skipped)`: snapshot `{ listStatus: "unavailable", hoursSinceLatest: null, backups: [], latestBackup: null, alertIfOlderThanHours: 24 }`. Guardian status **WARNING** with finding `backup state could not be verified`. Not OK. Not CRITICAL. Age unavailable.
4. If text is `No backups found for {website}.`: established empty. Guardian **CRITICAL**. `hoursSinceLatest: null`.
5. If text contains one or more `id=` lines: the first data line is newest. Parse `createdOn=` with the same rules as list (JSON-quoted digits; `Number`; values `>= 1000000000000` are milliseconds else seconds). Compute `hoursSinceLatest` from that epoch vs now. WARNING if `hoursSinceLatest > 24`.
6. Structured `createdOn` / `latest` may be used when the harness exposes structuredContent; it is not required. Do not require `metadata`.
7. Snapshot: `{ backups, latestBackup, hoursSinceLatest, listStatus?, alertIfOlderThanHours: 24 }`.
