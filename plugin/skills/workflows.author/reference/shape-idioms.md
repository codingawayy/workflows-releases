---
covers: the small vocabulary of good workflow shapes to start a build from
---

# Shape idioms

A workflow is almost always one of a few shapes, adapted to a domain. Start from the closest idiom and
change as little as possible — don't invent a novel state machine when one of these fits. This is a
vocabulary of good shapes, not a template library: adapt the transitions, statuses, and documents to the
customer's work.

## 1. Capture → work → done (the minimum)

The smallest useful workflow. One interactive capture, one autonomous worker, a terminal.

```
add (interactive, → captured)  ──auto──▶  do (leaf, captured → done)
```

- `add` interviews the human and writes the entry document; `do` does the work and produces the result.
- Use when the work is a single autonomous step with no review or human gate.
- Add a drop edge `captured → dropped` so a bad item can be abandoned.

## 2. Linear pipeline (capture → analyze → design → plan → implement)

The backlog idiom: a chain of autonomous transitions, each producing a document the next consumes, with
**one human gate** in the middle where a go/no-go decision lives.

```
add-item (interactive, → proposed)
  ──auto──▶ analyze-item (leaf, proposed → analyzed)        its step edits the analysis document
            finalize-design (interactive, analyzed → designed)   ← the human gate (go/no-go/drop/send-back)
  ──auto──▶ plan-implementation (leaf, designed → planned)  its step edits the plan document
  ──auto──▶ implement (needsBuildEnv, planned → implemented) its step edits the summary document
```

- Each autonomous stage is `auto` so the pipeline flows; the one interactive stage is where a human
  steers. The human reads the `analysis` and decides — they never watch the agent work.
- The implement stage is `needsBuildEnv: true` because its steps run the repository's code — every
  transition gets its own checkout, and this one's is given a dependency install and the repo's `setup`.
- Add rework as an ordinary edge back (`analyzed → proposed`, `removes: ["analysis"]`) and drop edges
  off each pre-terminal status.
- Use for any "an idea becomes a shipped change through reviewed stages" process.

## 3. Review-gate (work → review → fix loop)

A worker followed by a bounded review that loops back until clean — built with a `gate-loop` step inside
one transition, not separate transitions.

```
implement transition:
  steps:
    - 01-implement   (leaf)
    - 02-review      (gate-loop, fixWith: 01-implement, gateField: mustFix, maxRounds: 3)
    - 03-summary     (leaf)
```

- `02-review` checks the work; while `mustFix` is non-empty it re-runs `01-implement`, up to `maxRounds`.
- Use when output needs an automated quality gate before it's accepted (a UI review, a lint/verify pass,
  a correctness check).
- Combine with a `fanout` step when the review has independent lenses (run one subagent per lens, then a
  `leaf` synthesis) — the analyze stage of the backlog idiom does exactly this.

## Composing them

Real workflows compose these: a linear pipeline whose implement stage carries a review-gate, with a
capture front and drop edges throughout. Start from the idiom closest to the customer's described
process, then add only the transitions and steps that each land a document someone reads.
