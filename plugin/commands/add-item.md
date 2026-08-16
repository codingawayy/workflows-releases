---
description: Turn natural-language work into well-recorded items — break it down, route each piece to the right workflow, record it to that workflow's quality bar, and (on approval) create the items with their dependencies and epics.
argument-hint: [what to capture]
---

## Required Workflows MCP capability

Before starting **capture and route workflow items**, confirm that the MCP server named `workflows` and its bootstrap tools are available. If the server is absent or failed to initialize, stop before loading another procedure or beginning the operation.

Before business work, call `select_repository` once — exactly once — with the active repository's absolute root path. This is the only Workflows tool call allowed as a readiness probe. If this entry's command wrapper already completed the same probe, treat it as complete and do not call it again. If selection fails or context-dependent project tools remain unavailable, stop.

On any readiness stop, do not use the repository CLI, browser automation, raw HTTP or API calls, or another write path. Use a different interface only when the user explicitly requests that specific interface; that request grants no authority beyond existing product boundaries.

Respond with these four lines and do not claim partial success:

```text
MCP: Workflows (`workflows`)
Operation: capture and route workflow items
Outcome: not started
Recovery: Restore the `workflows` server, use `/mcp` to retry it, and start a fresh Claude Code session only if the repaired tool catalog is not visible in this session.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

Run the `workflows.add-item` skill on the following input: $ARGUMENTS

If no input was provided, ask the user what work they want to capture, then proceed.
