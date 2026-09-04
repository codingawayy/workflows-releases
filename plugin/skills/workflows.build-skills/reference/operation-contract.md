# Generated workflow operation contract

Use this contract for every repository operation produced by `build-skills`.

## Ownership and layout

The repository has one canonical procedure per operation:

```text
.workflows/operations/<operation>.md
```

Add only the harness surfaces approved by the user:

```text
.claude/commands/<operation>.md
.claude/skills/<operation>/SKILL.md
.agents/skills/<operation>/SKILL.md
.opencode/commands/<operation>.md
.opencode/skills/<operation>/SKILL.md
```

The canonical procedure contains the complete behavior. Each skill or command wrapper identifies its native
invocation and tells the agent to read and follow that canonical file, passing through the user's item IDs
and choices. A wrapper must not carry a second copy of transition logic.

Put this marker immediately after frontmatter in every generated file, changing the values but not the key
names:

```html
<!-- workflows.build-skills: {"schema":1,"operation":"finalize-design","role":"canonical","workflow":"Backlog","transition":"finalize-design"} -->
```

Use `role` values `canonical`, `claude-command`, `claude-skill`, `codex-skill`, `opencode-command`, and
`opencode-skill`. A consolidated operation may use arrays for `workflow` and `transition`. The marker means
the generator owns the file only when `.workflows/operations.json` names the same path and identity.

The manifest shape is:

```json
{
  "schema": 1,
  "generatedBy": "workflows.build-skills",
  "definitions": { "Backlog": "<definitionStamp>" },
  "operations": [
    {
      "name": "finalize-design",
      "covers": [{ "workflow": "Backlog", "transition": "finalize-design" }],
      "files": [".workflows/operations/finalize-design.md"]
    }
  ]
}
```

Sort definition keys, operations, coverage entries, and file paths for reviewable diffs. Paths are
repository-relative with `/` separators. Record no content hashes: the marker-plus-manifest ownership check
keeps a person free to edit a generated file, while reconciliation can show the actual diff before replacing
it.

## Native wrapper conventions

Match nearby project-local files before choosing frontmatter. At minimum:

- Files under `.claude/commands/` carry `description` and an item-oriented `argument-hint` when supported.
- Skills under `.claude/skills/` and `.agents/skills/` carry `name` and a discriminating `description`.
- Commands and skills under `.opencode/` use that harness's project-local filename convention.
- Every relative link from a wrapper to `.workflows/operations/<operation>.md` must resolve from that
  wrapper's directory.

Keep the operation model-invocable when its intent is recognizable from ordinary language. Use an
explicit-only invocation only when the user requests that policy.

## Required context envelope

Every canonical transition procedure must complete this read pass before it acquires a claim, takes a move,
or recommends a decision:

1. Call `get_item` for every target and read every named artifact plus the complete Q&A or supervision
   dialogue.
2. Call `list_items` once for a complete compact project map. Walk each target's `parent_id` recursively to
   the root with `get_item`; read the ancestors' artifacts that define outcomes, boundaries, or decisions.
3. Identify every direct sibling from the project map. Load each sibling's item record, then read the
   artifacts of siblings whose scope, decisions, dependencies, or ownership overlap the target. When an
   ancestor divides work among children, account for every sibling before recommending a boundary change.
4. From the target record, inspect every dependency, dependant, and open child that can constrain scope or
   sequence. Follow another level only when the relation is material to the transition's decision.
5. For a batch, finish this context pass for every target before interviewing or starting any transition.
   Reuse shared reads, and refresh a later target when an earlier result changes shared context.

The procedure's orientation to the user states the target's current status, its parent outcome, its role
among peers, and the exact named transition about to run. It surfaces stale premises, scope collisions, and
ancestor conflicts rather than inheriting them silently.

When the transition asks for product, architecture, repository, or external facts, inspect the repository's
current authority and source before recommending. The ticket's existing analysis is a hypothesis, not proof.

## Driving a stepped transition

Confirm from the live definition that the exact transition still consumes the target's current status. Call
`next_step` with the target item and exact transition name on the first call only. Follow the returned
instructions and guidance one step at a time, writing only the server-designated working files and returning
schema-conforming results through `submit_step`.

For an interactive step:

- explain the decision in the target's ancestor and peer context;
- ask one material question at a time and keep the exact answers for the step result;
- research factual premises instead of asking the user to supply discoverable facts;
- require an explicit override when a choice conflicts with an ancestor or settled peer boundary;
- never fabricate human input in an unattended session.

Do not write transition-produced artifacts with `write_artifact`. The step loop owns its working copies,
publication, audit, and status change. Handle `saved`, `invalid`, conflicts, pauses, lost claims, and other
states exactly as the server response directs.

When the selected transition returns `done`, stop. Do not call `next_step` again: that could acquire the next
transition, which this named operation was not invoked to run. Verify the final item and absent claim with
`get_item`. If the user abandons an attended execution before publication, call `cancel_execution` and report
that the prepared work was discarded.

## Taking a manual transition

Refresh `get_item`, match the exact transition in `moves`, explain its current status, output status,
`removes`, and relevant ancestor/peer consequences, and obtain a non-blank rationale. Use `take_move` once
with that exact name. A stale or refused move returns to deliberation; never substitute `set_status`.

## Automatic and routed transitions

A named operation is the person's explicit selection of that edge. It may deliberately invoke an
`auto: true` transition or bypass router choice by naming one routed transition, but it still uses the normal
`next_step` / `submit_step` protocol and stops at that transition's `done`. If the definition or item status
no longer offers the selected edge, report the stale operation and run `build-skills` again; never let the
router silently choose a replacement.
