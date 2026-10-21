---
name: pressidium-act-enable-ssh
description: Pressidium building block — ensure the user has an SSH public key, then enable website SSH and print the ssh command. Low-risk write; hosting only.
---

# act_enable_ssh

Low-risk write. No `confirm`. Hosting only — not EDGE. Never generate or request a **private** key.

## Steps

1. Resolve the website (`read-site-identity` / `pressidium_list_sites`) if unnamed. **Stop** on EDGE.
2. Call `pressidium_ssh_keys` with `action: list` first, starting at `page: 1` (default `page_size` 25). Parse **text** only. Track `sawSkipped` across pages (sticky: once true, stays true).

**Header line** = first line of text. **Confirmed-empty page** = entire text, after removing at most one trailing newline, equals exactly `Loaded 0 SSH key(s).` (no skipped suffix, no `nextPage=`, no row lines).

| Observation | Action |
|-------------|--------|
| `isError` or entire text equals `SSH key list unavailable (unrecognized response).` | **Stop.** Report uncertainty / the error. Do not enable. Do not instruct keygen as if empty. |
| Header `Loaded N SSH key(s).` with `N >= 1` (a displayed row follows) | **Proceed** (account has a registered key). Stop walking. |
| Zero displayed and header has `nextPage=P` (`P` positive integer) | Call list again with `page: P`. If this page had a skipped suffix, set `sawSkipped`. |
| End of pagination (no `nextPage`) and `sawSkipped` (this page or an earlier page) | **Stop.** Report uncertainty (unusable rows existed). Do not enable. Do not keygen. |
| Confirmed-empty page on the **initial page 1** (no suffixes) | **Keygen** path (do not enable until a later list is non-empty). |
| Walked past page 1; every page had 0 displayed and never `sawSkipped`; last page has no `nextPage` | **Keygen** (true empty across pages). |
| Any other text | **Stop.** Uncertainty. |

**Do not** treat `Loaded 0 SSH key(s). (N skipped)` as empty. **Do not** treat a later confirmed-empty page as empty if `sawSkipped` is set.

3. If the keygen path applies: **do not enable** (even if SSH is already on). Instruct local key creation and wait for a pasted **public** key:

   ```bash
   ssh-keygen -t rsa -C "you@example.com"
   ```

   - Linux/macOS: `~/.ssh/id_rsa.pub`
   - Windows PowerShell: `$env:USERPROFILE\.ssh\id_rsa.pub`
   - If Windows says `ssh-keygen` is not recognized: Settings → Apps → Optional features → add **OpenSSH Client**, then retry in a new PowerShell window. Git Bash also provides `ssh-keygen`.

   Paste the one-line **public** key (must start with `ssh-rsa `). Never paste the private key. If the paste looks like a private key (`BEGIN OPENSSH PRIVATE KEY`, `BEGIN RSA PRIVATE KEY`, or does not start with `ssh-rsa `), refuse and ask for the `.pub` line. Do not add `ssh-ed25519` or ECDSA keys.

   After a valid paste: `pressidium_ssh_keys` `action: add` with `publicKey` (optional `title`). If `add` fails, show the API error and **do not enable** until a later `list` is non-empty (a displayed row with `N >= 1`).
4. Keys exist. Call `pressidium_website_ssh` `action: status`. If `enabled=on` and the `ssh -p` line is present, print it and **stop**. Otherwise call `pressidium_website_ssh` with `action: enable`, `wait: true`. Parse `jobStatus=` from **text**. If `jobStatus=success` (including `job=none` / no job id), print status tokens and the `ssh` command (`ssh -p` when present), then stop without polling — do not call `await_job`. If `jobStatus=running` **and** a `jobId` is present, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`. After the loop, if still `running`, tell the user `jobId` / `await_job`; on `success`, print the same status tokens and the `ssh` command then stop. If `running` with no `jobId`, do not call `await_job`; report the wait status; suggest `read-ssh-state` if tokens are missing. On `error` / `unknown`, do not invent connection facts; use `read-ssh-state` after the job.

Default `environment` is `production` unless the user asked for staging.
