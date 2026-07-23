---
description: Connect this repo + machine to a Workflows board through the packaged machine-connection operation.
argument-hint: <apiUrl> <token> <projectId>   (or paste the link from the board's Setup page)
---

Connect this repository and machine to a Workflows board.

Input the user provided: `$ARGUMENTS`

1. Parse either a `workflows://connect?apiUrl=…&token=…&projectId=…` link or whitespace-separated
   `apiUrl token projectId`. URL-decode link parameters. Require all three values; never guess a project id.

2. Call the packaged `connect_machine` MCP tool exactly once with `{ apiUrl, token }`. This is the one
   executable connection policy: it validates the board URL, performs the single `/health` discovery,
   serializes the machine-auth mutation, recovers malformed machine state, and never returns the bearer.
   Do not fetch `/health`, inspect machine files, or reproduce any of those rules in this command.

3. Only after `connect_machine` succeeds, merge `{ "projectId": "<projectId>" }` into this repository's
   `.workflows/config.json`, preserving existing `verify` and `worktreeSetup` fields. This repository file
   contains no credential.

4. Report the connected board URL and project id, never the token. The user may need to restart Claude
   Code or run `/mcp` so the server reopens with the new repository context.
