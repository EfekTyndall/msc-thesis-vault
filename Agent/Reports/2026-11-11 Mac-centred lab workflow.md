# Mac-centred lab workflow

How Martyn works on the MacBook only, with the lab Linux PC (`fractal-pc`), the PICO headset and the robot reached over the lab network. Agreed points from the thread "Plan the MacBook, lab PC, PICO and robot workflow" (2026-11-11). Nothing is installed or changed yet.

Facts from Martyn (2026-11-11): the lab (ARENA2036) network is separate from the company internal network. On it: MacBook (lab Wi-Fi), UR10e, `fractal-pc` (shared with other students, own user account, `ssh fractal-pc` lands in it), PICO 4 Ultra. `fractal-pc` reaches github.com. The Dex1-1 USB-to-RS-485 adapter plugs into the PC.

## Picture

```mermaid
flowchart LR
  subgraph Mac["MacBook (Wi-Fi): where Martyn works"]
    F[Fusion + Bambu Studio]
    V[VS Code: code, LaTeX]
    S[MuJoCo sim]
    C[Claude Code / Claude app]
    R1[(~/dex1-1-handheld)]
  end
  G[(GitHub: dex1-1-handheld)]
  subgraph PC["fractal-pc (cable): runs things"]
    R2[(~/dex1-1-handheld clone)]
    X[XRoboToolkit PC Service]
    RC[robot + gripper control]
  end
  P[PICO 4 Ultra]
  U[UR10e]
  D[Dex1-1]
  R1 -- push --> G -- pull --> R2
  V -- Remote-SSH --> PC
  C -- ssh fractal-pc --> PC
  P -- Wi-Fi --> X
  RC -- RTDE --> U
  RC -- USB / RS-485 --> D
  X -. UDP trigger, tests only .-> S
```

## Rules

1. **Git is the single source.** Code is written in `~/dex1-1-handheld` on the Mac (split from `msc-thesis` on 2026-11-11, msc-thesis PR #23) and pushed to GitHub; the PC has its own clone and runs `git pull` before a run. Replaces the scp copy of `software/xr/trigger_udp.py` in `~/xr_trigger`, which can drift from the Mac original.
2. **PC access from the Mac only.** Terminal: `ssh fractal-pc`. Editor, terminal and file tree on the PC inside VS Code: [Remote-SSH](https://code.visualstudio.com/docs/remote/ssh). It installs a server in the home folder on the PC ([VS Code Server](https://code.visualstudio.com/docs/remote/vscode-server)), no sudo; it is still a new install on a shared PC, so it waits for Martyn's go.
3. **Claude stays on the Mac.** A Remote Control thread on the Mac runs `ssh fractal-pc …` in Martyn's shell (used for the survey and setup on 2026-11-09). This may make the Claude Code install in [[2026-11-09 Lab workstation Claude Code plan]] unnecessary. Yanmei's OK is still needed, since Claude would run commands on a company PC.
4. **Real-time work runs on the PC, not over Wi-Fi.** XRoboToolkit has no macOS support; RTDE on e-Series runs at up to 500 Hz ([UR RTDE guide](https://www.universal-robots.com/articles/ur/interface-communication/real-time-data-exchange-rtde-guide/)); the Dex1-1 is on the PC's USB. The Mac only watches, edits and runs the sim. Wi-Fi latency between PC and Mac: unmeasured.

| Where | What |
| --- | --- |
| Mac | Fusion, Bambu Studio, MuJoCo sim, LaTeX, vault, all code editing, Claude |
| PC | XRoboToolkit PC Service, UR10e control, Dex1-1 over RS-485, recording, later GPU training |

## Setup steps

1. **Done 2026-11-11:** read-only deploy key `~/.ssh/dex1_1_handheld_deploy` on `fractal-pc` (GitHub title `fractal-pc`, no write access), ssh alias `github-dex1` in `~/.ssh/config`; `ssh -T github-dex1` authenticates. Options were a read-only [deploy key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/managing-deploy-keys#deploy-keys) for `dex1-1-handheld` (PC can only pull; all edits on the Mac) or a personal key (PC can push too); Martyn chose the deploy key.
2. **Clone done 2026-11-11:** `~/dex1-1-handheld` on the PC (`git clone github-dex1:EfekTyndall/dex1-1-handheld.git`). Open: point the trigger sender at the repo copy instead of `~/xr_trigger`.
3. **Done 2026-11-11:** Remote-SSH extension on the Mac, connected to `fractal-pc`, folder `~/dex1-1-handheld`, interpreter `~/anaconda3/envs/xrobotoolkit/bin/python`. Edits stay on the Mac (PC key is read-only).
4. **Claude in VS Code (Martyn, 2026-11-11):** Claude Code panel only in the Mac window (`~/dex1-1-handheld` local); it reaches the PC with `ssh fractal-pc …`. The Remote-SSH window to the PC has no Claude: in a remote window the extension most likely runs on the PC itself ([VS Code remote extensions](https://code.visualstudio.com/api/advanced-topics/remote-extensions); not covered by the [Claude Code VS Code docs](https://code.claude.com/docs/en/vs-code); unverified), which is the open Claude-on-PC question.
5. **Claude rules in VS Code (2026-11-12, tested by Martyn):** `~/dex1-1-handheld/CLAUDE.local.md` imports the project-instructions copy and both CLAUDE.md files; `.claude/settings.local.json` gives read access to `~/msc-thesis` and `~/msc-thesis-vault`; both hidden with `.git/info/exclude`. Master copies and install steps: `msc-thesis/docs/setup/claude-project/dex1-1-handheld-local/`. The project-instructions copy is updated whenever the instructions change.

## Open

- Where raw recordings (MCAP) are stored and how they reach the Mac or training; raw data stays out of Git.
- Yanmei's OK for Claude running commands on the PC.
- **Later topic (Martyn, 2026-11-11):** can a Raspberry Pi replace the PC for the Dex1-1 / RS-485 link? Answered in [[2026-11-12 Raspberry Pi vs lab PC for the Dex1-1]]. Correction (2026-11-12): fractal-pc is far from the robot station, so the Dex1-1 is not plugged into it; it goes on a station computer (teleop Jetson or a station Pi), as in the existing VR-teleop setup.
