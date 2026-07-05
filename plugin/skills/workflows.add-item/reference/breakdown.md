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

## Survey the rest of the tracker before proposing

You are curating one shared tracker, not dropping items into a void. Before proposing:

- Survey the items still **in flight** — not yet completed and not abandoned — and skim what they are
  (`list_items`, then `get_item` on the related ones).
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
`create_item`, or `set_parent` afterward) — distinct from a dependency: a parent *groups*, a dependency
*gates*.

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
