---
name: workflows.connect
description: Use when connecting a repository and machine to a Workflows board.
---

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
   `.workflows/config.json`, preserving existing `verify` and `worktreeSetup` fields. This repository file
   contains no credential. Require the `mcpEntry` returned by `connect_machine`, then merge a project-scoped `[mcp_servers.workflows]` table into `.codex/config.toml` with `command = "bun"` and `args = ["run", "<absolute mcpEntry>"]`, replacing any existing table of that name and preserving every unrelated setting. Do not set `cwd`: Codex must launch app.mcp from this repository.

5. Report the connected board URL and project id. The user may need to restart Codex or start a new Codex session
   so the server reopens with the new repository context.
