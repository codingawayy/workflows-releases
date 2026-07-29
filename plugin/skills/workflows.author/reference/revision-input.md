---
covers: the typed `input` shape passed to set_workflow_definition — every field and how the pieces fit
---

# The typed authoring shape (`input`)

`set_workflow_definition` takes one object, `input`, the whole definition. `read_workflow_definition`
returns this exact shape for editing — read, modify the object, resubmit. The store derives the status
table and artifact ownership from what you declare; you never write those directly.

```jsonc
{
  "transitions": [ /* the state machine — the edges (see below) */ ],
  "artifacts":   [ "problem", "analysis", "summary" ],   // the {{...}} document namespace
  "offChainStatuses": [],                                // statuses NO transition touches (usually empty)
  "statusDisplay": [ { "name": "proposed", "label": "Proposed", "color": "#888", "ordinal": 0 } ] // optional board metadata + column order
}
```

The revision is PURELY the state machine (transitions, artifacts, offChainStatuses, statusDisplay). An
unknown top-level key is refused, not ignored.

## Workflow documents are NOT in `input`

A **workflow document** (cross-item knowledge like a lessons file) is shared by every revision and every
item and read fresh on each run — so no revision declares one, and `input` has no `documents` field.
A document name enters the workflow one of two ways:

- **`ensureDocuments`**, a sibling argument of `input` on `set_workflow_definition` — the names this
  write brings into existence. A name the workflow already owns is a no-op, so it is safe to repeat.
- **its own first write**, through `write_workflow_document`.

No step declares which document it writes: a document a step EDITS is written back because its content
moved. So a document is brought into existence by the write that first needs it:

```jsonc
{
  "workflow": "backlog",
  "ensureDocuments": [ "lessons" ],
  "input": { /* … a step whose prompt reads and edits {{lessons}} … */ }
}
```

Documents and artifacts share one flat `{{...}}` namespace, so a name may not be both.

The **input contract** (`itemEntryCriteria` + `entryDocumentGuidance`) is likewise NOT part of `input` —
it is plain workflow-registry metadata set separately via `create_workflow` / `update_workflow`
(see `reference/best-practices.md`), never through `set_workflow_definition`.

`statusDisplay[].ordinal` is the board's column order — omit it (or pass `null`) and the store fills it
from the graph's pipeline shape, so you never have to compute it by hand; set it explicitly only to pin an
order the derivation wouldn't produce on its own (e.g. a status reachable only through a rework edge).

## A transition (an edge in the graph)

```jsonc
{
  "name": "analyze-item",          // unique, kebab-case; this is the artifact-keyed name
  "ordinal": 2,                    // authoring/pipeline order (a board-order hint)
  "statusIn": "proposed",          // the status it leaves FROM; omit ONLY on the single entry transition
  "statusOut": "analyzed",         // the status it lands ON
  "label": "Drop",                 // optional button text for the move; blank = derived (see below)
  "execution": "shared",           // "shared" (default) or "isolated" (runs in its own git worktree)
  "description": "what this edge is for — short prose",
  "auto": true,                    // engine auto-advances through it when an item rests at statusIn
  "routerCondition": "...",        // prose the router reads to pick this edge among a status's exits
  "removes": ["analysis"],         // opt-in: documents this edge deletes when taken (never a movement condition)
  "produces": "analysis",          // the artifact this transition's deliverable IS
  "steps": [ /* ordered steps, for an autonomous transition */ ]
}
```

Key rules the store enforces (it rejects a violation with a clear message):

- **Exactly one entry transition** — the one with no `statusIn` (the store requires exactly one, with a
  `statusOut`). By convention it `produces` the entry document that `create_item`'s `entryDocument`
  writes — that coupling is applied by `create_item`, not the revision validator.
- **`auto` vs human-started.** `auto: true` marks a transition as its source status's automatic exit —
  the engine advances an item through it without asking. Omit `auto` and the status is *human-resting*:
  the engine never starts the transition on its own (a human does, e.g. from the board). A transition
  containing an `interactive` step is human-LED regardless (a human runs it) and must not be `auto`.
- **At most one `auto` exit per status.** A status may fork into several exits, but only one may be the
  automatic one. Multiple non-auto exits are `routerCondition`-routed (the router picks) or human-started.
