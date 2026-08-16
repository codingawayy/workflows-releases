---
name: connect
description: Use when connecting a repository and machine to a Workflows board.
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
Recovery: Restore or reinitialize the bundled `workflows` server, then start a fresh Codex session.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

Connect this repository and machine to a Workflows board.

Use the connection values provided in the user's request.

1. Parse either a `workflows://connect?apiUrl=…&projectId=…` link or whitespace-separated
   `apiUrl projectId`. URL-decode link parameters. Require both values; never guess a project id.

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
   contains no credential. Then call `select_repository` exactly once with this repository's absolute root path. The path selects transient app.mcp context for this session; never persist it in `.codex/config.toml` or any other configuration.

5. Report the connected board URL and project id. The bundled app.mcp is ready in this session; no MCP registration or restart is required.
