# Pressidium Agent Skills — Client Getting Started

Connect your AI agent to the **Pressidium-hosted** Dashboard MCP server and install the Pressidium skill library on your machine.

You do **not** install or run the MCP server locally. Pressidium hosts it.

After MCP is connected, **ask the agent to install/update Pressidium skills.**

This library ships **42 skills** under `pressidium/` (**33** building blocks + **9** composites).

| Environment | MCP URL |
|-------------|---------|
| **Production** | `https://mcp-dashboard.pressidium.com/dashboard` |
| **Staging** (QA) | `https://mcp-dashboard-staging.pressidium.com/dashboard` |

| Product | MCP config | Skills |
|---------|------------|--------|
| **Cursor IDE** | `.cursor/mcp.json` or `~/.cursor/mcp.json` | Copy into `.cursor/skills/` |
| **Claude Code** | project `.mcp.json` | Copy into `.claude/skills/` |
| **Claude Desktop** | `claude_desktop_config.json` | Upload skill zips in the app UI |
| **Codex** | `~/.codex/config.toml` | Copy into `~/.agents/skills/` |

---

## Prerequisites

1. A Pressidium Dashboard account. Prefer a **dedicated** non-MFA account (service account) for MCP — password login only until token auth ships. Restrict permissions on files that hold credentials (e.g. `chmod 600` on macOS/Linux for `mcp.json`, `claude_desktop_config.json`, and `~/.codex/config.toml`).
2. The public Release zip [`skills-latest.zip`](https://github.com/pressidium/pressidium-agent-skills/releases/latest/download/skills-latest.zip) (contains a `pressidium/` folder with `blocks/` and `composites/`).
3. One of: Cursor, Claude Code, Claude Desktop, or Codex.

Optional environment variables (recommended where your product expands them):

```bash
export PRESSIDIUM_MCP_URL=https://mcp-dashboard.pressidium.com/dashboard
# Staging QA: export PRESSIDIUM_MCP_URL=https://mcp-dashboard-staging.pressidium.com/dashboard
export PRESSIDIUM_EMAIL=you@example.com
export PRESSIDIUM_PASSWORD=your-password
```

Never commit passwords into git.

---

## Guided install (recommended)

Connect MCP first (section 2), then follow **section 3** (`pressidium_skills`).

- **Absent tree:** if `pressidium/` is not at the product skills destination, the agent offers install. It states the destination and warns that local edits under that folder will be lost **before** you confirm. Confirming downloads the Release zip and writes the tree.
- **Existing tree:** if `pressidium/` is already present (including a missing or invalid `VERSION`), the agent offers to **check for updates** first (permission to download to a temp directory). If versions differ, it asks a **separate** replacement approval. Confirming replacement uses the archive that was checked.

Do not confirm replacement until you have seen the destination and the local-edit warning.

Claude Desktop is not a filesystem install: use the zip-upload steps in section 4.

---

## 1. Get the skills package

Download [`skills-latest.zip`](https://github.com/pressidium/pressidium-agent-skills/releases/latest/download/skills-latest.zip).

1. Unzip it in an empty folder: `unzip skills-latest.zip`
2. Confirm you see a `pressidium/` directory with `blocks/` and `composites/` inside.
3. Keep that folder handy for the manual install steps below.

**Human option:** clone the public repository instead of the zip:

```bash
git clone https://github.com/pressidium/pressidium-agent-skills.git
```

Cloning alone does **not** install or discover skills. You still need the guided install (ask the agent) or a manual copy/upload into a product skills location.

You only need the skills tree. You do not need any MCP server source code on your laptop.

---

## 2. Connect the MCP server

Send these headers on MCP initialize (every product):

| Header | Value |
|--------|--------|
| `X-Pressidium-Email` | Dashboard account email |
| `X-Pressidium-Password` | Dashboard password |

Use the **production** URL unless Pressidium asked you to use staging.

### Cursor IDE

Create or edit `~/.cursor/mcp.json` (personal) or project `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "pressidium": {
      "url": "https://mcp-dashboard.pressidium.com/dashboard",
      "headers": {
        "X-Pressidium-Email": "you@example.com",
        "X-Pressidium-Password": "your-password"
      }
    }
  }
}
```

Replace the email and password with your own. Keep this file out of git if it contains secrets.

In Cursor: **Settings → MCP** — enable `pressidium`, then refresh MCP servers or restart Cursor.

**Known limitation:** some Cursor builds treat URL-based MCP servers as OAuth and call `POST /register`. Pressidium does not implement MCP OAuth. If you see `Cannot POST /register`, use **Claude Code** or **Codex** until your Cursor build supports custom header HTTP without OAuth.

### Claude Code

In your project root, create `.mcp.json`:

```json
{
  "mcpServers": {
    "pressidium": {
      "type": "http",
      "url": "${PRESSIDIUM_MCP_URL}",
      "headers": {
        "X-Pressidium-Email": "${PRESSIDIUM_EMAIL}",
        "X-Pressidium-Password": "${PRESSIDIUM_PASSWORD}"
      }
    }
  }
}
```

Export `PRESSIDIUM_MCP_URL`, `PRESSIDIUM_EMAIL`, and `PRESSIDIUM_PASSWORD` in your shell before starting Claude Code.

CLI alternative (flags may vary by version; `.mcp.json` is canonical):

```bash
claude mcp add --scope project --transport http pressidium \
  --url "$PRESSIDIUM_MCP_URL" \
  --header "X-Pressidium-Email=$PRESSIDIUM_EMAIL" \
  --header "X-Pressidium-Password=$PRESSIDIUM_PASSWORD"
```

Restart Claude Code. Run `/mcp` and confirm `pressidium` is connected.

### Claude Desktop

Edit the Claude Desktop config file:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "pressidium": {
      "type": "http",
      "url": "https://mcp-dashboard.pressidium.com/dashboard",
      "headers": {
        "X-Pressidium-Email": "you@example.com",
        "X-Pressidium-Password": "your-password"
      }
    }
  }
}
```

Restart Claude Desktop after saving. Do not commit this file if it contains passwords.

If Claude Desktop rejects remote HTTP MCP (version or policy limits), use **Claude Code** or **Codex** with the same URL and headers instead.

### Codex (CLI and IDE extension)

Edit `~/.codex/config.toml`:

```toml
[mcp_servers.pressidium]
url = "https://mcp-dashboard.pressidium.com/dashboard"
enabled = true

