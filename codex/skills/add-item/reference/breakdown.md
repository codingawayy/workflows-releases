---
covers: decomposing work into clean items, surveying the rest of the tracker, ordering by dependency, and placing items under epics
---

# Breaking work down

## Decompose into clean items

- Each item is a **single coherent change with one verifiable outcome** ("done when …"), independently
  workable end-to-end without overwhelming whoever picks it up.
- **Splitting/merging test:** if an item needs "and" to describe it, consider splitting; if two pieces
  can't be verified apart, consider merging.
- Draft each item's entry document in plain language: the problem, what is **in** scope, what is
  explicitly **out**, and the done-when. State what *is* — not a plan narrative or a status. (The exact
  sections come from the target workflow's `entryDocumentGuidance` — see `routing.md`.)

## Read the workflow's execution budget when sizing

A workflow's steps each run at an assigned **model + reasoning effort** — the depth of thinking the
pipeline invests at that stage. Read the target workflow's definition (`read_workflow_definition`) to
see where that budget is spent, because it tells you which stages carry the design reasoning and which
are expected to execute from decisions already made:

- The **analysis and planning** transitions typically get the heaviest budget — they absorb ambiguity.
  An item can arrive under-specified and be shaped there.
- The **implement** transition runs from resolved documents — its job is to execute the plan, not to
  make design calls the earlier stages skipped.

Size and grain each item so the hard decisions land in the high-budget reasoning stages, not deferred
into execution. An item that would force the implement stage to invent design is mis-sized: split it so
the reasoning has a home upstream, or route it to a workflow whose budget matches the work.

## Survey the rest of the tracker before proposing

You are curating one shared tracker, not dropping items into a void. Before proposing:

- Call `list_items` with `scope: "all"` once. The compact survey is the complete retained history,
  including active, completed, and abandoned items. Use its `id`, `title`, `status`, `terminal`, and
  `parent_id` fields to identify plausible matches, then call `get_item` only for those candidates.
- Treat anything other than the complete inline survey as a hard stop. If the client truncates it,
  spills it to a file, returns a file reference, or otherwise withholds rows, tell the human that the
  overlap check could not complete and do not propose or create items from the partial result.
- Use the active candidates to decide dependencies; use the whole history to detect duplicates,
  supersession, and earlier abandoned attempts.
- Decide cross-relationships **in both directions**:
  - a new item that can't proceed until an in-flight one lands → the **new** item depends on the
    existing one;
  - an in-flight item that now logically waits on a new one → the **existing** item depends on the new
    one.
- Flag any in-flight item a new one **supersedes or overlaps** — surface it rather than silently
  duplicating. An idea previously abandoned is recreated fresh with the lineage noted, not un-abandoned.

## Record in dependency order

- Create each item only after the items it depends on exist, recording those dependencies in the same
  `create_item` call (`dependsOn`) — not as follow-up edits, so the item is correctly gated the instant
  it exists.
- Give each item its full entry document at creation, so it is complete the moment it exists.
- Add any remaining links afterward — **including updating existing items to depend on the new ones**.

## Place items under an epic

An **epic** is an item that *contains* a cluster of items serving one shared goal — the home for that
goal's shared context (its brief/scope). File items under it via the **parent** relation (`parentId` on
`create_item`, or `set_parent` afterward). A parent *groups* and a dependency says what must finish
first, but both **gate**: an item does not advance while anything under it is open, at any depth. So
filing an item under an epic holds that epic until the item closes, which is what makes the epic run
its members before itself.

- All the new items serve one goal → set the **same parent** on all of them. Reuse an existing epic if
  one already names the goal; otherwise propose creating one and writing its brief. An epic runs the
  project's planning/epic workflow — find it with `list_workflows`; if the project has none, adopting the
  bundled epics template (`install_workflow_template epics`) is the way to get one (a gated action, like
  any workflow creation — propose it, don't do it silently).
- They split across goals → **one parent per goal**.
- Don't manufacture epics: an item serving no shared goal stays a root (no parent) rather than be forced
  under a noise container.

(Labels are a separate axis — work-type and area tags, not epics. Assign them from the existing
vocabulary with `list_labels` + `assign_label`; never coin a new label while recording, and assign none
when none fits — a wrong label makes the board's filters lie.)
