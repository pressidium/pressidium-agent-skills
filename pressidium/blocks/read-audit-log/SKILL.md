---
name: pressidium-read-audit-log
description: Pressidium building block — team audit log (recent platform actions by user). Use for Reliability Engineer who-changed-what RCA; read-only.
---

# read_audit_log

**Permission:** read (auto-run OK)

## When to use

- Reliability Engineer — correlate recent deploys, cache purges, firewall changes, plugin actions with incident timing
- Incident triage when operator activity may explain a sudden change

## Steps

1. If target unknown, run the `read_site_identity` block first.
2. Resolve `website` from identity when available:
   - **Managed hosting:** use `website` from identity.
   - **EDGE:** use `website` from identity when present; if identity has no website name, skip the tool call and return **Data unavailable** for this section (audit list is team-scoped and filtered by website).
3. Call `pressidium_audit_logs` with `{ website, pageSize: 25 }` when `website` is known. Optional filters (`email`, `action`, `page`) only when the caller requests them.
4. **Role:** audit logs require **team owner** or **team administrator**. If the tool returns an error about role / not allowed (including a remapped Dashboard-API 503), write **Data unavailable — current role cannot read audit logs** (or the tool text) — do not invent entries and do not treat it as a transient outage.
5. Parse the tool **text** summary first (timestamp, `action`, `userEmail`, `website`, `environment` when present); forward `structuredContent` when the composite needs payload detail.
6. Empty list is a successful response — write **no recent audit entries** (or equivalent from tool text), not **Data unavailable**.
7. Return the snapshot to the calling composite; do not mutate the site:

```json
{
  "website": "acorn2clone",
  "pageSize": 25,
  "logs": { "<audit rows>": "..." }
}
```

**Read-only** — never implies or triggers writes.
