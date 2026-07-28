---
name: workflows-run-item
description: Drive a workflow item through its autonomous transitions from THIS interactive session — the next_step → execute → submit_step loop — instead of the headless engine. Invoke when the user wants to run, drive, advance, or work a backlog item in-session (e.g. "/workflows-run-item B26.070", "drive B26.012 in-session", "advance this item here").
---

# Run an item in-session

Drive one workflow item forward from this interactive session, using the `next_step` / `submit_step`
MCP tools. The server owns sequencing and records state; **you are the executor** — you do the actual
thinking each step asks for. Every tool response carries a `guidance` field that describes exactly what
to do next; this skill is the wrapper so the user need not know the tool names. Follow the response, do
**only** the step you are handed, never skip ahead.

## 1. Identify the item (and an optional target transition)

Use the item id the user gave (e.g. `/workflows-run-item B26.070` → `B26.070`). If none was given, ask which item
— don't guess. Everything below uses that id as `item`.

A second argument is an optional **target transition** to drive (e.g. `/workflows-run-item B26.070 finalize-design`
→ `item = B26.070`, `transition = finalize-design`). This is how the board's deep link hands off a
specific stepped move at a forked status — the human clicked *that* move, so:

- Pass `transition` to **only the first** `next_step` call (see below); never re-pass it after a
  transition completes (`done`) — the item has advanced, and the next move is routed by status.
- A named-transition invocation is **attended by construction** — the human asked for this exact move.
  So when an **interactive** `run-step` comes back, collaborate (do **not** apply the "if unattended,
  stop" rule from §2 — the click is the human's presence).
- If `next_step` refuses the named move as no longer available (a stale link — the item already moved),
  retry **without** the transition argument to drive the item's current move, then continue normally.

## 2. Run the loop

Repeat: call `next_step({ item })` — on the **first** call only, include the target transition if one
was given (`next_step({ item, transition })`) — act on the returned `state`, and continue. Each
iteration handles exactly one response.

- **`run-step`** — a single leaf. Treat `instructions` as your system guidance, carry out `prompt`, and
  write your result to the `output` file. Then call `submit_step({ item, transition, step, result })` with
  `result` a JSON object conforming to `schema`. Handle the submit response (§3), then loop. (A
  document-writing leaf is a `run-step` too — its `guidance` says so, and its `output` already holds the
  target workflow document's current content; edit that in place into the full updated document.) An
  **interactive** step also comes back as a `run-step` — its `guidance` says to work it through TOGETHER
  WITH THE USER. If a user is present, surface the brief and collaborate, then submit their settled
  outcome; if you are driving unattended, **stop** and report that the item has reached a step that needs
  the user (don't fabricate their input).
- **`run-fanout`** — N members that must run **isolated**, one at a time. Take one member fully to completion — its `output` written — before starting the next; if a member's work genuinely cannot stay isolated when handled this way, stop and report to the user rather than risk cross-contamination. Each subagent uses the shared `instructions`, carries out its
  member's `prompt`, and writes a `schema`-conforming result to its member `output`. When **every**
  member output is written, call
  `submit_step({ item, transition, step, result: {} })`. Handle the response (§3), then loop.
- **`run-gate-step`** — one leaf of a review→fix→re-review gate-loop; the driver owns the loop. Carry out
  this single leaf (a review round, or the fix when `phase` is `"fix"`), write to `output`, and call
  `submit_step({ item, transition, step, result })` — submit to the gate-loop `step` you were given, not the
  fix step. The server hands you the next leaf. Handle the response (§3), then loop.
- **`done`** — a transition completed and the item advanced. **Keep going**: loop back to `next_step` to pick
  up the next autonomous transition (it will report `noop` when there is no more to drive).
- Any **stop** state (§4) — halt the loop and report (§5).

## 3. After a submit

- **`saved`** — the step recorded; continue the loop (`next_step`).
- **`done`** — the transition finished and the item advanced; continue the loop (a later transition may still be
  drivable).
- **`invalid`** — your result didn't satisfy the schema (leaf/gate) or some fan-out member outputs are
  missing. Fix exactly what `errors` lists — correct the result, or re-run the missing members — and call
  `submit_step` again. Nothing was recorded; do **not** advance past it.
- **`paused`** — a leaf returned clarification questions; the item is blocked at its current transition. **Stop
  the loop.** Surface the questions to the user; once they answer (in conversation, then recorded on the
  item's Q&A thread), they can re-invoke `/workflows-run-item` to resume.
- **`conflict`** — a document-writing step's target workflow document changed underneath you between read
  and write; nothing was recorded. The server refreshed the **document file** in the work dir to the
  current content (this is the `{{<name>}}` input your prompt references — NOT the `output` file, which
  holds your now-stale draft). Re-read that refreshed document file, re-apply your changes on top, write
  the full updated document to `output` again, and call **`submit_step`** again — **do not** call
  `next_step`, and **do not** overwrite the other run's update. Stay in this re-submit loop until it lands.
- **`artifact-conflict`** — one of the ITEM's own documents (an artifact this transition publishes) changed
  after this session read it, so the finish stored **nothing at all**: no document, no status advance. The
  server refreshed that artifact's file in the work dir to its current content, and the response carries
  that content too. Re-read it, merge what this transition produced into it, write the merged result back
  to the same **publish** file the step wrote, and call **`submit_step`** again — the claim is still held,
  so a re-submit picks up where you were. **Never `cancel_execution` here**: that discards the work dir,
  and with it everything this run composed.
- **`gate-exhausted`** — the gate-loop hit its `maxRounds` cap with findings still open; nothing advanced.
  **Stop the loop** and report the open findings; the user resolves them and resets the transition, or runs it
  headless.
- **`claim-lost`** — another run took the item over after this session's claim lapsed; nothing was
  recorded. **Stop the loop** and tell the user (the auto-run engine likely picked it up).
- Any other state (e.g. `error`, `unsupported`) — **stop the loop** and report the message verbatim.

## 4. When to stop

Halt the loop — do not keep calling `next_step` — on any of these. Report which one and why (§5).

| State          | Meaning                                                                          |
| -------------- | -------------------------------------------------------------------------------- |
| `noop`         | No autonomous transition consumes the item's status — nothing to drive.               |
| `blocked`      | Unmet dependencies (listed in `unmet`) — those items must finish first.          |
| `busy`         | Another run (likely the auto-run engine) holds the run claim — wait or stop it.  |
| `unsupported`  | This step kind isn't drivable in-session — run the transition headless instead (the item's move on the board, or auto-run). |
| `error`        | A precondition failed (e.g. unsupported definition format) — report the message. |
| `paused` / `gate-exhausted` / `claim-lost` | Pause/cap/handover from a submit (§3).                          |

## 5. Report

When the loop stops, tell the user plainly: the item's final status, the transitions you advanced it through
this run, and the stop reason with the one concrete next action it implies (answer questions, resolve a
dependency, collaborate with the user on a step that needs them, etc.).
