---
date: 2026-11-19
tags: [report, audit]
---
# Three-repo consistency audit (2026-11-19)

Read-only audit of `msc-thesis`, `msc-thesis-vault` and `dex1-1-handheld`. It covers the local clones on the Mac and the GitHub remotes. Nothing was changed, deleted or committed except this file. Each finding gives its location and a suggested action (**update** / **remove** / **keep**). Martyn decides.

**Severity:** **High** means it contradicts a current decision, could cause a wrong print, or is a handover risk for the company repo. **Medium** means a stale document that misleads a reader. **Low** means housekeeping or cosmetics.

**Taken as current, so not flagged:** the sleeve is frozen as v22 + tape (D10, 2026-11-19). The next print is v23 with the plate-1 settings (flow 0.95) once the inserts arrive. PRs #10/#11 (handheld) and #37 (msc-thesis) are merged. Fusion controller-grip v24 still has `back_inner_offset` = 0.3 mm active, and that caveat is already in [[Waiting]].

## How it was checked

- **Git state:** `git fetch --prune`, `git status --ignored`, `git branch -a -vv`, `git worktree list`. Each branch was checked with `git rev-list origin/main..<branch>`, and `gh pr list` was run per repo.
- **dex1-1-handheld:** `git ls-files` plus a grep for vault IDs, vault paths, "Claude", "thesis" and "vault". `shasum -a 256 -c SHA256SUMS` was run in every hardware folder, and all files listed there pass.
- **Duplicate files:** `cmp` of every file in `msc-thesis/hardware/` against `dex1-1-handheld/hardware/`.
- **Text comparison:** the same facts were compared across the D01–D10 notes, `docs/thesis-plan.md` §7, `docs/hardware-specs.md`, `PRINT-TEST-QUEUE.md`, the part READMEs, `SLICER-SETTINGS.md`, [[Waiting]], [[2026-11-18]] and [[2026-11-19]].
- **Instructions copy:** `docs/setup/claude-project/project-instructions.md` was compared with the current project instructions. `diff` was run on the local Claude setup copies.

## Summary

| Repo | High | Medium | Low |
| --- | --- | --- | --- |
| dex1-1-handheld | 3 | 6 | 5 |
| msc-thesis | 0 | 5 | 6 |
| msc-thesis-vault | 0 | 5 | 6 |
| Cross-repo (git/GitHub) | 0 | 2 | 3 |

**Checked and consistent (keep):**
- `project-instructions.md` matches the current project instructions word for word.
- The local `CLAUDE.local.md` and `.claude/settings.local.json` in `~/dex1-1-handheld` equal their master copies in `msc-thesis/docs/setup/claude-project/dex1-1-handheld-local/`. Both are hidden from Git by `.git/info/exclude`, and neither is tracked.
- No open PR in any repo. All 11 handheld PRs and all 37 msc-thesis PRs are merged. Vault PRs #23 and #44 are closed unmerged.
- Local `main` equals `origin/main` in all three repos, with no stash.
- Decision log §7 has a row for every D-note history entry since 2026-11-16 (D01, D04, D06, D07, D08, D10 × 3), and these rows agree with the notes.
- `msc-thesis/CLAUDE.md` says "Training export: absolute end-effector pose". This agrees with D07 (2026-11-18): the export stores absolute poses, and the relative conversion happens at training.

## dex1-1-handheld

### High

1. **v23 print settings contradict each other in `hardware/PRINT-TEST-QUEUE.md`.**
   - Line 37 (under v22): "Next print: flow ratio 1.00 … 'Spectrum PETG @BBL H2S' … no brim". This was superseded on 2026-11-19.
   - Line 46 (v23): "the plate 1 settings that snapped, i.e. Generic PETG @BBL H2S, flow 0.95 … no brim".
   - Line 32 records that plate 1 had an **8 mm outer brim**, and [[D10 Handheld mechanism]] (Evidence, "Sleeve slip and v24") names flow **and brim** as the two differences between plate 1 and plate 3. So "plate 1 settings … no brim" is not the plate-1 setup. D10 History and [[Waiting]] say only "plate 1 settings (Generic PETG, flow 0.95)" and do not mention the brim.
   - **Update:** mark line 37 as superseded. Martyn should state explicitly whether v23 prints with or without the brim, and use the same wording in all three places.
