---
name: workflows-unblock
description: Use when inspecting a project's human-intervention queue, reviewing selected waiting items, or guiding the user through one intervention at a time.
---

# Unblock project work

Inspect the configured project's intervention snapshot, then help the user take one explicit, fenced action at
a time. Invoke this procedure as `/workflows-unblock` with optional item identifiers. The server supplies every current
cause, constraint, action, and explanation; do not invent a primary cause, reorder causes, or hide an
unfamiliar cause.

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
   `start-over` and explain that choice; omit `passChoice` only when no pass is open.
4. For `amend-routing-input`, call `read_artifact` again for the selected artifact and use that returned
   `stamp`, current content, the full proposed replacement content, and one non-blank reason.

If the action disappeared, its effects/blockers changed, its transition is absent from current `moves`, or a
blocker is present, do not ask for approval from stale facts. Report the change and return to one-item
deliberation. Otherwise state, in one approval prompt: the exact action and item, every current effect and
blocker (including an empty blocker list), the exact rationale, and any pass choice or full document change.
Proceed only on an explicit yes to that complete prompt; an invocation or earlier general instruction is not
approval.

## 4. Execute only the approved action

Map only these known action kinds. The operation that owns the change also owns its audit; never add a second
`append_question`, raw `set_status`, or other duplicate write.

- `unblock-stop` — call `unblock_item` once with the refreshed `allowVersion`, rationale, and a pass choice
  only when the refreshed `unblock.pass` is present. This is the only stop action. It neither launches a run
  nor changes status or published artifacts;
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
