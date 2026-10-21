---
name: pressidium-backup-guardian
description: Pressidium Starter composite — daily backup verification report. Invoke with "Run Backup Guardian on {website}".
---

# Backup Guardian

**Schedule intent:** daily  
**Blocks:** `read_site_identity` → `read_backup_state`  
**Writes:** none in Phase 1

## Report template

```
# Backup Guardian — {website} ({environment})

## Status
{OK | WARNING | CRITICAL}

## Latest backup
- ID: ...
- Type: ...
- Created: ... ({hoursSinceLatest}h ago)

## Findings
- {bullet list}

## Recommended actions
- {only read-only recommendations in Phase 1}
```

## Rules

1. Run read blocks only.
2. Use `read_backup_state` snapshot status. WARNING if latest backup older than 24 hours **or** finding `backup state could not be verified` (unrecognized list or all rows skipped; age unavailable). Not OK.
3. CRITICAL only when list text is `No backups found for {website}.` (recognized empty). Unverified is not CRITICAL.
