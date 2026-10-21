---
name: pressidium-read-ssh-state
description: Pressidium building block — website SSH enabled flag and connection info (host, port, username). Hosting only.
---

# read_ssh_state

**Permission:** read. Hosting only — not EDGE.

Call `pressidium_website_ssh` with `action: status` and hosting `target` (`website` + `environment`, default production).

Report `enabled=`, `host=`, `port=`, `username=`, and the `ssh -p` line from tool **text**. Do not invent connection facts when `enabled=unknown`.
