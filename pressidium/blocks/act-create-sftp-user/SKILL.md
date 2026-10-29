---
name: pressidium-act-create-sftp-user
description: Pressidium building block — create an SFTP user on a managed hosting website, wait for the job, and print host/port/path plus an sftp command. Low-risk write; hosting only.
---

# act_create_sftp_user

Low-risk write. No `confirm`. Hosting only — not EDGE. Never print the password.

## Steps

1. Resolve the website (`read-site-identity` / `pressidium_list_sites`) if unnamed. **Stop** on EDGE.
2. Operator supplies **username** (full name; MCP does not add `{website}-prod-` / `{website}-stag-`) and **password**. If either is missing, **stop** and ask. Do not invent either. Never print the password.
3. Call `pressidium_sftp_users` with `action: create`, `wait: true`. Parse `jobStatus=` from tool **text** (not the headline).
4. If `jobStatus=success` from a finished wait: print `username=` / `host=` / `port=` / `path=` and the `sftp -P` line from create **text** when present, then stop without `await_job`. If those connection tokens are missing, call `connection` and build `sftp -P {port} {username}@{host}` from the username the skill submitted when host and port are present. Do not infer success from a missing job ID: SFTP create without a job ID returns `unknown`.
5. If `jobStatus=running` **and** a `jobId` is present: follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`. After the loop, if still `running`, tell the user `jobId` / `await_job`. On `success`, call `connection` (operations text has no SFTP facts) and print tokens plus the command from the submitted username; then stop. If `running` with no `jobId`, do not call `await_job`; report the wait status; suggest `read-sftp-state`.
6. If the initial result is `jobStatus=unknown` **and** text says the result could not be confirmed (`could not be confirmed` / `probeFailed` / unconfirmed) **and** a `jobId` is present: call `pressidium_operations` `action: await_job` once with that `jobId`. On `success`, call `connection` and print available tokens plus the command from the submitted username. If that retry returns `running` with a `jobId`, continue the canonical running loop; count the retry toward its ceiling of 10 `await_job` calls and do not reset the one-unconfirmed-retry allowance. If the retry remains `unknown` or `error`, stop. Never re-POST create to recover job status.
7. On `error`, ordinary `unknown` without unconfirmed language, or `unknown` / `running` without a `jobId`: stop without `await_job`. Do not claim the user exists or creation succeeded; do not invent connection facts. `read-sftp-state` is allowed for host/port/path only. Creation is confirmed only by `jobStatus=success`.

Default `environment` is `production` unless the user asked for staging.
