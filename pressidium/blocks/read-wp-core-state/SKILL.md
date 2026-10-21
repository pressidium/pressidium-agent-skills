---
name: pressidium-read-wp-core-state
description: Pressidium building block — WordPress core version and available upgrades. Use for Maintenance Advisor.
---

# read_wp_core_state

**Permission:** read

## Steps

1. Call `pressidium_wordpress_core` `action: "inspect"` (aliases `check`, `list`), `scope: "site"`, `target`.
2. Parse inspect **text** for current / default / upgradeable versions. Snapshot `{ data: structured.data }` when structured is visible.
3. If update available, recommend backup before any `act_apply_updates`.
