---
name: record-decision
description: Record a new or changed thesis design decision. Use when the user says a decision was made, changed or reversed (e.g. "we switch to Vive trackers because…").
---

# Record a decision

1. Find the matching note in `Decisions/`. If none fits, create `Dnn <Title>.md` (next free number) from `Templates/Decision.md`.
2. Update the note: Decision, Evidence (linked source or measurement; else write "unverified"), Options if they changed, Revisit if. Set `status` (accepted / superseded / reversed).
3. Append to History: `- YYYY-MM-DD: <what changed> (<why>)`. Never delete History lines.
4. Add a row to the decision log in `msc-thesis/docs/thesis-plan.md` §7: `| <date> | <one-line decision> | <one-line reason> | <revisit if> | [[<note name>]] |`. If that repo isn't available here, give the user the row to paste.
5. If the change contradicts `msc-thesis/docs/proposal.md` (approved by the supervisor), don't edit it; tell the user, since the supervisor may need to be informed.
6. Report: note changed, log row added, anything left open.
