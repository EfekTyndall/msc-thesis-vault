---
date: 2026-11-24
topic: AI use register (living note, source for the thesis "Register of AI tools used")
status: living, updated by every thread whose output may enter the thesis
---
# AI use register

The h_da EIT flyer "AI Use for Students" (Version 04/2026, Rev. 1, uploaded by Martyn 2026-11-24) requires, for a final thesis, that every disclosed AI use is (1) cited with the tool and its primary prompts, IEEE style, and (2) listed in a table "Register of AI tools used". This note collects both while the work happens, so the thesis table can be copied out in April instead of rebuilt from memory. ABPO §22(9) (signed declaration of independent work, all aids named) is the legal basis.

## What the flyer says for a final thesis

| Category | Uses |
| --- | --- |
| No disclosure needed | brainstorming a topic and gathering ideas; introduction to research and literature searches; arranging references |
| Allowed if disclosed | AI-generated text (with citation rules); suggesting a structure; images, graphics, diagrams; translation; **coding: only in agreement with the examiner**; data analysis; grammar, style, language; **presentations: only in agreement with the examiner** |
| Always | check every AI output for accuracy; read every cited source in the original (hallucinated sources); state functional uses of AI tools |
| Company | the supervising company's AI guideline must also be followed (footnote 1); Daimler guideline: not yet checked |

## Open actions

- [ ] Agreement with the examiner on AI-written source code (dex1-1-handheld, analysis scripts) and on AI-assisted presentations; ideally in writing.
- [ ] Check whether Daimler has an AI-use guideline that applies to the thesis.
- The two items above are tracked in [[Waiting]], Writing (2026-11-24 day close).
- [ ] Before any history rewrite of dex1-1-handheld at handover: refresh the commit list below (see Handover.md).

## Register (thesis table draft)

Columns as in the flyer's example 3. "Sections" is filled once the thesis chapters exist. "Own input" states what Martyn did: tested, measured, decided, revised.

| GenAI use | Sections | Own input | AI tool | Remark |
| --- | --- | --- | --- | --- |
| Source code: replay pipeline (MuJoCo, URSim, real UR10e), pose and trigger logging (dex1-1-handheld `software/`) | methods, system chapter (tbd) | Requirements and safety limits set by Martyn; every script run and checked on the lab hardware (Replay Ladder RL1 (replay in MuJoCo on the Mac), RL2 (URSim on fractal-pc) and RL3 (real UR10e in air) passed 2026-11-23) | Claude (Anthropic), Claude Code | Needs examiner agreement (coding). Commit evidence: `2026-11-24 AI-assisted commits.csv` |
| Planning and design documents (msc-thesis `docs/`: proposal, thesis plan, research notes, hardware specs) | tbd (only if text is reused) | Decisions taken by Martyn; text revised by Martyn | Claude (Anthropic) | Text taken into the thesis must be cited or rewritten |
| Literature searches and research summaries (vault `Agent/Research/`) | none directly | Sources read in the original via Zotero before citing | Claude (Anthropic) | Search support needs no disclosure; summaries reused as thesis text do |
| CAD edits in Fusion (handheld sleeve, adapter) via the Fusion connector | hardware chapter (tbd) | Design intent, dimensions and every print test by Martyn | Claude (Anthropic) | "Other outputs" in the flyer's sense; disclose if CAD is shown |
| Explainer pages, diagrams, mind maps (e.g. D01 (handheld gripper body) mind map, data-format explainer) | tbd (only if a figure is reused) | Content checked by Martyn | Claude (Anthropic) | Disclose any figure that enters the thesis |
| RL4 (Robotiq pick-and-place) procedure and pose analysis: start pose HOME_FACE by inverse kinematics on the team UR10e model, PICO controller model lined up with the scan in Fusion, fingertip offsets (vault `Agent/Reports/2026-11-24 RL4 Robotiq pick-and-place procedure.md`) | methods, experiments (tbd) | Start-pose direction, recording with the dummy, and the camera-adapter correction by Martyn; jaw direction and 170 mm measured by Martyn at the robot; HOME_FACE not yet checked on the robot | Claude (Anthropic), Claude Code, Fusion connector | Simulation and CAD values unverified until measured at the robot |
| LaTeX skeleton (chapter files, layout) in msc-thesis `thesis/` | whole document structure | h_da template chosen and set up by Martyn | Claude (Anthropic) | Structure suggestion, disclose |
| Camera View: Dex1-1 camera view rendered in MuJoCo from Unitree's URDF and meshes (scene and render script, picture in vault `Agent/Research/figures/2026-11-25 Dex1-1 camera view sim.png`) | hardware or methods chapter (tbd) | Idea to check the camera field of view in simulation by Martyn; values from Unitree sources; not yet checked against the real camera | Claude (Anthropic), Claude Code | Figure: disclose if reused; simulation only (pinhole, real lens wider at the corners) |

## Primary prompts

Numbered as in the flyer's example 2. Add a prompt when its output enters the thesis (text, figure, code, analysis). Quote Martyn's request, shortened, with the date.

| No. | Primary prompt | Date |
| --- | --- | --- |
| P1 | "At the end I want to have a full modular framework that can be easily adjusted and adapted into different robots … three main modes: Teleoperation, Recording, and Replay mode … Does this actually feasible?" (output: vault `Agent/Research/2026-11-24 SwapKit modular framework options.md`) | 2026-11-24 |
| P2 | "Plan RL4 Robotiq pick-and-place" (Replay Ladder thread; Martyn's corrections: robot works in front of itself, gripper facing the operator; record with the full handheld and the dummy; camera adapter between flange and Robotiq) (output: vault `Agent/Reports/2026-11-24 RL4 Robotiq pick-and-place procedure.md`) | 2026-11-24 |
| P3 | "For the Dex One camera, is it possible that we check the field of view in simulation using Unitree Mujoco library … check if they have the Dex one gripper camera specification in the repo" (output: vault `Agent/Research/2026-11-25 Camera View Dex1-1 and D435i field of view.md`) | 2026-11-25 |
| P4 | "Current works: some of them already designed the UMI handheld with modularity, right? … my DexOne gripper comes late and we only have another gripper, modularity is key … I'm getting doubts now" (output: vault `Agent/Research/2026-11-25 Handheld Rethink real gripper vs passive design.md`) | 2026-11-25 |
| P5 | "Yes let's do that" (adaptation plan for HandUMI with Dex1-1 tips as a parallel fallback, after Martyn's own reasoning on simplicity vs modularity) (output: vault `Agent/Reports/2026-11-25 HandUMI fallback adaptation plan.md`) | 2026-11-25 |

## Commit evidence

`2026-11-24 AI-assisted commits.csv` (same folder) lists every non-merge commit on `main` of the three repositories, with date, author, subject and whether it was AI-assisted (author Claude, a `Co-Authored-By: Claude` line or a Claude session link). Count of 2026-11-24:

| Repository | Commits (no merges) | AI-assisted |
| --- | --- | --- |
| msc-thesis | 115 | 108 |
| msc-thesis-vault | 430 | 324 |
| dex1-1-handheld | 81 | 69 |

Note: "AI-assisted" means Claude wrote or co-wrote the commit; Martyn's review, tests and decisions are not visible in this count.

## How this note is kept up to date

- Every thread that produces something that may enter the thesis (text, figure, code, analysis, CAD) adds or updates a register row and, if useful, a prompt.
- The commit list is regenerated at the end of each month and before the handover history rewrite.
