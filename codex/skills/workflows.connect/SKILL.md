---
name: workflows.connect
description: Connect this repo + machine to a Workflows board — write the per-machine credential and the per-repo project id so the workflows MCP can authenticate and resolve its project.
---

Connect to a Workflows board. The `workflows` MCP server needs **two** facts to come up, and this
command writes both — the board's **Setup** page (`/setup`) hands you the values:

- a **per-machine** credential — the board `apiUrl` + a bearer `token` (shared across every repo on
  this machine), and
- a **per-repo** `projectId` — which Workflows project *this* repo maps to.

Use the connection values provided in the user's request.

Do this:

1. **Parse the inputs.** Accept either a `workflows://connect?apiUrl=…&token=…&projectId=…` link (extract
   and URL-decode the query params) or whitespace-separated values in the order `apiUrl token projectId`.
   Validate that `apiUrl` is an `http(s)://` URL, the token is non-empty, and `projectId` is non-empty.
   If any is missing, stop and ask the user to paste the link or the three values from the board's Setup
   page — do not guess a `projectId`.

2. **Write the per-machine credential.** The file is `client.json` under the Workflows home's
   `.workflows/` directory — i.e. `$WORKFLOWS_HOME/.workflows/client.json` when `WORKFLOWS_HOME` is set,
   otherwise `~/.workflows/client.json` (create the directory if needed). Read any existing file first and
   **merge**, setting `apiUrl` and `apiToken` and leaving other keys intact:

   ```json
   { "apiUrl": "<apiUrl>", "apiToken": "<token>" }
   ```

   The token is a code-execution-grade secret — write it only to this file, never echo it back in full,
   and never write it into the project repo.

3. **Write the per-repo project id.** In the current repo, write `.workflows/config.json` with the
   `projectId` (merge into any existing file, leaving `verify`/`worktreeSetup` intact):

   ```json
   { "projectId": "<projectId>" }
   ```

   Without this, the `workflows` MCP server cannot resolve which project the repo operates and will fail
   to start. (This file holds no secret — it is the per-repo fact the team can commit.)

4. **Confirm.** Report the board URL and the project id you connected (never the token), and tell the
   user the `workflows` MCP tools are now configured — they may need to restart Codex or start a new Codex session
   for the server to pick up the new config.
