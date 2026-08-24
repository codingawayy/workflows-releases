---
name: workflows.unblock
description: Use when inspecting a project's human-intervention queue, reviewing selected waiting items, guiding the user through one intervention at a time, or interviewing the user through the decisions of many waiting items in one batch review.
user-invocable: false
---

## Required Workflows MCP capability

Before starting **inspect and resolve workflow interventions**, confirm that the MCP server named `workflows` and its bootstrap tools are available. If the server is absent or failed to initialize, stop before loading another procedure or beginning the operation.

Before business work, call `select_repository` once — exactly once — with the active repository's absolute root path. This is the only Workflows tool call allowed as a readiness probe. If this entry's command wrapper already completed the same probe, treat it as complete and do not call it again. If selection fails, stop.

On any readiness stop, do not use the repository CLI, browser automation, raw HTTP or API calls, or another write path. Use a different interface only when the user explicitly requests that specific interface; that request grants no authority beyond existing product boundaries.

Respond with these four lines and do not claim partial success:

```text
MCP: Workflows (`workflows`)
Operation: inspect and resolve workflow interventions
Outcome: not started
Recovery: Restore the `workflows` server, use `/mcp` to retry it, and start a fresh Claude Code session only if the repaired tool catalog is not visible in this session.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

# Unblock project work

Inspect the configured project's intervention snapshot, then help the user take one explicit, fenced action at
a time. Invoke this procedure as `/workflows:unblock` with optional item identifiers. The server supplies every current
cause, constraint, action, and explanation; do not invent a primary cause, reorder causes, or hide an
unfamiliar cause. When the user asks to review many waiting items together, use **batch review mode**
(section 7): it interviews first, collects every decision, and writes only at the end.

## 1. Choose the item without writing

Call `list_interventions` once. This initial list and the choice of mode are read-only: do not change an item,
document, dialogue, claim, pass, routing decision, repository, or session state. Deferring an item means only
remembering that choice in this conversation.

- With item identifiers, use **selected-item mode**. Preserve every supplied identifier, duplicate, and its
  order; process no substitute or newly surfaced item. Compare each identifier with the current intervention
  list. For one that is missing from the list, call `get_item`: report it as absent when no item is returned,
  terminal when `archived_at` is present or `status_category` is `completed`/`cancelled`, and otherwise not
  waiting. Continue with the next supplied identifier only after reporting or finishing the current one.
- Without identifiers, use **guided mode**. Take the first current entry, name the item and every reason it is
  waiting, and introduce no other item until this one is resolved or explicitly deferred in the conversation.
  If the list is empty, report that no project item currently needs human intervention and stop.
- When the user asks to review several waiting items as one interview — a themed sweep such as every item
  waiting at one status or under one parent, or multiple identifiers with a request to decide them together —
  use **batch review mode** (section 7). Several identifiers to process independently, in order, stay in
  selected-item mode.

## 2. Load one waiting item

For the current item, call `get_item`, `read_artifact` for every artifact it names, and
`get_item_supervision`. Keep every artifact stamp. The supervision result supplies the dialogue rounds, stop
history, any active stop, timeline, and recovery facts; `get_item` supplies the claim. Explain unfamiliar
status, cause,
constraint, and action terms from the server-authored descriptions. Ask one material question at a time.

Treat every cause as independent. Show every legal human transition even when an active stop coexists, and
show each action's complete `effects` list. An unknown cause or action remains visible but is
unsupported by this installed package; do not infer an operation for it.

## 3. Obtain an exact decision

Offer every server action, including every `choose-transition` action when a stop coexists. Do not prioritize
one cause as the item; explain the complete `effects` and `blockedBy` arrays on each relevant descriptor. Ask
one material question at a time until the user has selected one exact action and supplied its rationale.

Immediately before asking for final approval, refresh this item:

1. Call `list_interventions` and extract only this identifier's current entry. Do not introduce other entries.
2. Call `get_item` to refresh its status, claim fields, artifact names, and current `moves`.
3. For `unblock-stop`, call `get_item_supervision` and use only the current open episode's
   `unblock.allowVersion`. If `unblock.pass` is present, require the user to choose exactly `keep` or
   `start-over` and explain that choice; omit `passChoice` only when no pass is open. Separately offer the
   optional launch-repair prompt: blank means ordinary scheduling, while non-blank text is stored verbatim
   and authorizes exactly one repair attempt before one workflow launch. It does not choose a machine or
   alter Keep versus Start over.
4. For `amend-routing-input`, call `read_artifact` again for the selected artifact and use that returned
   `stamp`, current content, the full proposed replacement content, and one non-blank reason.

If the action disappeared, its effects/blockers changed, its transition is absent from current `moves`, or a
blocker is present, do not ask for approval from stale facts. Report the change and return to one-item
deliberation. Otherwise state, in one approval prompt: the exact action and item, every current effect and
blocker (including an empty blocker list), the exact rationale, any pass choice or full document change, and
either the exact launch-repair text or that no repair is authorized.
Proceed only on an explicit yes to that complete prompt; an invocation or earlier general instruction is not
approval.

## 4. Execute only the approved action

Map only these known action kinds. The operation that owns the change also owns its audit; never add a second
`append_question`, raw `set_status`, or other duplicate write.

- `unblock-stop` — call `unblock_item` once with the refreshed `allowVersion`, rationale, the approved
  `repairPrompt` only when non-blank, and a pass choice only when the refreshed `unblock.pass` is present.
  This is the only stop action. It neither launches a run nor changes status or published artifacts;
  it may also clear the matching routing decline named in `effects`.
- `retry-routing` — call `retry_routing` once with the descriptor's exact `expectedRoutingVersion` and the
  approved non-blank rationale. It changes no routing input and launches no run.
- `amend-routing-input` — call `write_artifact` once with the selected artifact, full approved content,
  refreshed optimistic `stamp`, and rationale. The edit itself invalidates the decline. A routing-input edit
  and `retry_routing` are alternatives: never perform both for one approval.
- `choose-transition` — match the descriptor's transition to the refreshed `get_item.moves`. For a `manual`
  move, call `take_move` once with its exact name and rationale. For a `stepped` move, call `next_step` once
  with the exact item and transition, then follow each returned instruction through the normal
  `next_step`/`submit_step` loop. The transition's own operation records its outcome; do not add a duplicate
  dialogue round. A stop does not suppress either kind of transition.
- `owner-handoff` — perform no mutation. Name the exact recorded `input.machine`, what that owner must
  reconcile, and every cause in `effects`. Never release the claim, inspect or mutate Git or worktrees, claim
  completion, or offer a source-checkout command as a customer remedy.

An unknown cause or action remains visible with its server explanation and is `unsupported`; never guess a
tool mapping. A non-empty `blockedBy` list likewise authorizes no write.

## 5. Handle a fence refusal

A typed refusal, artifact conflict, changed status or move set, changed active-stop confirmation or
`unblock.pass`, changed claim,
or changed routing version means the approved action did not land. Report the precise stale fact. Refresh the
intervention entry and only this item's affected context: `get_item`, plus `get_item_supervision` for a stop or
`read_artifact` for an edit. Do not auto-retry, auto-merge, switch routing remedies, or silently choose a new
transition. If an action is still possible, disclose its current effects/blockers and obtain a new exact
decision and approval.

## 6. Continue or report

In guided mode, after an action resolves the current topic or the user explicitly defers it in this
conversation, call `list_interventions` again before naming another item. Do the same after an owner handoff or
unsupported result if the user chooses to continue. Do not persist deferral anywhere. In selected-item mode,
refresh as needed to classify the next supplied identifier, but never substitute another item.

Call `list_interventions` once more for the final project snapshot, then end with a concise ledger. Keep all
seven headings even when empty:

- **Resolved** — landed actions and the cause effects they cleared.
- **Deferred** — conversation-only deferrals.
- **Changed** — items whose refreshed fence or action facts invalidated a decision.
- **Unsupported** — unknown action/cause kinds with no guessed mutation.
- **Owner-bound** — exact machine handoffs still owned elsewhere.
- **Newly surfaced** — identifiers in the final project snapshot that were absent initially; report but do not
  process them in selected-item mode.
- **Still waiting** — every identifier remaining in the final snapshot and its current authorized next step,
  or a plain statement that none is available.

## 7. Batch review mode

One read-only interview across many waiting items, then one approved apply pass. The fences are the same as
in the one-item loop; only the conversation shape changes. Nothing is written before the apply phase.

### Survey

Call `list_interventions` once and keep only the items in the user's requested scope. Group the scoped items
by parent item first, then by status and cause.

### Investigate before asking

Build a brief for every scoped item from `get_item`, the artifacts it names, and its dialogue. Keep each
item's reading isolated so one item's documents cannot color another item's brief; run the readings in
parallel when the harness offers isolated helpers, otherwise one item at a time. Each brief states the
problem, the proposed design, every open decision with its recommendation and stakes, what is already
settled, and any relation to another scoped item.

Facts are yours to find, never the user's. Verify freshness before interviewing: confirm the code and items
each analysis names still exist, and check whether work that landed after the analysis touched the same
area. An item whose analysis facts have moved gets a handling question — re-analyze, proceed with a recorded
caveat, or drop — instead of its design questions.

### Interview in rounds

Open with one big-picture table: the groups, their items, and their open-decision counts. Then work the
decisions as a tree in rounds. The frontier is every question whose prerequisites are settled; a question
that depends on another question still open waits for a later round. Ask cross-item questions first —
sequencing, ownership, shared code — because their answers reshape the rest. Then take one themed group per
round, so each round stays small.

Number questions consecutively across the whole session and format each as:

```
❓ **Q<n>** - **<title>**: <the question, with its concrete options>

