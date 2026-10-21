---
name: pressidium-act-disable-ssh
description: Pressidium building block — disable SSH on a managed hosting website. Low-risk write; hosting only.
---

# act_disable_ssh

Low-risk write. No `confirm`. Hosting only — not EDGE.

Call `pressidium_website_ssh` with `action: disable`, `wait: true`.

Parse `jobStatus=` from tool **text** (not the headline). Do not treat “submitted” as success.
If `jobStatus=success` (including `job=none` / no job id), report SSH off using any `enabled=` / host / port / username tokens in the text, then stop without polling — do not call `await_job`.
If `jobStatus=running` **and** a `jobId` is present, follow **await_job loop** in `docs/skills/block-contracts.md`. Resume with `pressidium_operations` `action: await_job` and this `jobId`. After the loop, if still `running`, tell the user `jobId` / `await_job`; on `success`, report the same SSH-off tokens then stop. If `running` with no `jobId`, do not call `await_job`; report the wait status; suggest `read-ssh-state` if tokens are missing. On `error` / `unknown`, do not invent connection facts; suggest `read-ssh-state` later.
