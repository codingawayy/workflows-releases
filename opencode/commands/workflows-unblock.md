---
description: Use when inspecting a project's human-intervention queue, reviewing selected waiting items, or guiding the user through one intervention at a time.
---

## Required Workflows MCP capability

Before starting **inspect and resolve workflow interventions**, confirm that the MCP server named `workflows` and its bootstrap tools are available. If the server is absent or failed to initialize, stop before loading another procedure or beginning the operation.

Before business work, call `select_repository` once — exactly once — with the active repository's absolute root path. This is the only Workflows tool call allowed as a readiness probe. If this entry's command wrapper already completed the same probe, treat it as complete and do not call it again. If selection fails or context-dependent project tools remain unavailable, stop.

On any readiness stop, do not use the repository CLI, browser automation, raw HTTP or API calls, or another write path. Use a different interface only when the user explicitly requests that specific interface; that request grants no authority beyond existing product boundaries.

Respond with these four lines and do not claim partial success:

```text
MCP: Workflows (`workflows`)
Operation: inspect and resolve workflow interventions
Outcome: not started
Recovery: Restore the Workflows bundle and registration, reload or restart OpenCode, and start a fresh session if the repaired tool catalog remains stale.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

Load the `workflows-unblock` skill and follow it completely.

Inspect project interventions using the instructions below for: $ARGUMENTS

Each argument is an optional item id (e.g. `B26.070`). Preserve every supplied identifier and its order. With no identifiers, start the guided project pass.
