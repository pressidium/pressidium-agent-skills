---
name: pressidium-read-staging-state
description: Pressidium building block — managed hosting staging presence from get_site text. Use for Staging Validator hard-stop when staging is absent.
---

# read_staging_state

**Permission:** read

## When to use

- Staging Validator after `read_site_identity` confirms managed hosting
- Any workflow that needs an explicit staging present/absent fact from tool text

## Steps

1. Require hosting `target` from `read_site_identity` (or user input). If `siteSummary.kind === edge`, do not use this block for promote workflows — Staging Validator already stops as unsupported.
2. Call `pressidium_get_site` with `{ website }` (and optional `environment` if the caller already set one — staging presence is on the website detail payload regardless).
3. Prefer the **tool text** response. Look for `staging: present` (optionally with domain in parentheses) or `staging: absent`.
4. Build snapshot for the composite:

```json
{
  "website": "myblog",
  "staging": "present",
  "pressidiumDomainName": "optional-when-present"
}
```

Use `"staging": "absent"` and omit fabricated domains when text says absent. If text lacks a clear present/absent cue, set staging to unknown and let the composite write **Data unavailable** / conservative CAUTION — do not invent present.

5. Return snapshot to the calling composite; do not create, delete, or deploy staging.
