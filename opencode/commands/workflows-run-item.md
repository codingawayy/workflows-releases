---
description: Drive a workflow item through its autonomous transitions from THIS interactive session — the next_step → execute → submit_step loop — instead of the headless engine. Invoke when the user wants to run, drive, advance, or work a backlog item in-session (e.g. "/workflows-run-item B26.070", "drive B26.012 in-session", "advance this item here").
---

Load the `workflows-run-item` skill and follow it completely.

Drive item `$ARGUMENTS` using the instructions below.

The first argument is the item id (e.g. `B26.071`); an optional second argument names the exact transition to take at a forked status. If no item id was provided, ask the user which item to drive.
