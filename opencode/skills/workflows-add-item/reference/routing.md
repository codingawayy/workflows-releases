---
covers: routing an item against the workflow input contract, recording to the document guidance, the non-conformance ladder, and the nothing-fits nudge
---

# Routing and recording to spec

A workflow declares its **input contract** — two prose fields you read from `list_workflows`:

- **`itemEntryCriteria`** — what kinds of item belong in this workflow, by **type and size**. This is
  what you route against.
- **`entryDocumentGuidance`** — what a good entry document for this workflow contains. This is what you
  record to.

Either may be null (the workflow declares no bar). When a workflow has no criteria, fall back to your
own judgment of fit; when it has no guidance, record to the universal principles in `breakdown.md`.

## Route first

For each item, before writing anything:

1. Read every workflow's `itemEntryCriteria`.
2. Match the item's **type** (bug / feature / refactor / epic / …) and **size** (a one-sitting change vs
   a multi-week initiative) against each workflow's criteria.
3. Pick the best fit. When two fit, prefer the more specific criteria. When the item is itself a large
   container of other work, route it to the planning/epic workflow and create the pieces under it.

Only once the workflow is chosen do you build the entry document — to **that** workflow's guidance.

## Record to spec

Build the entry document to the chosen workflow's `entryDocumentGuidance`: its sections, its bar, what
it says to avoid. The guidance is the target, not a hint. If the guidance asks for "specific, verifiable
acceptance criteria" and you can only write "make it better", the item isn't well-defined yet — keep
refining the item (or ask) before recording, rather than recording something that fails the bar.

**If the fit breaks while recording, reconsider from the start.** Writing the document to the bar often
reveals the truth about an item — that it's really an epic, or a different kind of work than it first
looked. When that happens, loop back to routing; don't force a document that doesn't fit the bar.

## The non-conformance ladder

When an item doesn't cleanly conform to a workflow's contract, climb this ladder in order — escalate
only when the rung below can't resolve it:

1. **Enrich** — improve the entry document so it conforms to the guidance (add the missing acceptance
   criteria, sharpen the scope, split an "and"). Most non-conformance is a recording gap, fixable here.
2. **Redirect** — if enriching can't make it fit because it's the wrong *kind* or *size* for this
   workflow, route it to a better-fitting workflow and record to that one's guidance instead.
3. **Ask** — if neither enriching nor redirecting resolves it (the work is genuinely ambiguous, or spans
   workflows in a way only the human can settle), surface the question to the human. Without write access
   (an autonomous context), draft the item with the open question called out, for a human to settle and
   record.

## When nothing fits

If no workflow's criteria fit the item at all:

- Route it to the **closest** workflow and record it there.
- **Nudge**, don't build: note that a dedicated workflow for this kind of work might be worth creating
  ("consider a workflow for X"). Surface the nudge to the human (or, without write access, carry it in
  the drafted proposal).
- **Never auto-create a workflow.** Creating or amending a workflow is a separate, gated, deliberate
  action — it is not part of recording an item.

## The proportional gate

Routing decisions carry different risk, and the gate scales to it:

- **Routing into an existing workflow that fits** — automatic. Show it in the proposal; don't block.
- **A loose or forced fit, or a nudge to create a workflow** — gated. It needs explicit human approval.

Record via the MCP write ops only once the applicable gate is satisfied — and only from a write-capable
context (see the SKILL's note on callers).
