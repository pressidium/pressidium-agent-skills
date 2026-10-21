---
name: pressidium-read-cert-dns-state
description: Pressidium building block — domain and SSL certificate inventory. Use for Site Guardian and cert expiry monitoring.
---

# read_cert_dns_state

**Permission:** read

## When to use

- Site Guardian SSL / domains section
- Pre-flight checks before domain or cert changes
- Flag certificates nearing expiry

## Steps

1. Require `target` from `read_site_identity` or user input.
2. Call `pressidium_domains` with `action: "list"` and `target`.
3. Site SSL is in **domains list text**. Parse `https: on|off`, `force-https: on|off`, and `cert:` (`valid` / `expired` / `none` / `unavailable`). `cert: not found (id …)` on a short page is a dangling id; `cert: not found in first 100` means the account certificate page was full (pagination honesty). Header is `Domains for …: primary …`. A platform-default availability line means SSL status is not exposed via the mapped-domain API — do not invent a cert for that hostname.
4. Call `pressidium_certificates` with `action: "list"` (no site `target` — account-wide). This is **fleet expiry**: UUID `id` values plus `cert: valid|expired` and `expiration`. Do not join account rows to the site by hostname guesswork.
5. From tool **text**, collect domain records (`https:`, `force-https:`, `cert:`). Structured `domains` (or equivalent) is optional when the client exposes it.
6. From tool **text**, collect certificate records (`id`, `domains`, websites, `cert: valid|expired`, `expiration`). For certificate times, prefer `*Human` companions when present (e.g. `importedOnHuman` for custom cert import time). Certificate **expiry** is `expiration` — a human-readable string from the API, not a Unix epoch; parse `expiration` directly when flagging certs expiring within 30 days (do not expect `expirationHuman`). Populate `expiringSoon` and set `status: "warning"` when any cert is within 30 days of expiry.
7. Build snapshot:

```json
{
  "domains": [],
  "certificates": [],
  "expiringSoon": []
}
```

8. Return snapshot to calling composite; do not mutate domains or certificates.