[mcp_servers.pressidium.env_http_headers]
"X-Pressidium-Email" = "PRESSIDIUM_EMAIL"
"X-Pressidium-Password" = "PRESSIDIUM_PASSWORD"
```

Export `PRESSIDIUM_EMAIL` and `PRESSIDIUM_PASSWORD` before starting Codex. In the TUI, run `/mcp` to confirm tools appear. Codex discovers skills automatically.

---

## 3. Install skills with pressidium_skills

`pressidium_skills` is a **read-only pointer**. It returns the public GitHub Release zip URL, repo URL, `pressidium/VERSION` path, and destination folders. It does **not** download or write files. Call this tool for URLs and paths; do not guess GitLab public/ or a Pressidium-hosted zip.

After MCP is connected (section 2), say in agent chat:

> Install Pressidium skills.

or:

> Call pressidium_skills.

or, for an existing tree:

> Update Pressidium skills.

Optional `product` (case-sensitive): `cursor`, `claude_code`, `codex`. Unknown or omitted lists all three filesystem destinations. Ask project vs personal for Cursor and Claude Code. Codex is personal only: `~/.agents/skills/pressidium`. Do not write `~/.codex/skills`. If a leftover exists only at `~/.codex/skills/pressidium`, mention it and install at the supported path; do not delete the leftover unless asked.

**Before any download or write you should see:**

1. The destination path.
2. A warning that replacing `pressidium/` deletes local edits under that folder.

**Then confirm each step:**

| Your tree | First confirm | Second confirm |
|-----------|---------------|----------------|
| **Absent** `pressidium/` | Install (download + write) | — |
| **Existing** tree (including missing or invalid `VERSION`) | Check for updates (temp download only) | Replace only if versions differ; reuse the checked archive — do not fetch latest again |

Declining a check downloads nothing. Declining replacement leaves the existing tree and deletes temp files.

Claude Desktop: do not unzip into a Desktop skills directory. Use the upload steps in section 4.

After success, the agent reports installed `VERSION`. A new chat may be required. Confirm the product **lists** nested skills such as `read-site-identity` and `site-guardian` — file presence is not enough.

---

## 4. Install skills (manual, absent destination)

Prefer section 3 (`pressidium_skills`). Manual copy is for an **absent destination** only. Do not `cp -R pressidium` onto an existing `pressidium/` tree — that can nest or retain stale files. If the tree already exists, ask the agent to check for updates before replacement.

Paths below assume you unzipped the package and your current directory contains `pressidium/`.

### Cursor

Project:

```bash
mkdir -p .cursor/skills
if [ -e .cursor/skills/pressidium ]; then
  echo "Pressidium skills already exist. Ask the agent to check for updates before replacement."
else
  cp -R pressidium .cursor/skills/pressidium
