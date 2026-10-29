---
name: pressidium-read-sftp-state
description: Pressidium building block — SFTP connection info (host, port, path) for a managed hosting website environment. Hosting only.
---

# read_sftp_state

**Permission:** read. Hosting only — not EDGE.

Call `pressidium_sftp_users` with `action: connection` and hosting `target` (`website` + `environment`, default production).

Report `host=`, `port=`, `path=` from tool **text**. Do not invent missing tokens. Do not invent a username or `sftp -P` line. The unavailable sentence is a successful empty connection, not a tool error.
