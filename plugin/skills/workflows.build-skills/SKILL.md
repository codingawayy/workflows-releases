---
name: workflows.build-skills
description: Inspect every workflow in the connected project and propose, create, reconcile, or remove project-local agent operations for named transitions and human intervention. Use when the user wants workflow-specific skills, slash commands, or an operations surface generated from the project's live workflow definitions.
user-invocable: false
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
Recovery: Restore the `workflows` server, use `/mcp` to retry it, and start a fresh Claude Code session only if the repaired tool catalog is not visible in this session.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

# Build project workflow skills

Build a repository-owned operations layer from the connected project's live workflow definitions. The
result should let a person invoke a meaningful transition by name, conduct its human interview when it has
an interactive step, deliberately run an automatic or routed transition, and reach the existing intervention
queue without learning MCP tool names.

This operation changes repository files only. While surveying or generating operations, never call
`next_step`, `submit_step`, `take_move`, `set_status`, `write_artifact`, `append_question`,
`unblock_item`, `retry_routing`, or any other tool that changes an item, claim, dialogue, document, or
workflow definition.

Before inspecting or drafting files, read [reference/operation-contract.md](reference/operation-contract.md).

## 1. Read the live project and the existing operations surface

Call `list_workflows`, then call `read_workflow_definition` once for every returned workflow. Keep each
workflow's `definitionStamp`; it is the freshness fence for the proposal.

Inventory the operation catalog already visible in this session and every project-local surface that exists:

- `.claude/commands/` and `.claude/skills/`;
- `.agents/skills/`;
- `.opencode/commands/` and `.opencode/skills/`;
- `.workflows/operations/` and `.workflows/operations.json`.

Read the complete file for every operation whose name, description, body, or managed metadata suggests it
drives Workflows. Include generic installed Workflows operations in the capability inventory even when their
files are outside the repository and cannot be changed. Classify repository files as **managed** only when
their marker and manifest entry agree; everything else is **unmanaged**.

For every live transition record its workflow, exact name, input and output status, description, router
condition, step kinds, artifacts read or published, and classification:

- **interactive** — at least one step has `kind: interactive`;
- **automatic** — `auto: true`;
- **routed** — `routerCondition` is present;
- **human-started** — neither automatic nor routed;
- **manual move** — it is human-started and has no steps, so it is taken as a shared-state move;
- **step-less automatic/routed** — it has no authored steps but is still selected through its automatic or
  routed execution path;
- **stepped** — it has one or more steps and is driven through `next_step` / `submit_step`.

Do not infer these facts from names such as `approve`, `finalize`, or `close`.

## 2. Derive the smallest complete operation set

Propose one dedicated operation for every interactive transition. Its procedure must conduct the human work
the interactive step requests; a generic "run the item" wrapper does not cover it.

Also propose a named operation for every automatic or routed transition so a person can deliberately select
that exact edge when auto-run is disabled or they do not want the router to choose. Several transitions may
share one operation only when they express one human intent and the operation still selects an exact
transition without guessing.

Treat the installed intervention command as the default cross-workflow **unblock** operation when it can call
`list_interventions` and act on every server-authored action. Add a project-specific triage operation only
when it contributes real project semantics such as a stable review grouping or a domain-specific interview;
do not duplicate the generic queue merely to rename it.

Human-started manual transitions may receive dedicated operations when their consequences or frequency make
that useful. Otherwise the generic intervention operation may surface them. Propose scripts only for
repeatable deterministic repository-file discovery or validation; scripts never mutate Workflows state or
replace MCP calls.

Prefer a transition's stable, action-oriented name. If two workflows would produce the same operation name,
prefix both with a short workflow slug. Never shadow an unmanaged operation or an installed Workflows
operation; propose a distinct name or an explicit amendment instead.

## 3. Present one reconciliation plan

Before writing, show a compact coverage table with every transition that is interactive, automatic, routed,
or already covered by an existing workflow operation. Then present exactly these groups:

- **Create** — new operation, transitions covered, harness surfaces, and why it is useful;
- **Amend** — existing operation, the concrete gap, and the files that would change;
- **Keep / reuse** — existing operation and the coverage it already supplies;
- **Remove** — obsolete managed operation and the live-definition fact that made it obsolete;
- **Conflicts** — unmanaged files, naming collisions, or ambiguous consolidation decisions.

Include `.workflows/operations.json` in the proposed file set. State that generated transition procedures
always load ancestors, peers, dependencies, dependants, artifacts, and dialogue before acquiring a claim or
recommending a decision.

Ask for approval of this exact plan. An invocation of this skill is approval to inspect and propose, not to
write or delete repository files. If any workflow's `definitionStamp` changes before approval, refresh all
definitions and re-present the affected plan.

## 4. Apply only the approved reconciliation

Use the layouts and content contract in the reference. Write every approved operation to one canonical
procedure under `.workflows/operations/`, then create only the approved harness wrappers. Keep wrappers thin;
transition intelligence has one source.

Create or update `.workflows/operations.json` last. It records the definition stamps used, the operation to
transition mapping, and every file this generator owns. Never put credentials, machine paths, item state, or
artifact content in it.

An unmanaged file is user-owned. Change it only when the approved plan names that exact file and preserves
its unrelated content. Remove only files listed as managed in both the existing marker and manifest, and
only when the approved plan names each removal. Never edit installed plugin files or global harness config.

## 5. Verify the repository result

Re-read all generated files and verify:

- every live interactive transition has a dedicated operation;
- every live automatic and routed transition has an exact manual trigger or an explicitly approved shared
  operation;
- every generated transition procedure contains the complete context envelope from the reference;
- wrapper names and links resolve for every approved harness;
- no unmanaged file changed and no unapproved file disappeared;
- the manifest's definition stamps still match fresh `read_workflow_definition` results.

If a stamp changed, leave the written files in place, report them as stale, and propose the refresh; do not
silently regenerate from a definition the user did not approve. Finish with the operations created, amended,
kept, and removed, plus any transition still lacking approved coverage.