fi
```

Personal (all projects):

```bash
mkdir -p "$HOME/.cursor/skills"
if [ -e "$HOME/.cursor/skills/pressidium" ]; then
  echo "Pressidium skills already exist. Ask the agent to check for updates before replacement."
else
  cp -R pressidium "$HOME/.cursor/skills/pressidium"
fi
```

Enable skills in Cursor: **Settings → Rules / Skills** (labels may vary by version).

### Claude Code

Project:

```bash
mkdir -p .claude/skills
if [ -e .claude/skills/pressidium ]; then
  echo "Pressidium skills already exist. Ask the agent to check for updates before replacement."
else
  cp -R pressidium .claude/skills/pressidium
fi
```

Personal:

```bash
mkdir -p "$HOME/.claude/skills"
if [ -e "$HOME/.claude/skills/pressidium" ]; then
  echo "Pressidium skills already exist. Ask the agent to check for updates before replacement."
else
  cp -R pressidium "$HOME/.claude/skills/pressidium"
fi
```

### Codex

Personal only: `~/.agents/skills/pressidium`. Codex discovers skills automatically.

```bash
mkdir -p "$HOME/.agents/skills"
if [ -e "$HOME/.agents/skills/pressidium" ]; then
  echo "Pressidium skills already exist. Ask the agent to check for updates before replacement."
else
  cp -R pressidium "$HOME/.agents/skills/pressidium"
fi
```

### Claude Desktop

Claude Desktop does **not** load a filesystem skills tree the same way as Claude Code. Upload skills in the app UI (Settings → Capabilities / Features / Skills — labels vary by version):

1. For each skill you want, zip **that skill's folder** (the folder that contains `SKILL.md`).
2. Upload the zip in Claude Desktop.
3. Enable the skill.

**Quick start (recommended on Desktop):** upload the **9 composite** folders under `pressidium/composites/` only (Site Guardian, Backup Guardian, Plugin Auditor, Maintenance Advisor, Performance Review, Reliability Engineer, Growth Monitor, Staging Validator, Capacity Advisor).

Building blocks under `pressidium/blocks/` are optional on Desktop; Cursor, Claude Code, and Codex can use the full tree via copy.

Skills do **not** auto-sync from Claude Code into Claude Desktop — install on each surface separately.

---

## 5. Verify

In your agent chat:

> Use pressidium_whoami and pressidium_list_sites.

You should see your account identity and a list of websites or EDGE sites.

Then confirm the product **lists** `read-site-identity` and `site-guardian` in its skill picker, `/skills`, or `$` mention. File presence on disk does not pass this check. A new chat may be required after install.

---

## 6. Quick start — run composites

Composites are **read-only by default**. Write actions (backups, updates, cache purge, staging deploy, and similar) require your explicit approval.

### Starter

| Say in agent chat | Skill |
|-------------------|-------|
| Run Site Guardian on **myblog** | Site Guardian |
| Run Backup Guardian on **myblog** | Backup Guardian |
| Run Plugin Auditor | Plugin Auditor (fleet) |
| Run Plugin Auditor on **myblog** | Plugin Auditor (one site) |
| Run Maintenance Advisor on **myblog** | Maintenance Advisor |

### Operations

| Say in agent chat | Skill |
|-------------------|-------|
| Run Performance Review on **myblog** | Performance Review |
| Run Reliability Engineer on **myblog** | Reliability Engineer |
| Run Growth Monitor on **myblog** | Growth Monitor |
| Run Staging Validator on **myblog** | Staging Validator (managed hosting only; not EDGE) |
| Run Capacity Advisor | Capacity Advisor (fleet) |
| Run Capacity Advisor on **myblog** | Capacity Advisor (one site) |

Replace `myblog` with your website name or EDGE site id as appropriate.

---

## Setup blockers

| Symptom | What to try |
|---------|-------------|
| Auth / 401 errors | Use a non-MFA Dashboard account; confirm email/password headers on initialize |
| Cursor `Cannot POST /register` | Cursor OAuth probe — use Claude Code or Codex, or a Cursor build that supports header HTTP without OAuth |
| MCP URL unreachable | Confirm you are using the URL Pressidium gave you (production vs staging) |
| Agent ignores skills | New chat session; correct skills path; Cursor: skills enabled in Settings; Codex: restart if they do not appear |
| Claude Desktop has MCP but no workflows | Upload composite skill zips in the Desktop UI (filesystem copy alone is not enough) |
| Claude Desktop rejects remote HTTP MCP | Use Claude Code or Codex with the same URL and headers; Desktop support varies by version |

---

## See also

- [block-contracts.md](./block-contracts.md)
