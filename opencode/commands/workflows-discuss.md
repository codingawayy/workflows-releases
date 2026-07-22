---
description: Talk to one workflow item — load its documents, Q&A history, and place in its pipeline, discuss it with the user, and carry any amendment back through the safest path (Q&A guidance, a manual move, a stepped transition, or a claim-respecting document rewrite). Invoke when the user wants to discuss, understand, question, or amend an item in conversation (e.g. "/workflows-discuss B26.070", "talk to I26.002 about its proposal", "why is this item where it is?").
---

Discuss item `$ARGUMENTS` using the instructions below.

The argument is the item id (e.g. `B26.070`). If none was provided, ask the user which item they want to discuss.

# Discuss an item

Hold a conversation about one workflow item, grounded in what the store actually holds — its
documents, its Q&A thread, its position in its workflow. Reading never contends with a run, so an
item can be discussed in **any** state, even mid-run. Writing can contend: the store refuses a
write while another run holds the item (`RunClaimHeld`), so amendments go through the paths in §3
and the refusal handling in §4 — never around them.

## 1. Load the item (claim-free)

Use the item id the user gave (e.g. `/workflows-discuss B26.070`). If none was given, ask — don't guess.
Then load, without taking any claim:

- `get_item` — status, the available `moves`, dependencies, children, parent, and the claim
  fields (`claimed_by`, `claim_phase`).
- `read_artifact` for each name in `artifacts` — the item's actual deliverables.
- `get_questions` — the full dialogue log (clarifications, steer reasons, notes), oldest-first.
- `read_workflow_definition` for the item's workflow — to place its status in the pipeline and
  explain what each available move would do.

If `claimed_by` is set, a run is live: say so up front — discussion is unaffected, but amendments
will be refused until the run ends (`moves` also comes back empty under a live claim).

## 2. Discuss

Answer from the loaded material — quote the documents, use the workflow's own vocabulary for
statuses and transitions, and say plainly when something isn't recorded rather than inferring it.
Explain *why* the item is where it is (which transition produced what, what a move would change)
from the definition, not from guesswork. Related items that come up are one `get_item` away.

## 3. Amend

Transitions produce documents — that is the product's spine — so prefer the amendment path that
keeps the generative act inside the pipeline. In order of preference:

1. **Record guidance** — the user wants a future (re-)run to know something: `append_question`
   (`answer` when they're answering an open question, `note` for a free-form steer).
2. **Take a manual move** — one of the `kind: "manual"` entries in `moves` (there is no separate
   "send back" operation — going back is just another manual move):
   - a terminal move (a drop/reject) → `drop_item` with the rationale;
   - any other manual move (park, re-open, redo) → `set_status` to the move's `statusOut`, with
     `reason` set to the user's rationale (recorded as an audited Q&A round). This only moves the
     status — a move's `removes` (the documents it would delete) applies solely when a human takes
     that declared edge from the board; `set_status` leaves documents in place until whatever
     produces them runs again, so say so if the user expects an immediate clear;
3. **Conclude in a stepped transition** — the discussion decided real pipeline work should run (a
   `kind: "stepped"` move): with the user's go-ahead, call `next_step({ item, transition })` and
   follow each response's `guidance` verbatim, submitting via `submit_step` as it directs — the
   server owns the sequencing. The user is present, so work any interactive step through with them.
4. **Rewrite a document directly** — `write_artifact` with a required `reason`. This is the
   exception path: use it when the change is genuinely an *edit to an existing deliverable* the
   discussion has settled, not new work a transition should produce. Show the user what you're
   about to write (the change, not necessarily the whole document) and get their yes first. The
   `reason` (a one-line why) is mandatory and is recorded structurally: the system writes an `edit`
   round on the Q&A thread and the board badges the document as edited outside a transition — so
   you no longer hand-write a separate provenance note, the reason IS the record.

## 4. When a write is refused

A `RunClaimHeld` refusal means a live run holds the item. Report who holds it (`claimed_by`,
`claim_phase`), keep the discussion going, and offer to retry the write once the run ends.
Never call `release_claim` or stop the run just to push a chat write through — interrupting a
run is the user's explicit decision, not a write-retry tactic.
