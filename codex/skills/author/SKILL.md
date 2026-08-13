---
name: author
description: Use when creating or changing a workflow definition, including its transitions, steps, documents, conditions, outcomes, or input contract.
---

## Repository context

Before calling any other Workflows MCP tool, call `select_repository` once with the active repository's absolute root path. This transient argument compensates for Codex not advertising MCP roots; never write the path into repository or user configuration.

# Authoring a workflow

A workflow is the document-producing state machine an item flows through: a graph of **transitions**
(edges) between **statuses** (board columns), where each autonomous transition runs ordered **steps**
that produce **documents**. You build and amend it as DATA through the `workflows` MCP — never by
editing files. Your job is to keep a non-expert's process well-shaped without making them learn the
machine: propose the shape, explain it in plain terms, and write it on approval.

You hold the expertise the customer lacks. Lead with a recommended shape drawn from the idioms; don't
hand them a blank slate and ask them to design a state machine.

## The two jobs

- **Build** — create a workflow from the customer's intent. Pick the closest **shape idiom**, adapt it
  to their domain, and author it whole.
- **Amend** — change an existing workflow. Read it, apply the change, and write it back. The amend changes
  the live definition. An unfrozen head may retain its revision id; when the current head is pinned/frozen,
  copy-on-write forks a new revision and existing items keep their pins. Say this plainly when it matters
  (the customer often expects an edit to retroactively change running items — it does not).

## How to author (the loop)

1. **Ground.** Read what exists with `list_workflows` (names, descriptions, the input contract) and, to
   edit one, `read_workflow_definition` (the full typed `input` plus its opaque `definitionStamp`). For a
   common pattern, `install_workflow_template` adopts a bundled definition instead of authoring from
   scratch.
2. **Choose the shape.** Match the intent to one of the idioms in `reference/shape-idioms.md`
   (linear pipeline · capture→work→done · review-gate). Most workflows are one idiom, lightly adapted.
3. **Draft against best practice.** Build the typed `input` (the state machine) following
   `reference/best-practices.md` (how to shape transitions, step system-prompts, documents, router
   conditions, terminal outcomes) and the field-by-field contract in `reference/revision-input.md`.
   Always author the workflow's **input contract** (`itemEntryCriteria` + `entryDocumentGuidance`) too —
   it is what the add-item skill routes and records against, and a workflow without it gives that skill no bar.
   The input contract is plain workflow-registry metadata supplied with `create_workflow` for a new
   workflow or changed via `update_workflow` later, NOT part of the revision `input` (see
   `reference/best-practices.md`).
4. **Propose, then gate.** Authoring or changing a workflow is a high-impact, gated action — never write
   silently. Show the customer the shape (a short transition list / small diagram), call out the
   trade-offs and what each step costs, and **wait for approval**. Surface one real decision at a time.
5. **Write.** On approval, use the verb whose authority matches the job:
   - For a new workflow, call `create_workflow` once with its name, whole typed `definition`, metadata,
     input contract, and any `ensureDocuments`. The accepted call returns a usable workflow with a live
     head and fresh definition stamp. A `WorkflowCreationConflict` means another workflow already claimed
     the case-insensitive name or prefix; inspect `list_workflows` and choose another identity.
   - For an amend, call `set_workflow_definition` with the whole typed `input` and the unchanged
     `definitionStamp` returned by the read. A conflict means another author landed first, so preserve the
     draft, read fresh, and reapply rather than overwriting. Change input-contract metadata separately
     with `update_workflow`; that edit never forks a revision.
   - Use `replace_workflow_definition` only when the customer explicitly chooses destructive
     last-write-wins create-or-replace authority.

   The store validates the whole request and rejects an invalid one without partial state. Relay its
   domain message verbatim and fix the shape.
6. **Confirm.** Report what landed, and — when an amend forked a pinned/frozen head — remind that existing
   items keep their pinned revision.

## Judgment

Authoring is mostly judgment calls with no single right answer (how many transitions, where a human
gate belongs, whether a step earns its cost). Before you propose a shape, critique your own draft from
first principles — not "is it built well?" but "does it cohere and deserve to exist?":

- **Does every transition and step earn its cost?** Each one is latency and a place to get stuck. If a
  step produces no document a later transition or a human reads, cut it.
- **Can a non-expert steer this by reading the documents?** If the only way to supervise is to watch the
  agent work, the document shape is wrong.
- **Is this the smallest shape that does the job?** Bias hard to fewer transitions. Reach for an extra
  stage only when it lands a document someone genuinely consumes.
- **Tune a step's *strength* (model + effort), not only its existence.** Orthogonal to "does this step
  earn its place": once a step exists, its per-step `model`/`effort` set how much compute the leaf gets.
  Leave both **blank by default** — the step then inherits the run-wide model and the CLI's default
  effort, which is right for most steps. Override *upward* (a stronger model, higher effort) for a
  genuinely hard step — deep synthesis, a load-bearing review gate — where a weak pass quietly degrades
  every document downstream. Be wary of *downgrades*: a too-weak step produces plausible-but-worse output
  with **no error and no feedback loop** — the failure is invisible until a human reads the document —
  so only drop a step's strength when its work is genuinely mechanical (boilerplate, formatting).

When in doubt, ship the simpler shape and let the customer ask for more.

## Reference

This skill's `reference/` folder holds the detail, one file per concern (`revision-input.md`,
`best-practices.md`, `shape-idioms.md`). Each file's `covers:` frontmatter line says what it documents —
read the one whose `covers:` matches what you're doing rather than loading all three up front.
