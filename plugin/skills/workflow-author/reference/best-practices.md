---
covers: how to shape each part of a workflow well — transitions, step prompts, documents, routing, terminal outcomes, the input contract
---

# Authoring best practices

The thesis behind every rule here: **the human steers at a distance through the documents the workflow
produces.** A good workflow is one where a non-expert can supervise by reading the documents, not by
watching the agent work. Shape for that.

## Transitions — the spine

- **One transition = one meaningful change of state** that lands a document a later transition or a human
  reads. If a transition produces nothing anyone consumes, it shouldn't exist.
- **Fewest transitions that do the job.** Each is latency and a place to get stuck. A three-stage
  pipeline beats a seven-stage one unless the extra stages each earn a document.
- **Name the status for the state the item is IN** (`analyzed`, `designed`, `planned`), not the action.
  Name the transition for the action (`analyze-item`, `finalize-design`).
- **Put the human gate where a decision can't be delegated** — go/no-go, a product decision, sign-off.
  Make that transition `interactive` (human-led). Everywhere else, prefer autonomous (`auto`) so the
  pipeline flows on its own.
- **Mark autonomous forward transitions `auto`** so the engine advances them. Leave an autonomous
  transition *unmarked* only when a human must decide WHEN to start it (closing an epic is the standing
  example: the work is autonomous, but a human picks the moment).

## Step system-prompts — where the real quality lives

A step's `systemPrompt` is the role and the method; the `userPrompt` is the concrete task and the paths.
This prose is the single biggest lever on output quality. Write it as you would brief a capable
colleague who starts cold:

- **State the job and the cold start.** Each step is a fresh agent with no memory of prior steps —
  tell it what to read (`{{problem}}`, `{{in:01-...}}`) to ground itself.
- **Say what "good" looks like and when to stop.** Give the bar, not just the task. Scale effort to the
  item: "length should match the item's size; bias briefer."
- **Make the deliverable structure explicit.** If the step writes a document, specify its sections and
  who reads it. A document with a known reader and shape is one a human can steer from.
- **One concern per step.** If a step's instructions need "and" to join unrelated jobs, split it.
- **Return structured facts the control plane branches on.** A step's `schemaJson` should surface the
  booleans/arrays that drive `runWhen`/`over`/`gateField` (e.g. `reviewApplies`, `lenses`, `mustFix`).

## Documents — the steering surface

- **`produces` artifacts** are per-item deliverables (the `problem`, `analysis`, `summary`). Declare one
  per transition that lands a deliverable; name them in domain terms.
- **`documents`** are cross-item, workflow-scoped, versioned knowledge (a running `lessons` doc). Use
  one when the workflow should improve run-over-run. A `leaf`/`interactive` step rewrites it via
  `writesDocument` (the engine writes it conflict-safely).
- Keep each document's audience and purpose distinct — don't make one document serve two readers at two
  altitudes; split it.

## Router conditions — branching

- Add `routerCondition` to an exit only when a status genuinely **forks** on a judgment the agent makes
  at run time. Write the condition as the prose a chooser reads ("the work needs redoing", "the user
  opted out").
- A status with an `auto` exit must not also carry routed exits (the auto exit is unconditional). Either
  a status auto-advances, or it forks on conditions, or it waits for a human — pick one.
- **Backward edges** (`invalidates`) model rework: an edge back to an earlier status that clears the
  documents whose producers must re-run. List exactly the documents that go stale.

## Terminal outcomes — how work ends

- Model every way an item legitimately ends. A natural completion is a `produces` terminal (an item
  lands at `implemented`/`closed` with its final document). An abandonment is a **drop edge**: a
  step-less manual transition to a terminal status (`dropped`) that a human takes from the board.
- A terminal a transition *produces* (e.g. `dropped` reached by a `drop` edge) is NOT an
  `offChainStatus`. Use `offChainStatuses` only for a status no transition touches.
- Don't over-model endings. One drop path and one natural completion cover most workflows.

## The input contract — always author it

Every workflow you build or amend must carry both contract fields. They are what the intake path routes
and records against; a workflow without them gives intake no bar and degrades recording quality.

They are plain workflow-registry metadata (the same row as name / description / prefix), NOT part of the
revision. Set them via `create_workflow` (a new workflow) or `update_workflow` (an existing one) — both
take `itemEntryCriteria` and `entryDocumentGuidance`; on `update_workflow` a null/blank value clears the
field. Editing the input contract is saved in place and **never forks a revision** — it is independent of
`set_workflow_definition`.

- **`itemEntryCriteria`** — the admission predicate: what kinds of item belong here, by **type AND
  size** (e.g. bug / feature / refactor / epic; small / large). Write it so a router can tell, from an
  item's description, whether it fits — and what to do when it doesn't (too large → an epic; a different
  kind → another workflow).
- **`entryDocumentGuidance`** — what a good entry document for this workflow contains: the sections, the
  bar (e.g. "specific, verifiable acceptance criteria, not 'looks better'"), and what to avoid
  (prescribing the solution). This is the recording-quality target the intake path writes to.

Both are inert prose — the store never enforces them. They exist to make a non-expert's recording good
by default, which is the whole point: item-recording quality is the input contract to everything
downstream.