2. **Vault and thesis references in a repo that will be shared with the company.** All of these need cleaning at handover. **Update/remove** at handover; until then, **keep** but add no new ones.
   - `hardware/controller-grip/README.md:3` "(vault: D10)".
   - `hardware/dex1-1-reference/README.md:3` "(vault: D01, D02, D10)", and `:7` "(daily note 2026-11-03)", `:8` "measurements in vault D10".
   - `hardware/dex1-1-reference/from-unitree-ros/README.md:3` "(vault: D01, `Agent/Reports/2026-10-28 Handheld design start plan.md`, step 2)".
   - `hardware/pico-4-ultra-glock-shell/README.md:3` "(vault: D10)".
   - `hardware/pico-4-ultra-scan/README.md:3` "(vault: D04, D10)".
   - `software/xr/trigger_udp.py:8` "(vault: D04, research gaps B7)".
   - `hardware/PRINT-TEST-QUEUE.md`:
     - `:19` "vault `Agent/Research/2026-11-09 Grip sleeve snap-fit closure.md`"
     - `:57` "`docs/thesis-plan.md` §7"
     - `:75` "full note in the design notes"
     - `:96` "(vault PR #31)"
     - `:105` "(thesis plan, handheld design start plan step 3)"
     - `:3` "`experiments/<YYYY-MM-DD>-<name>/`", which is an msc-thesis folder.
3. **Claude attribution in the history.**
   - 34 of 88 commits have the author `Claude <noreply@anthropic.com>`, and 55 commit messages carry `Co-Authored-By: Claude`.
   - All 11 PRs came from `claude/project-thread-*` or similar branches, and PR titles and bodies stay on GitHub.
   - `msc-thesis/CLAUDE.md` says commit lines are removed at handover. Author rewriting and the GitHub PR history are not covered by that rule.
   - **Update** at handover: decide between a history rewrite and a fresh export repo. A fresh export also drops the PR history.

### Medium

4. **CAD-in-Git rule conflicts with tracked files.**
   - `hardware/real-adapter/files/real-adapter-v9.f3d` and `.step` are tracked (commit b8ae783, 2026-11-16).
   - `.gitignore` says "Own designs: source CAD (.step, .f3d) is tracked", and `real-adapter/README.md` says "Only the source CAD is tracked".
   - The standing rule from 2026-11-16 (project memory "No hardware files in Git") is to never commit `.f3d`/`.step`/`.stl`/`.3mf`.
   - Meanwhile `hardware/real-adapter/files/` is gitignored entirely, so v10–v14 are not tracked. Git therefore holds an outdated v9 master.
   - **Update:** pick one rule. If CAD stays out of Git, untrack v9 and fix the `.gitignore` comment and the README. If source CAD goes in, track v14.
5. **`hardware/real-adapter/README.md` says "Current version: v9".**
   - The current version is v14 (two-piece clamp, PRINT-TEST-QUEUE §3).
   - The file table and `SHA256SUMS` list only v9. The local `files/` also holds v10, v13 and v14 exports, which are in neither.
   - **Update:** set the current version to v14 and add the v14 files and their sums.
6. **`hardware/controller-grip/README.md` lists only the v21 and v19 exports.**
   - Its `SHA256SUMS` lists only the 4 v21/v19 files.
   - The local `files/` holds `sleeve-A-{front,back}-v22.stl`, `sleeve-A-front-v23.stl` (the v23 print file), `sleeve-A-back-v24-offset{0.2,0.3}.stl`, `insert-coupon-4.0-4.1-4.2.stl` and a `_analysis/` folder. None of these are documented or summed.
   - "0.3 mm clearance" is the design value. PRINT-TEST-QUEUE line 39 measures v23 at about 0.4 / 0.9 / 0.3 mm by band.
   - **Update:** add v22/v23 with their sums, mark v24 as dropped, and give the clearance by band.
7. **`hardware/PRINT-TEST-QUEUE.md` §3 has stale open items.** **Update:** tick or annotate each item.
   - Line 65: the dummy rear-section item still says "Bolt `real-adapter` (v9)".
   - Line 85: "Not sliced yet", superseded by line 86.
   - Line 88: "[ ] Print the clamp base standing on its bottom edge…" is still open, but it was printed on 2026-11-18 (line 87).
   - Line 95: "check … that the M3 clamp screw closes the split" refers to the one-piece ring. v14 uses 2 × M3 × 16 and has no split.
   - Line 96 mixes the 2026-11-06 values (Ø4.0, 2.0 mm wall) with v23 (Ø4.1, 1.95 mm).
