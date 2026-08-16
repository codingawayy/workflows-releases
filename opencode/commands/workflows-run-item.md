---
description: Drive a workflow item through its autonomous transitions from THIS interactive session — the next_step → execute → submit_step loop — instead of the headless engine. Invoke when the user wants to run, drive, advance, or work a backlog item in-session (e.g. "/workflows-run-item B26.070", "drive B26.012 in-session", "advance this item here").
---

## Required Workflows MCP capability

Before starting **drive a workflow item**, confirm that the MCP server named `workflows` and its bootstrap tools are available. If the server is absent or failed to initialize, stop before loading another procedure or beginning the operation.

Before business work, call `select_repository` once — exactly once — with the active repository's absolute root path. This is the only Workflows tool call allowed as a readiness probe. If this entry's command wrapper already completed the same probe, treat it as complete and do not call it again. If selection fails or context-dependent project tools remain unavailable, stop.

On any readiness stop, do not use the repository CLI, browser automation, raw HTTP or API calls, or another write path. Use a different interface only when the user explicitly requests that specific interface; that request grants no authority beyond existing product boundaries.

Respond with these four lines and do not claim partial success:

```text
MCP: Workflows (`workflows`)
Operation: drive a workflow item
Outcome: not started
Recovery: Restore the Workflows bundle and registration, reload or restart OpenCode, and start a fresh session if the repaired tool catalog remains stale.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

Load the `workflows-run-item` skill and follow it completely.

Drive item `$ARGUMENTS` using the instructions below.

The first argument is the item id (e.g. `B26.071`); an optional second argument names the exact transition to take at a forked status. If no item id was provided, ask the user which item to drive.
