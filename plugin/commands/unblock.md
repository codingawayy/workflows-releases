---
description: Inspect the configured project's human-intervention queue, handle selected items in order, or guide one waiting item at a time without taking a run claim.
argument-hint: [item-id ...]
---

Run the `workflows.unblock` skill for `$ARGUMENTS`.

Each argument is an optional item id (e.g. `B26.070`). Preserve every supplied identifier and its order. With no identifiers, start the guided project pass.
