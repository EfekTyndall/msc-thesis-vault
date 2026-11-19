---
name: record-decision
description: Record a new or changed thesis design decision. Use when the user says a decision was made, changed or reversed (e.g. "we switch to Vive trackers because…").
---

# Record a decision

1. Find the matching note in `Decisions/`. If none fits, create `Dnn <Title>.md` (next free number) from `Templates/Decision.md`.
2. Update the note: Decision, Evidence (linked source or measurement; else write "unverified"), Options if they changed, Revisit if. Set `status` (accepted / superseded / reversed).
3. Append to History: `- YYYY-MM-DD: <what changed> (<why>)`. Never delete History lines.
4. Add a row to the decision log in `msc-thesis/docs/thesis-plan.md` §7: `| <date> | <one-line decision> | <one-line reason> | <revisit if> | [[<note name>]] |`, on a branch with an msc-thesis PR. If that repo isn't available here, give the user the row to paste.
5. Search the files that repeat decision values for anything the change makes stale: `msc-thesis/docs/hardware-specs.md`, `dex1-1-handheld/hardware/PRINT-TEST-QUEUE.md`, `dex1-1-handheld/hardware/SLICER-SETTINGS.md`, the affected part READMEs in `dex1-1-handheld/hardware/`, and `Waiting.md`. Fix them in the same change (PR for the repos, direct for the vault) or list them in the report. In `dex1-1-handheld` write no vault IDs or vault paths.
6. If the change contradicts `msc-thesis/docs/proposal.md` (approved by the supervisor), don't edit it; tell the user, since the supervisor may need to be informed.
7. Report: note changed, log row added, other files fixed or listed, anything left open.
