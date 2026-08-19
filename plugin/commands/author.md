---
description: Build a new workflow or amend an existing one to good shape — its transitions, step prompts, documents, router conditions, terminal outcomes, and input contract — over the workflows MCP write ops.
argument-hint: [workflow to create or change]
---

## Required Workflows MCP capability

Before starting **create or change a workflow**, confirm that the MCP server named `workflows` and its bootstrap tools are available. If the server is absent or failed to initialize, stop before loading another procedure or beginning the operation.

Before business work, call `select_repository` once — exactly once — with the active repository's absolute root path. This is the only Workflows tool call allowed as a readiness probe. If this entry's command wrapper already completed the same probe, treat it as complete and do not call it again. If selection fails, stop.

On any readiness stop, do not use the repository CLI, browser automation, raw HTTP or API calls, or another write path. Use a different interface only when the user explicitly requests that specific interface; that request grants no authority beyond existing product boundaries.

Respond with these four lines and do not claim partial success:

```text
MCP: Workflows (`workflows`)
Operation: create or change a workflow
Outcome: not started
Recovery: Restore the `workflows` server, use `/mcp` to retry it, and start a fresh Claude Code session only if the repaired tool catalog is not visible in this session.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

Run the `workflows.author` skill for the following request: $ARGUMENTS

If no request was provided, ask the user which workflow they want to create or change, then proceed.
