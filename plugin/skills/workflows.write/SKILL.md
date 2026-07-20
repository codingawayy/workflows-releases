---
name: workflows.write
description: The system's writing-craft guidance for any document, report, or written deliverable — lead with the conclusion, orient before detail, visualize, write for the cold-start reader who has only the documents, and be concise. Invoke when composing or tightening written content, especially outside a workflow transition (a transition already injects this same guidance into every step).
user-invocable: false
---

# Writing craft — for every document, deliverable, and intermediate file you produce

This is standing guidance on *how to write* whatever this step produces — an artifact, a workflow
document, or an intermediate `_work/` file a later step reads cold. Apply it wherever it aids the
reader; where a step has nothing written to apply it to (it emits code or pure JSON), it simply
doesn't apply.

The reader you are writing for has **only the documents** — no memory of this conversation, no
context beyond what earlier documents established. Write so that reader can act.

1. **Lead with the conclusion.** The first paragraph carries the verdict, decision, or shape; the
   supporting reasoning follows. Don't make the reader earn the point.
2. **Orient before detail.** In a longer document, give the map before the specifics ("there are
   two kinds of X…") so each detail has somewhere to land.
3. **Visualize.** Board-rendered documents display Mermaid diagrams (```mermaid fences),
   syntax-highlighted code, and inline HTML — reach for one where structure, flow, or a comparison is
   the point and a picture beats prose, and only there; a folder tree suits files or layout. A diagram
   that merely restates a sentence is noise; keep each tight and let it replace the prose it duplicates.
   In a flowchart or mindmap, wrap any node/edge label containing punctuation like parentheses or
   brackets in double quotes (`A["app.web.api (gateway)"]`, not `A[app.web.api (gateway)]`) —
   otherwise the parser reads it as node syntax and the board silently drops the diagram to raw
   source.
4. **Show code where it helps.** A short snippet often lands faster than a paragraph describing it.
5. **Concrete over abstract.** A specific example or number lands faster than a general statement.
6. **Don't repeat** what the item's earlier documents already established — build on them, cite
   them, don't restate them.
7. **Link, don't name-drop.** Refer to another item as a link in the project-scoped URL form
   (`[B123](/my-project/item/B123)`), and link related material, including external sources.
8. **Qualify code references.** Prefix a code symbol with the parent it belongs to —
   `WorkflowModel.entryDocument()`, not "the entryDocument method".
9. **Write for the cold-start reader.** Use the vocabulary earlier documents established; define any
   term this document itself introduces. Self-check: reread as a first-timer who is mid-task and
   will stop reading the moment they can act.
10. **State what is, not the journey.** The document is the current truth — no meeting minutes, no
    "after discussion we decided". History lives in the Q&A thread, not the deliverable.
11. **Be concise.** Cut anything that doesn't change what the reader will do or decide. Concision by
    selection, not by compression — remove whole points that don't earn their place, don't cramp the
    ones that do.
