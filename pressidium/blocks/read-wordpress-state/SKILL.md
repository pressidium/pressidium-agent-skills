---
name: pressidium-read-wordpress-state
description: Pressidium building block — normalized plugins, themes, disk usage, available updates. Use for Site Guardian, Maintenance Advisor, Plugin Auditor site scope.
---

# read_wordpress_state

**Permission:** read

## Steps

1. Require `target` from `read_site_identity` or user input.
2. Call `pressidium_wordpress_state` with target.
3. Extract `structured.snapshot` from the tool response and forward it to the composite. On **hosting and EDGE**, the snapshot includes `wpVersion` and `phpVersion` (`string | null`). EDGE `get_site` still has no WP/PHP — use this block, not identity, for EDGE runtime versions.
4. Flag `pluginsSummary.vulnerableCount > 0` as severity high in downstream reports.
