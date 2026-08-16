---
name: workflows-discuss
description: Use when discussing, understanding, questioning, or amending one workflow item in conversation.
---

## Required Workflows MCP capability

Before starting **discuss and amend a workflow item**, confirm that the MCP server named `workflows` and its bootstrap tools are available. If the server is absent or failed to initialize, stop before loading another procedure or beginning the operation.

Before business work, call `select_repository` once — exactly once — with the active repository's absolute root path. This is the only Workflows tool call allowed as a readiness probe. If this entry's command wrapper already completed the same probe, treat it as complete and do not call it again. If selection fails or context-dependent project tools remain unavailable, stop.

On any readiness stop, do not use the repository CLI, browser automation, raw HTTP or API calls, or another write path. Use a different interface only when the user explicitly requests that specific interface; that request grants no authority beyond existing product boundaries.

Respond with these four lines and do not claim partial success:

```text
MCP: Workflows (`workflows`)
Operation: discuss and amend a workflow item
Outcome: not started
Recovery: Restore the Workflows bundle and registration, reload or restart OpenCode, and start a fresh session if the repaired tool catalog remains stale.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

# Discuss an item

Hold a conversation about one workflow item, grounded in what the store actually holds — its
documents, its Q&A thread, its position in its workflow. Reading never contends with a run, so an
item can be discussed in **any** state, even mid-run. Writing can contend: the store refuses a
write while another run holds the item (`RunClaimHeld`), so amendments go through the paths in §3
and the refusal handling in §4 — never around them.

## 1. Establish the full item context once (claim-free)

Use the item id the user gave (e.g. `/workflows-discuss B26.070`). If none was given, ask — don't guess.
Treat a continuous conversation about the same item as one discussion session. On the first turn
for that item, load this complete snapshot without taking any claim:

- `get_item` — status, the available `moves`, dependencies, children, parent, and the claim
  fields (`claimed_by`, `claim_phase`).
- `read_artifact` for each name in `artifacts` — the item's actual deliverables. Each returns
  `{ content, stamp }`; **keep the stamp** — rewriting that document later requires it.
- `get_questions` — the full dialogue log (clarifications, steer reasons, notes), oldest-first.

Retain that snapshot and subsequent tool results in the conversation. A later user message or
another invocation of this skill does **not** start a new discussion session: do not repeat these
reads merely because the user sent another message. If the user switches to a different item,
establish a new full snapshot for that item.

`get_item.moves` is the claim-aware source of truth for the current available actions. Call
`read_workflow_definition` only when the user asks about the wider pipeline and the retained
`moves` do not answer the question; retain that definition for the rest of the discussion session.

Refresh only the narrowest relevant part of the snapshot when the user asks for the latest state
or reports an external change, a tool reports a conflict or stale state, this session completes a
transition that may have changed information now being discussed, or required information has not
yet been loaded. After a known write, update the retained snapshot from the operation and its result;
do not read the same data back merely to confirm the write. Mark only unknown derived data stale —
for example, after `set_status` the new status is known, but call `get_item` if a later reply needs
the newly available `moves`.

If `claimed_by` is set, a run is live: say so up front — discussion is unaffected, but amendments
will be refused until the run ends (`moves` also comes back empty under a live claim).

## 2. Discuss

Answer from the retained session snapshot — quote the documents, use the workflow's own vocabulary for
statuses and transitions, and say plainly when something isn't recorded rather than inferring it.
Explain *why* the item is where it is (which transition produced what, what a move would change)
from the retained evidence, not from guesswork. Related items that come up are one `get_item` away.

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
4. **Rewrite a document directly** — `write_artifact` with a required `reason` and the `stamp` you
   read. This is the exception path: use it when the change is genuinely an *edit to an existing
   deliverable* the discussion has settled, not new work a transition should produce. Show the user
   what you're about to write (the change, not necessarily the whole document) and get their yes
   first. The `reason` (a one-line why) is mandatory and is recorded structurally: the system writes
   an `edit` round on the Q&A thread and the board badges the document as edited outside a
   transition — so you no longer hand-write a separate provenance note, the reason IS the record.
   The `stamp` is the one you got from `read_artifact` (`null` for a document that does not exist
   yet); the write lands only while it still matches, and there is no force-overwrite.

## 4. When a write is refused

A `RunClaimHeld` refusal means a live run holds the item. Report who holds it (`claimed_by`,
`claim_phase`), keep the discussion going, and offer to retry the write once the run ends.
Never call `release_claim` or stop the run just to push a chat write through — interrupting a
run is the user's explicit decision, not a write-retry tactic.

An `ArtifactConflict` means the document changed after you read it, so **nothing was stored**. Do not
retry the same body — it will be refused again. The error carries the document's current stamp AND its
current content: show the user what changed, merge the settled edit into that current content, and call
`write_artifact` again with the new stamp. You are the right place to resolve this — the user is present
and can say which version wins.
