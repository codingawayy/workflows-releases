---
description: Inspect every workflow in the connected project and propose, create, reconcile, or remove project-local agent operations for named transitions and human intervention. Use when the user wants workflow-specific skills, slash commands, or an operations surface generated from the project's live workflow definitions.
---

## Required Workflows MCP capability

Before starting **build project-specific workflow operations**, confirm that the MCP server named `workflows` and its bootstrap tools are available. If the server is absent or failed to initialize, stop before loading another procedure or beginning the operation.

Before business work, call `select_repository` once — exactly once — with the active repository's absolute root path. This is the only Workflows tool call allowed as a readiness probe. If this entry's command wrapper already completed the same probe, treat it as complete and do not call it again. If selection fails, stop.

On any readiness stop, do not use the repository CLI, browser automation, raw HTTP or API calls, or another write path. Use a different interface only when the user explicitly requests that specific interface; that request grants no authority beyond existing product boundaries.

Respond with these four lines and do not claim partial success:

```text
MCP: Workflows (`workflows`)
Operation: build project-specific workflow operations
Outcome: not started
Recovery: Restore the Workflows bundle and registration, reload or restart OpenCode, and start a fresh session if the repaired tool catalog remains stale.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

Load the `workflows-build-skills` skill and follow it completely.

Build project-specific workflow operations for the following request: $ARGUMENTS

Treat the input as an optional focus. With no input, inspect every workflow and every existing workflow operation before proposing changes.
