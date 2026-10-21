---
name: pressidium-maintenance-advisor
description: Pressidium Starter composite — weekly safe update plan without auto-applying. Invoke with "Run Maintenance Advisor on {website}".
---

# Maintenance Advisor

**Optional freshen pre-step (not on every run):** When the user asks to freshen inventory / refresh plugins and themes / recalculate disk, or when you recommend a refresh and the user agrees, run the relevant block(s) **before** the reads below:

- `act_refresh_wp_inventory` — plugin/theme scan
- `act_refresh_disk_usage` — disk usage recalculation

Never auto-refresh on every Maintenance Advisor run.

**Blocks (in order):**
1. `read_site_identity`
2. `read_wordpress_state` — includes `snapshot.pluginsSummary` and full `snapshot.plugins` (no separate plugin block needed for site scope)
3. `read_wp_core_state`
4. `read_backup_state`

**Optional write (user must opt in):** `act_create_backup` only — never `act_apply_updates` without explicit separate approval.

## Report template

```
# Maintenance Advisor — {website}

## Executive summary
One paragraph: safe to defer vs update soon.

## WordPress core
...

## Plugins — vulnerable (fix first)
...

## Plugins — updates available
...

## Recommended sequence
1. Create backup (offer to run act_create_backup if user agrees)
2. Update plugin X
3. ...

## Not recommended now
...
```

Phase 1: plan only — no automatic updates.