- **`produces` names a declared artifact**, and each artifact is produced by at most one transition.
- **`removes` names declared artifacts to delete when the edge is taken** — an opt-in effect, never a
  condition of movement (there is no "backward edge" concept). An edge with none leaves its documents in
  place until a re-run overwrites them; the only guards are that each name is declared and none repeats.
- **`label` sets the move button's text** — the word a human reads to steer (a close action's "Drop" /
  "Abandon" / "Won't fix", a re-open, a "Continue"). Leave it blank and the read-model derives one (a
  humanized transition name, "Re-open {Status}" from the edge's target status, or the generic "Drop"). Keep it a **plain action verb**:
  the SAME string reaches both the board button and the agent (`get_item` `moves[].label`), and each
  surface adds its own framing — so never embed surface-specific phrasing like "Drop from the board". If two
  close actions on one status would read the same, the board auto-appends each one's destination.

## A step (one unit of an autonomous transition)

```jsonc
{
  "key": "01-analyze",      // natural key within the transition; over/runWhen/fixWith reference it
  "kind": "leaf",           // leaf | fanout | gate-loop | interactive
  "systemPrompt": "...",    // the step's instructions (its role + how to do the work)
  "userPrompt": "...",      // the task: what to read, what to write, the {{placeholders}} it uses
  "schemaJson": "{...}",    // JSON-schema string for the step's structured return
  "runProfile": "implementation",  // the repository run profile this step runs under; omit for the default
  // kind-specific:
  "over": "01-analyze.lenses",        // (fanout) the prior step's array field to fan out over
  "runWhen": "01-analyze.reviewApplies", // run this step only when a prior boolean field is truthy
  "fixWith": "01-implement",          // (gate-loop) the sibling step re-run to fix findings
  "gateField": "mustFix",             // (gate-loop) the array field whose emptiness ends the loop
  "maxRounds": 3                      // (gate-loop) the loop cap
}
```

**Step kinds:**

- **`leaf`** — one headless worker subprocess does the work and returns structured JSON. The workhorse.
- **`interactive`** — a human and the agent work it through in conversation (makes the transition
  human-led). Use for capture and sign-off gates.
- **`fanout`** — runs one subagent per element of a prior step's array (`over`), in parallel.
- **`gate-loop`** — runs, checks `gateField`; if non-empty, runs `fixWith` and re-checks, up to
  `maxRounds`. A bounded review→fix loop.

A step declares no model, effort, or tool list — those are repository-owned. `runProfile` names one of
the repository's profiles semantically (`implementation`, `ui-validation`, … — the machine resolves what
each one means); omit it for the repository default. Declaring `model`, `effort`, or `tools` is rejected.

**Placeholders** the prompts use (resolved at run time). Every one is a path the engine DERIVES, so a
step never authors a filename:

| Placeholder | Resolves to |
| --- | --- |
| `{{<artifactName>}}` | the item's artifact as an editable **working copy**, e.g. `{{problem}}` — the file a step reads is the file it writes, and whatever moved is published when the step returns |
| `{{<documentName>}}` | a workflow-scoped document (`{{lessons}}`), same treatment |
| `{{output}}` | this step's own output file — one per member in a fanout |
| `{{scratch}}` | this step's scratch directory, for whatever it merely uses; dropped once the step completes |
| `{{in:<stepKey>}}` / `{{inDir:<stepKey>}}` | a prior step's output file / a fanout's member directory |
| `{{questions}}` | the item's Q&A thread — read-only; a step adds to it by returning `needs-clarification` |
| `{{verifyFailure}}` | what the failed verify check said, for a heal step |
| `{{member.<field>}}` | a fanout member's field — the only placeholder yielding a value rather than a path |
| `{{itemDir}}` | the item's work dir. Prefer never to use it: everything a step legitimately reads or writes has its own derived reference above, and a path built by hand is one the engine does not know about |

A placeholder the engine cannot fill throws when the step is dispatched, so a name must match a declared
artifact or document exactly. `{{deliverable}}` is retired and rejected at save.

## What you do NOT declare

The **status table**, each status's **kind** (terminal / close-action target / rest stop), each
status's **`auto_transition_id`**, and each artifact's **owning transition** are all DERIVED from the
transitions you declare. Declare the edges and the namespaces; the store builds the rest.
