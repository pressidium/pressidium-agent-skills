---
name: pressidium-act-install-ai-kit
description: Pressidium building block — offer to download the site workspace kit zip into a confirmed new folder. Guidance only; user runs ./remote setup then ./remote provision. Hosting only.
---

# act_install_ai_kit

Guidance. Not a Dashboard write. Managed hosting only — not EDGE. No in-place replace. Call `pressidium_ai_kit` for the zip URL and rules; do not guess GitLab public/, the skills Release zip, or a Dashboard download.

See `docs/skills/block-contracts.md` (`## act_install_ai_kit`).

## When to use

- User asked to install or update the Pressidium site workspace kit
- Unsolicited offer: hosting target, destination has **neither** `./remote` nor root `VERSION`
- Unsolicited update-check: user confirmed a dedicated kit directory
- File/shell ask (files, WP-CLI, SQL, mount, local copy) **and no kit** — may offer install
- File/shell ask **and a confirmed kit is present** — do **not** use this block; continue with `./remote`. Do not re-enter update-check.

Do **not** invoke from composites (Site Guardian and others stay MCP-only).

## Steps

1. If the target is EDGE, stop. Say Layer B is hosting-only; Layer A MCP/skills still work.
2. Call `pressidium_ai_kit` (optional `website` string for the default folder name only — not a Target). Parse **text**. Zip: `https://github.com/pressidium/pressidium-ai-kit/releases/latest/download/pressidium-ai-kit.zip`. Version file: `VERSION`.
3. Ask for the destination **before** download. Default `~/pressidium-sites/{slug}` with `{slug}` a **single path-safe** component (non-empty, no `/` or `\`, no `..`, not `.` or `~`). Examples: `acorn2clone`, `myblog`. If the slug is unsafe, omit it and ask for a folder name. Occupied non-kit folder → another path. Do not unpack into `.cursor/skills/pressidium`, `~/.cursor/skills/pressidium`, `.claude/skills/pressidium`, or `~/.agents/skills/pressidium`.
4. Treat `./remote` and root `VERSION` as **hints**. Install only when dest is missing, empty, or has **neither** marker. A lone `VERSION` is not a kit. Before update-check, the user must confirm the path is a **dedicated kit directory**.
5. Confirm download in chat. Never download until they confirm.

### First install (neither marker; dest missing or empty)

Confirm → download to temp → validate → recheck destination is still missing/empty → unpack → cleanup → setup/provision reminder.

If that final occupancy recheck fails, stop and delete temp files without writing the destination.

Package-validate before unpack: zip path safety (no absolute paths, no `..`, no symlinks/hard links); `VERSION` + `remote` at zip root. Not authenticity.

### Update (confirmed kit directory)

Update-check yes is **download-only**. Compare VERSION (ASCII trim; core semver equality, not precedence). Same version: report, delete temp, stop.

Different: print this warning, then **preflight the old folder**:

> Do not overwrite a live kit folder. Unpack into a new folder beside the old copy. Copy site.env yourself. Do not copy .state/ or site/. Finish provision and unmount in the old folder first. Live mounts, pending provision in .state/, and unpushed site/ files are not migrated.

You may run `./remote doctor` and `./remote info` in the **old** folder (not setup/provision; never read `site.env`). Stop if provision is unfinished, a mount is live/starting/dirty/unknown, or status is uncertain.

Then ask for a **new** dest (not the old path). On yes: recheck dest is still missing/empty → unpack the **already** downloaded archive → cleanup. Do not GET the zip again. If the temp file is gone, stop; do not re-fetch. Never extract into the old folder. Never delete the old folder. Never copy `site.env`, `.state/`, or `site/`. If the recheck fails, stop and delete temp without writing dest.

### After every successful unpack

Print the destination. Tell the user to run **in a terminal**: `cd` there, `./remote setup`, **then** `./remote provision`. Do not run `./remote setup`. Do not run `./remote provision`. Do not write `site.env` or mcp.json. Do not read secrets. For updates, also tell them to copy `site.env` only from the old folder before setup.

Do not auto-submit `pressidium_feedback` as part of this flow. An unsolicited offer is consumed when presented, any outcome. Explicit install/update may retry.
