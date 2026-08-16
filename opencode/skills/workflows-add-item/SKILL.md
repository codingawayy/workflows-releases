---
name: workflows-add-item
description: Use when capturing work in the tracker, breaking it into items, routing each item to a workflow, or recording dependencies and epics.
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
Recovery: Restore the Workflows bundle and registration, reload or restart OpenCode, and start a fresh session if the repaired tool catalog remains stale.
```

After readiness succeeds, follow the existing procedure. A later domain or tool error from a business tool is not a bootstrap failure; keep the procedure's existing error handling.

# Add item — record work into the right workflow, to spec

Recording work is the highest-leverage quality lever in the system: every item is the cold-start input
to an autonomous pipeline, so a badly recorded item poisons every stage downstream. Your job is to take
natural-language work and turn it into items that are **broken down right, routed to the right
workflow, and recorded to that workflow's quality bar** — then record them only on approval.

Act like a tech lead curating the whole tracker, not a form-filler. **Propose first; record nothing
until approved** (see *The human gate* for what "approved" means per caller).

## The loop

For the work you're given, run this loop. Routing comes **first** — you record to a workflow's bar, so
you must know the workflow before you write the document.

1. **Break the work down.** Decompose into items that are each a single coherent change with one
   verifiable outcome. Follow `reference/breakdown.md`.
2. **Route each item — first.** Read `list_workflows` (names, descriptions, and each workflow's
   `itemEntryCriteria` + `entryDocumentGuidance`). Match the item's **type and size** against each
   workflow's `itemEntryCriteria` and pick the best fit. Follow `reference/routing.md`.
3. **Record to spec.** Build the item's entry document(s) — a workflow may declare more than one (e.g. a
   Backlog item's `problem` + `criteria`) — to the chosen workflow's `entryDocumentGuidance`, its sections,
   its bar. The guidance is the recording target, not a suggestion.
4. **If the fit breaks while recording, reconsider from the start.** If writing the document reveals the
   item doesn't actually belong in that workflow (it's larger than you thought, or a different kind),
   loop back to step 2 — don't force a misfit through.
5. **Handle non-conformance with the ladder.** When an item doesn't cleanly fit: **enrich** the document
   to conform → **redirect** to a better-fitting workflow → **ask** the human. When *nothing* fits,
   route to the closest workflow and **nudge** ("consider a workflow for X") — never auto-create one.
   Details in `reference/routing.md`.
6. **Survey the rest of the tracker.** Before proposing, read the in-flight items: dependencies in both
   directions, anything a new item supersedes or overlaps. Follow `reference/breakdown.md`.
7. **Propose, then gate.** Present the breakdown — a table (label · title · scope · **target workflow** ·
   depends-on), the dependency picture, the epic each cluster sits under — and **wait** for the gate.
8. **Record on approval, in dependency order.** Create each item after its dependencies exist, with its
   WHOLE entry-document set and dependencies in the same `create_item` call (its `entryDocuments` map —
   no document is ever written by a follow-up `write_artifact`); file each under its epic via `parentId`.
   Follow `reference/breakdown.md`.

## Sizing for cold-start

Size each item's grain so a fresh agent (or a human) can pick it up from its entry document **with no
other context**. Grain is decided by what makes a clean cold-start unit — *not* by how it will be
executed. How it runs (agent-driven, human-driven, in a checkout that can run the code) does not branch
the grain decision;
record the same well-scoped item regardless of who or what will work it.

## The human gate — proportional, and per caller

The gate scales to the risk of the action, and **who** is recording changes how an ungated step is
handled:

- **Routing an item into an existing workflow** is low-risk — show it, don't block on it.
- **Creating or amending a workflow, or a loose/forced fit**, is high-risk — it needs explicit approval.

Recording items needs **write access** to the tracker, so it records only from a write-capable
context. Two such callers share the same loop:

- **An interactive human** in the terminal — present the full proposal and wait for confirmation before
  recording. Surface the gated decisions (a loose fit, a workflow that should exist) one at a time.
- **An interactive workflow step** (a human-led step — e.g. a triage or closeout step a human runs to
  capture outstanding work) — the human is present, so it records like the case above, just seeded from
  the work the step surfaced rather than a fresh request.

When the skill runs in a context **without** write access — notably an autonomous workflow step, whose
tracker tools are read-only by construction — it can still break the work down, route it, and draft the
proposals into its own output document, but it must **not** claim to record them. Recording is deferred
to the next write-capable context (a human, or an interactive step). Don't pretend an item was created
when the caller couldn't create it.

## Reference

This skill's `reference/` folder holds the detail, one file per concern (`breakdown.md`, `routing.md`).
Each file's `covers:` frontmatter line says what it documents — read the one whose `covers:` matches
the step you're on rather than loading both up front.
