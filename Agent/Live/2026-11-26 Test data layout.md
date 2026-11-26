---
date: 2026-11-26
tags: [live, data, replay-ladder]
---
# Test data layout (one place: the Mac)

**Example.** The RL4 (Robotiq open/close during replay) real-robot log with the trigger hysteresis rule used to sit in fractal-pc `~/rl2-test/rl4-take-01-hyst.csv`. A copy of the take it replayed sat in the Jetson home, on fractal-pc and on the Mac Desktop. Now the log is at `~/thesis/msc-thesis/data/rl4/2026-11-25-fractal/rl4-take-01-hyst.csv` and the take is at `~/thesis/msc-thesis/data/takes/pose-log-20261125-150606/`. Both exist once, on the Mac.

**Why each part is there.**
- One machine (the Mac): the analysis and the notes are there, and a file can't go stale against another copy of itself (Martyn, 2026-11-26: "Just one place").
- `msc-thesis/data/`: Git already ignores it (`.gitignore`: `data/*` except `DATA.md`, and `*.mcap`), so large files and lab camera images stay out of Git. `data/DATA.md` is the index.
- `takes/`: raw handheld recordings keep the name the recorder gave them, because several phases replay the same take.
- `rlN/<date>-<what>/`: one folder for each Replay Ladder phase and run.

**What it means for the comparison.** Every recording and log that enters the thesis has one path to cite in `DATA.md` and in the experiment records.

## Tree (2026-11-26, 848 MB)

| Folder | Content | From |
|---|---|---|
| `takes/pose-log-20261123-142714/` | Handheld take 01 (RL2 (URSim replay)) | Jetson home |
| `takes/pose-log-20261123-161005/` | Handheld take 02 (RL3 (real UR10e replay)) | Jetson home |
| `takes/pose-log-20261125-150606/` | Handheld take with the Dex1-1 dummy (RL4) | Jetson home |
| `rl1/2026-11-23-synthetic/` | RL1 (MuJoCo replay on the Mac) synthetic input `raw.csv`, `syn-log.csv` | fractal-pc `~/rl2-test` |
| `rl2/2026-11-23-take01/` | URSim joint log `take-01-ursim.csv`, `rl2_take01.script` | fractal-pc `~/rl2-test` |
| `rl3/2026-11-23-take02/` | Real-robot log `take-02-robot.csv` | fractal-pc `~/rl2-test` |
| `rl4/2026-11-25-fractal/` | `rl4-take-01-robot*.csv`, `rl4-take-01-hyst*.csv` | fractal-pc `~/rl2-test` |
| `rl4/2026-11-25-jetson/` | `rl4-take-01-jetson*.csv` | Jetson home |
| `rl5/2026-11-25-d435i-frame/` | D435i test frame (`.png`, `.ppm`) | Mac Desktop |
| `rl5/2026-11-26-r4-pi/` | R4 (60 s camera take, camera on the Pi) `take-01.mcap` | Jetson `~/rl5-r4` |
| `rl5/2026-11-26-r4b-jetson/` | R4b (D435i on the Jetson) `take-01.mcap`, inspect output `takeA-1250-old-recorder.csv` | Jetson `~/rl5-r4-jetson`, Mac Desktop |
| `rl5/2026-11-26-retest/` | Retest `take-01..03.mcap`, `takeB1..B3-*.csv`, `rl5-frame-gaps.png` | Jetson `~/rl5-r4-jetson-b`, Mac Desktop |

Moved 2026-11-26 (Martyn: "move it accordingly and delete accordingly"). Each file's MD5 on the Mac matched its source before the source was deleted. Also deleted: copies that matched by MD5 (Mac Desktop, fractal-pc `~/rl2-test`); old script and model copies in fractal-pc `~/rl2-test` (the repo clone has newer ones); Jetson `~/rl5-recorder` (older than the PR #24 branch, compared file by file); Jetson `~/wrist` and `~/venvs/wrist` (unused RealSense env). Not touched: `~/team`, `~/venvs/xrobotoolkit`, the conda envs, and the station Pi. `rl5-frame-0900.jpg` was not found on the Mac.

## Rules for new data

- Record on whatever machine is at the robot, then copy to the Mac (`rsync -a station-jetson:<folder> ~/thesis/msc-thesis/data/<phase>/<date>-<what>/`). Check the MD5 and delete the original on that machine.
- To rerun a replay on the Jetson, copy the take back first (`scp -r`) and delete it again afterwards.
- Code lives only in git clones (`~/dex1-1-handheld`). Don't keep loose copies in home folders.
- Add one row to `msc-thesis/data/DATA.md` for each session.
- Most of this is test data. It is kept for now and deleted once it isn't needed (Martyn, 2026-11-26).