➡️ <recommended answer, with the one-line reason>
```

Use plain wording with a concrete referent, and explain workflow and server terms from their own
descriptions. If investigation exposed a question the item's documents never asked, ask it rather than
silently assume. Record every answer; write nothing yet.

### One apply plan, one approval

When the frontier is empty, present the complete apply plan: per item, the exact action as section 4 maps
it, the collected decisions it records, the rationale, and its effects. Order the plan deliberately: advance
items before adding dependency edges between them, because an unmet edge blocks advancement; and when an
item's own gate cannot pass yet, plan to park its collected decisions as one dialogue note instead of losing
them. Ask for one explicit yes to the whole plan; an invocation or earlier general instruction is not
approval.

### Apply with the existing fences

Execute the plan item by item under the rules of sections 3–5, with one change: the approved plan replaces
the per-action approval prompt. Immediately before each item's write, refresh that item exactly as section 3
directs; if any fence fact moved — the action, its effects or blockers, the move set, an active stop or
pass, the claim, or an artifact stamp — report the precise stale fact and skip the item. Never silently
adapt, reorder remedies, or substitute an action. When a transition's own steps ask for the human's
decisions, supply the collected answers; do not re-interview the user.

Close with the section 6 ledger; a skipped item reports under **Changed** with its stale fact.