8. **`hardware/SLICER-SETTINGS.md` "Changes we use" is incomplete.** It omits settings the print notes rely on:
   - flow ratio 0.95 (Generic PETG) vs 1.00 (user preset "Spectrum PETG @BBL H2S");
   - 255 °C nozzle and 70 °C bed;
   - brim (8 mm outer on plate 1, none afterwards);
   - PETG tree supports, build plate only, and "Don't support bridges" (plate 2c);
   - 40 % / 15 % / 100 % infill (plate 2c).

   **Update.**
9. **`hardware/real-dummy-gripper/README.md:21` says "M3 × 25 cap screws".** Since real-adapter v10 the rear mount uses M3 × 35 (PRINT-TEST-QUEUE line 68). **Update.**

### Low

10. **"Bambu Lab H2" vs H2S.** `PRINT-TEST-QUEUE.md:19` says "lab Bambu Lab H2". The printer is the H2S everywhere else, and `msc-thesis/docs/hardware-specs.md` conflict 6 already lists this. **Update.**
11. **`PRINT-TEST-QUEUE.md:3` describes the wrong file location.** It says "STL exports for printing go next to this file", but the exports live in `hardware/<part>/files/` and are not in Git. **Update.**
12. **`PRINT-TEST-QUEUE.md` §1 headings are out of date.**
    - Line 16 says "pick one for v1".
    - Line 18 is the variant A item and is still `[ ]`.
    - The sleeve is now frozen (v22 + tape).
    - **Update** the wording if wanted.
13. **Internal IP in the usage example of `software/xr/trigger_udp.py`.** It shows the Mac's address `10.36.1.125`. **Keep** for now; replace it with a placeholder at handover.
14. **Local-only Claude files in `~/dex1-1-handheld`.** `CLAUDE.local.md` and `.claude/` are excluded by `.git/info/exclude` and are not in Git. **Keep**, but leave them out if the folder is ever zipped or copied for the company. `.git/info/exclude` lists both entries twice, since the install ran twice; **remove** the duplicate lines (cosmetic).

## msc-thesis

### Medium

15. **`docs/hardware-specs.md` is stale since 2026-11-16 ("Last compiled: 2026-11-16").** **Update** each row:
    - Line 52: the sleeve row is "(v21) 0.3 mm clearance; wall 3.5 mm". It is now v22 + tape in use and v23 next.
    - Line 53: "Real adapter (v9) … not in Git, no README". It is now v14, a README exists, and the v9 `.f3d`/`.step` are tracked.
    - Line 63: "Adapter boss holes Ø4.0 × 7.0 mm … not yet applied in Fusion". The holes are Ø4.1 × 7.0 mm with a 1.95 mm wall in controller-grip v23, and they were applied in Fusion (D10, 2026-11-16 and 2026-11-18).
    - Line 64: "Adapter screws (v7) rear 4 × M3 × 25, clamp 1 × M3 × 16 + nut". Now: rear 4 × M3 × 35 (v10+), clamp 2 × M3 × 16 + 2 nuts (v14), pad 2 × M3 × 10.
    - The PETG row says "Spectrum assumed … brand U". Martyn confirmed Spectrum on 2026-11-16 (`dex1-1-handheld/hardware/SLICER-SETTINGS.md`). Set it to M.
16. **`docs/thesis-plan.md` §3–4 still plan a "replica vs real gripper" image test.** The core design now uses the real Dex1-1 (D01, 2026-11-16), so this test only applies to the fallback.
    - §3 Metrics, "Device" row: "replica-vs-real image match".
    - §4 Error budget, third row: "Replica vs real gripper in the camera image".
    - [[D09 Calibration]] has the matching row "Visual match of printed copy vs real gripper".
    - **Update**, but changing the thesis plan needs Martyn's go.
