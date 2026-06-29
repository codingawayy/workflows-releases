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
  "documents":   [ "lessons" ],                          // cross-item workflow docs (shared, versioned)
  "offChainStatuses": [],                                // statuses NO transition touches (usually empty)
  "statusDisplay": [ { "name": "proposed", "label": "Proposed", "color": "#888" } ], // optional board metadata
  "itemEntryCriteria": "what type+size of item belongs here",          // the input contract (always set it)
  "entryDocumentGuidance": "what a good entry document for this workflow contains"
}
```

## A transition (an edge in the graph)

```jsonc
{
  "name": "analyze-item",          // unique, kebab-case; this is the artifact-keyed name
  "ordinal": 2,                    // authoring/pipeline order (a board-order hint)
  "statusIn": "proposed",          // the status it leaves FROM; omit ONLY on the single entry transition
  "statusOut": "analyzed",         // the status it lands ON
  "execution": "shared",           // "shared" (default) or "isolated" (runs in its own git worktree)
  "description": "what this edge is for — short prose",
  "auto": true,                    // engine auto-advances through it when an item rests at statusIn
  "routerCondition": "...",        // prose the router reads to pick this edge among a status's exits
  "invalidates": ["analysis"],     // a BACKWARD edge: documents it clears on the way back
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
- **A backward edge** carries `invalidates` (the documents it clears so their producers re-run) and must
  be `execution: "shared"`, never isolated.

## A step (one unit of an autonomous transition)

```jsonc
{
  "key": "01-analyze",      // natural key within the transition; over/runWhen/fixWith reference it
  "kind": "leaf",           // leaf | fanout | gate-loop | interactive
  "systemPrompt": "...",    // the step's instructions (its role + how to do the work)
  "userPrompt": "...",      // the task: what to read, what to write, the {{placeholders}} it uses
  "schemaJson": "{...}",    // JSON-schema string for the step's structured return
  "writesDocument": "lessons", // (leaf/interactive only) the cross-item document this step rewrites
  "tools": ["Read", "Grep"],   // the allowlist of tools the leaf may use
  // kind-specific:
  "over": "01-analyze.lenses",        // (fanout) the prior step's array field to fan out over
  "runWhen": "01-analyze.reviewApplies", // run this step only when a prior boolean field is truthy
  "fixWith": "01-implement",          // (gate-loop) the sibling step re-run to fix findings
  "gateField": "mustFix",             // (gate-loop) the array field whose emptiness ends the loop
  "maxRounds": 3                      // (gate-loop) the loop cap
}
```

**Step kinds:**

- **`leaf`** — one `claude -p` subprocess does the work and returns structured JSON. The workhorse.
- **`interactive`** — a human and the agent work it through in conversation (makes the transition
  human-led). Use for capture and sign-off gates.
- **`fanout`** — runs one subagent per element of a prior step's array (`over`), in parallel.
- **`gate-loop`** — runs, checks `gateField`; if non-empty, runs `fixWith` and re-checks, up to
  `maxRounds`. A bounded review→fix loop.

**Placeholders** the prompts use (resolved at run time): `{{itemDir}}`, `{{deliverable}}` (the path the
transition's produced artifact is ingested from), `{{output}}` (an intermediate path), `{{questions}}`,
`{{<artifactName>}}` (a prior document, e.g. `{{problem}}`), `{{in:<stepKey>}}` /
`{{inDir:<stepKey>}}` (a prior step's output), `{{member.<field>}}` (a fanout member's fields).

## What you do NOT declare

The **status table**, each status's **kind** (terminal / close-action target / rest stop), each
status's **`auto_transition_id`**, and each artifact's **owning transition** are all DERIVED from the
transitions you declare. Declare the edges and the namespaces; the store builds the rest.
