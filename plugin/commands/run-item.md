---
description: Drive a workflow item through its autonomous transitions from THIS interactive session — the next_step → execute → submit_step loop — instead of the headless engine.
argument-hint: <item-id> [transition]
---

Run the `workflows.run-item` skill to drive item `$ARGUMENTS`.

The first argument is the item id (e.g. `B26.071`); an optional second argument names the exact transition to take at a forked status. If no item id was provided, ask the user which item to drive.
