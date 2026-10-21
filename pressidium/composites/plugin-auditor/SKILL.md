---
name: pressidium-plugin-auditor
description: Pressidium Starter composite — fleet or site plugin vulnerability and update rollup. Invoke with "Run Plugin Auditor" or "on {website}".
---

# Plugin Auditor

**Blocks:** `read_site_identity` (if site-scoped) → `read_plugin_state`  
Optional: `read_wordpress_state` for themes/update counts on single site

## Fleet report template

Fill **Site** columns from the `Sites with issues:` rollup line in account list **text**, then named `plugin @ website` findings. Keep the markdown tables.

```
# Plugin Auditor — account ({environment})

## Summary
- Sites with vulnerabilities: N
- Total vulnerable installations: N
- Total pending updates: N

## Critical (vulnerable)
| Plugin | Site | Notes |

## Updates available
| Plugin | Site | Current → Available |
```

Sort vulnerable entries first. Do not run updates unless user explicitly requests Maintenance Advisor with approval.
