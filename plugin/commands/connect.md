---
description: Use when connecting a repository and machine to a Workflows board.
argument-hint: <apiUrl> <projectId>   (or paste the connection value from the board's Setup page)
---

## Required Workflows MCP capability

Before starting **connect this machine and repository to Workflows**, confirm that the MCP server named `workflows` and its bootstrap tools are available. If the server is absent or failed to initialize, stop before loading another procedure or beginning the operation.

Confirm that the bootstrap tools are available. Do not call a project tool as a readiness probe; this connection operation establishes the context those tools require.

On any readiness stop, do not use the repository CLI, browser automation, raw HTTP or API calls, or another write path. Use a different interface only when the user explicitly requests that specific interface; that request grants no authority beyond existing product boundaries.

Respond with these four lines and do not claim partial success:

```text
MCP: Workflows (`workflows`)
Operation: connect this machine and repository to Workflows
Outcome: not started
Recovery: Restore the `workflows` server, use `/mcp` to retry it, and start a fresh Claude Code session only if the repaired tool catalog is not visible in this session.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

Connect this repository and machine to a Workflows board.

Input the user provided: `$ARGUMENTS`

1. Parse whitespace-separated `apiUrl projectId`. Require both values; never guess a project id.

2. Call the packaged `connect_machine` MCP tool exactly once with `{ apiUrl }`. This is the one
   executable connection policy: it validates the board URL, performs the single `/health` discovery,
   serializes the machine-auth mutation, recovers malformed machine state, and returns no credential.
   Do not fetch `/health`, inspect machine files, or reproduce any of those rules in this command.

3. If `connect_machine` reports `personalCredential: "sign-in-required"`, call `begin_device_login`
   exactly once. Show its verification URL and one-time code to the human, then pause for them to approve
   it. After they confirm approval, call `complete_device_login` exactly once with the returned
   `attemptId`. If either operation fails, report its safe error and leave the repository unconfigured.

4. Only after the personal credential is ready, merge `{ "projectId": "<projectId>" }` into this repository's
   `.workflows/config.json`, preserving existing `verify` and `setup` fields. This repository file
   contains no credential.

5. Report the connected board URL and project id. The user may need to restart Claude Code or run `/mcp`
   so the server reopens with the new repository context.
