---
description: Connect this repo + machine to a Workflows board through the packaged machine-connection operation.
argument-hint: <apiUrl> <projectId>   (or paste the link from the board's Setup page)
---

Connect this repository and machine to a Workflows board.

Input the user provided: `$ARGUMENTS`

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
   contains no credential.

5. Report the connected board URL and project id. The user may need to restart Claude Code or run `/mcp`
   so the server reopens with the new repository context.