17. **`docs/thesis-plan.md` header and §5–6, §8 still describe the replica as the main path.** **Update** (needs Martyn's go).
    - The header says "last updated Oct 23, 2026".
    - §5 November work: "Replica v1 …"; gate "Replica records camera, pose and width together".
    - §6 risk "Replica mechanism wobbles … swap the real gripper into the handheld". That swap is now the chosen path.
    - §8 open item "Collect replica mechanism designs from UMI and later variants". The copy path has been frozen since 2026-11-05.
18. **Leftover `hardware/` folder (105 MB, gitignored).** Every file in it is a byte-identical copy of the same path in `dex1-1-handheld/hardware/` (`cmp`). Its `.gitignore` entries (the `hardware/...` lines) date from before the move. **Remove** the local folder and the stale `.gitignore` lines.
19. **Decision log rows are out of date order.** §7 is mostly chronological, but these rows break the order:
    - "17 Oct" after "23 Oct";
    - "4 Nov" before "3 Nov";
    - "10 Nov" before "6 Nov";
    - "18 Nov" before "16 Nov";
    - "18 Nov" before "17 Nov".

    **Update** (sort), or **keep** if append order is intended.

### Low

20. **`README.md:3` points to a file that does not exist.** It names `docs/proposal.pdf`, but only `docs/proposal.md` exists. **Update.**
21. **Decision log 28 Oct row says "STEP exports in `hardware/`".** Hardware now lives in `dex1-1-handheld` and CAD exports are not in Git. **Keep**, since it is a historical row, or annotate it.
22. **Decision log 3 Nov row (CAD structure v2) has "Revisit if: D01 settled: drop the unused path's assembly and parts".** D01 settled on 2026-11-16 with the copy kept as fallback. This is already an open TODO in [[2026-11-19]] ("Keep or drop the copy-path Fusion assembly"). **Keep** until Martyn decides.
23. **`.gitignore` ignores some local LaTeX outputs only through generic patterns.** `thesis/main.bbl`, `main.pdf`, `main.synctex`, `main.tcp` and `main.tps` are local build output and are ignored correctly. **Keep.**
24. **`docs/hardware-specs.md` "Known conflicts" item 7 cites 846 g.** It quotes the figure from [[2026-11-16 Yanmei]], which is a meeting record and correct as history. **Keep** the meeting note; drop the item once something has been weighed.
25. **`docs/hardware-specs.md` conflict 1 lists notes that still use 90 or 89 mm stroke.**
    - [[D09 Calibration]] says "90 mm stroke".
    - [[D10 Handheld mechanism]] Evidence says "Stroke 89 mm (URDF joint limits)", while its own History of 2026-11-03 gives 82 mm.
    - This is already flagged there. **Keep** until the stroke is measured on delivery, then **update** D09/D10 with the record-decision skill.

## msc-thesis-vault

### Medium

26. **[[D06 Recording and synchronisation]] Decision is stale on the station Jetson.**
    - It says "approved unit 10.36.1.124, model and location not yet checked; Path 2 assumes an Orin", and Evidence says "Build and 500 Hz loop on the approved unit: untested".
    - Measured on 2026-11-16 ([[2026-11-16 Station Jetson check]], `msc-thesis/docs/hardware-specs.md`): it is a Jetson AGX Orin Developer Kit with GLIBCXX up to 3.4.30.
    - `dex1_1_service` was built and started on 2026-11-18 ([[2026-11-18]]). The 500 Hz loop with a gripper is still untested.
    - **Update** via record-decision. The decision itself is unchanged, so this is a History line only.
27. **[[D10 Handheld mechanism]] "Revisit if" names the wrong coupon.**
    - It says "Insert boss: the hole-size coupon (3.9 / 4.0 / 4.1 mm) shows 4.0 mm too tight or loose".
    - The coupon was 4.0 / 4.1 / 4.2 mm, it is measured, and Ø4.1 was chosen (D10 History 2026-11-18). Only insert fit and pull-out remain.
    - **Update** via record-decision.
28. **[[D10 Handheld mechanism]] Evidence points to old paths.** It cites `msc-thesis/hardware/dex1-1-reference/` (two places). These files now live in `dex1-1-handheld/hardware/dex1-1-reference/`, and the msc-thesis copy is the leftover folder from finding 18. **Update.**
29. **[[D10 Handheld mechanism]] and [[D05 Gripper width sensing]] still describe the copy path as the design.**
    - D10 Decision presents the rack-and-pinion, squeeze trigger and lock as the chosen mechanism.
    - D05 presents "ArUco markers on the printed fingers".
    - With D01 settled on the real Dex1-1, both mainly apply to the fallback.
    - This is already an open TODO in [[2026-11-19]] ("Decisions and Fusion"). **Keep** the TODO; **update** via record-decision when Martyn decides.
30. **[[2026-11-19]] TODOs are stale.** **Update** (human note, so only on Martyn's request).
    - "[ ] Merge dex1-1-handheld PR #10" is still open, but the PR was merged on 2026-11-19.
    - The v22 item ends "superseded for fit by v24 … test them on the chosen v24". v24 was dropped, and comfort and tracking now go on v22 + tape, then v23.
    - "Bow flex check … ([[D10 Handheld mechanism]]: adapter material open)": D10 does not say the adapter material is open. It only lists bow flex under "Revisit if" (unverified link).

### Low

31. **[[Waiting]] Purchases items have small stale parts.** **Update.**
    - The heat-set inserts item says "On arrival: hole-size coupon (4.0 / 4.1 / 4.2 mm, printed 2026-11-16)". The coupon is already measured, so only insert fit and pull-out remain.
    - The screw assortment item still names "20 × M3 × 25" for the adapter. The rear mount needs M3 × 35 since v10, which is inside the M3 × 6–40 range.
32. **Vault `CLAUDE.md` Layout omits `Agent/External/`.** It lists `Agent/` as `Research/, Reports/, Drafts/`, while `Agent/README.md` and the folder also have `External/` (external-report skill). **Update** (needs Martyn's go: CLAUDE.md).
33. **[[D02 Policy camera]] has a typo.** The verdict column reads "doesn't met the expectation". **Update** (record-decision or a direct edit on request).
34. **[[D08 Deployment]] History has two 2026-11-16 lines that disagree.** The first says thesis-plan §2 is "not changed", and the second says it was updated. This is correct as history. **Keep.**
35. **`Literature/` holds 2 notes (`@surendran2026`, `@zeng2025`).** `msc-thesis/thesis/literature.bib` has 28 entries. All 10 `Chapters/` have the status "not started", and the chapter `.tex` files total 102 words. This is information, not a contradiction: thesis-plan §5 plans related work in November. **Keep.**
36. **[[2026-11-18 Relative vs absolute actions decision draft]] is now recorded in [[D07 Software and data format]].** D07 links to it as evidence. **Keep.**

## Cross-repo: branches, worktrees, remotes

### Medium

37. **Merged branches are still on GitHub (msc-thesis).** These 12 remote branches are merged and have 0 commits not in `main`:
    - `claude/d10-insert-boss-log`
    - `claude/project-thread-{1v1knc, 5xtab6, iqir05, j8e4bk, kiorhe, mabrij, scoaza, wbdxlc, wi2sqt}`
    - `claude/task-parts-dimensions`
    - `decision-log-sleeve-petg`

    **Remove** (GitHub "delete branch", or `git push origin --delete`).
38. **`dex1-1-handheld` remote branch `print-queue-dummy-rear-section` is superseded.** It is 1 commit ahead of `main` (6dd398d). Its two added lines (v21 hole 3.9 mm; insert-hole coupon item) are already in `main` (PRINT-TEST-QUEUE lines 97–98, through PR #3). **Remove** the remote and local branch.

### Low

39. **Local branches whose upstream is gone or merged.** **Remove.**
    - msc-thesis: `claude/project-thread-47s7td`, `claude/project-thread-q8yf81` (upstream gone). `claude/d10-insert-boss-log`, `claude/task-parts-dimensions` and `decision-log-sleeve-petg` are merged; the last is 3 behind its remote.
    - dex1-1-handheld: `print-queue-coupon-results`, `print-queue-insert-coupon` and `slicer-settings-h2s` (all merged, upstream gone), and `print-queue-dummy-rear-section` (finding 38).
    - msc-thesis-vault: `claude/project-thread-47s7td`, `claude/project-thread-q8yf81`, `claude/project-thread-wkeqtt` (merged, upstream gone).
40. **Vault worktree and branch from a closed PR.**
    - Branch `claude/project-thread-jf3ac2` has 1 unmerged commit, b0c26fb "Daily 2026-11-11: group TODOs into hardware, software and admin", from closed PR #44.
    - Its worktree under `/private/tmp/claude-501/…/scratchpad/vault` no longer exists, so git marks it "prunable".
    - **Remove:** `git worktree prune`, then delete the branch, unless the 2026-11-11 regrouping is still wanted.
41. **`.DS_Store` files are ignored in all three repos, and `.obsidian/workspace.json` is ignored.** **Keep** (correctly ignored).

## Open

- Finding 1 needs Martyn's answer: is the v23 print with or without the brim?
- Finding 4 needs a rule: is source CAD (`.f3d`/`.step`) in `dex1-1-handheld` Git or not?
- Findings 16, 17 and 32 change the thesis plan or a CLAUDE.md, so Martyn's go is needed first.
- Findings 26–29 change decision notes, so they go through record-decision.
- Not checked: the Fusion state (excluded on purpose), the content of `docs/research-notes.md` and `docs/proposal.md` (the approved baseline, historical by design), daily notes before 2026-11-18, and the Agent/Research notes beyond the paths they are cited from.
