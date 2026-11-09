# Lab workstation: Claude Code with Remote Control

Plan to run Claude Code directly on the lab Linux PC (`fractal-pc`) with Remote Control, so project threads can work on it instead of relaying every command through `ssh fractal-pc` from the Mac. Starts after the XRoboToolkit install (thread "Test PICO trigger on simulated gripper") is finished. Nothing is installed yet; installing Claude Code is a new dependency and waits for Martyn's go.

Sources: `msc-thesis/docs/setup/SETUP.md` §6 (the original checklist); the read-only survey of `fractal-pc` on 2026-11-09 (Ubuntu 22.04.5 x86_64, Python 3.10, conda in `~/anaconda3`, IP 10.36.1.102, ufw active, sudo needs a password, `nvidia-smi` fails); official docs [setup](https://code.claude.com/docs/en/setup), [authentication](https://code.claude.com/docs/en/authentication), [Remote Control](https://code.claude.com/docs/en/remote-control), [network config](https://code.claude.com/docs/en/network-config), read 2026-11-09.

## 1. Original checklist (`SETUP.md` §6) status

| Step | Status | Note |
| --- | --- | --- |
| 1. Clone the repo, set up Python and the device SDKs | Partly open | Python 3.10 and conda present. Repo clone on the PC: unchecked. XRoboToolkit PC service + `xrobotoolkit-sdk` install in progress (other thread). `unitree_sdk2` waits for Martyn's go (Daily/2026-11-09). |
| 2. Claude desktop app → "Use this computer from your phone and claude.ai" (or `claude remote-control`) | Open | The PC has no GUI, so the desktop-app route does not apply; use the CLI `claude remote-control` (same step, second option). This plan. |
| 3. Fill `msc-thesis/data/DATA.md` as recordings are made | Open, not due | No recordings yet. |
| 4. Ask Yanmei to confirm that thesis code and data may live in the personal GitHub and be processed by cloud AI tools | Open, status unknown | No record found in the vault (Daily, Meetings, Waiting). Running Claude Code on a company PC sends file contents and command output from that PC to Anthropic, so this confirmation (and possibly company IT's) matters more now. |

Not in the checklist, seen in the survey: NVIDIA driver broken (`nvidia-smi` fails). Not needed for Claude Code or XRoboToolkit; needed later for training.

## 2. Install and sign-in steps (from the official docs)

Requirements ([setup](https://code.claude.com/docs/en/setup)): Ubuntu 20.04+, 4 GB+ RAM, x64; no Node.js for the native installer. Remote Control needs a Pro, Max, Team or Enterprise claude.ai login, not an API key; on Team/Enterprise an Owner must enable it in admin settings ([Remote Control](https://code.claude.com/docs/en/remote-control)). Which plan Martyn's account is on: unverified.

1. **Network check (read-only, Claude):** `curl -sI https://downloads.claude.ai` and `curl -sI https://api.anthropic.com` from `fractal-pc`. Remote Control is outbound HTTPS only and opens no inbound port, so no ufw rule is needed ([Remote Control](https://code.claude.com/docs/en/remote-control)). If a company proxy blocks these hosts, stop here (the docs list the hosts to allowlist: [network config](https://code.claude.com/docs/en/network-config)).
2. **Install (Claude, no sudo expected):** `curl -fsSL https://claude.ai/install.sh | bash`. The docs do not state the install path; the uninstall steps imply `~/.local/bin/claude` and `~/.local/share/claude`. Auto-updates in the background; `claude doctor` checks the install.
3. **Sign in (Martyn, once):** in `ssh fractal-pc`, run `claude auth login` (or `claude`, then `/login`). With no browser on the PC, press `c` to copy the login URL, open it on the Mac, and paste the code shown in the browser back at `Paste code here if prompted` ([authentication](https://code.claude.com/docs/en/authentication)). Credentials go to `~/.claude/.credentials.json` (mode 0600). `claude setup-token` cannot be used: such tokens "can't establish Remote Control sessions".
4. **GitHub access on the PC (Martyn):** for cloning and pushing the repos, an SSH key on `fractal-pc` added to Martyn's GitHub account (`ssh-keygen -t ed25519`, then paste the `.pub` into GitHub → Settings → SSH keys). Then clone `msc-thesis` (and the vault if wanted) to `~/msc-thesis`.
5. **Start Remote Control (Martyn, first time; Claude after):** `tmux new -s claude`, then `cd ~/msc-thesis && claude remote-control --name fractal-pc`. The first run asks `Enable Remote Control? (y/n)` and `Trust <directory>? [y/N]`; both need Martyn at the terminal. Detach with `Ctrl-b d`; the docs recommend `tmux` or `screen` so the session survives the SSH disconnect. Whether `tmux` is installed on the PC: unchecked (installing it needs sudo).
6. **Check:** the session appears in claude.ai/code and the Claude app with a computer icon and a green dot. Then a project thread is started on `fractal-pc` and runs one read-only command (`hostname`, `nvidia-smi`). Whether project threads list a server-mode `claude remote-control` session as a device the same way as the Mac: unverified until this test.

To undo: stop the tmux session, `rm -f ~/.local/bin/claude`, `rm -rf ~/.local/share/claude ~/.claude ~/.claude.json` ([setup](https://code.claude.com/docs/en/setup)), remove the SSH key from GitHub.

Limits ([Remote Control](https://code.claude.com/docs/en/remote-control)): the `claude` process must keep running; after a stop, `claude remote-control --continue` restores sessions for about four hours. After a reboot of the PC, Martyn (or a thread via the Mac) restarts it in tmux.

## 3. What Martyn does himself

- Give the go to install Claude Code on `fractal-pc` (new dependency).
- Confirm with Yanmei (checklist step 4), and with company IT if needed, that a company PC may run a cloud AI agent on thesis code and data.
- Sign in once (step 3), add the SSH key to GitHub (step 4), answer the two first-run prompts (step 5).
- Any `sudo` step (e.g. installing `tmux`), since sudo needs his password.

## Open

- Plan type of Martyn's claude.ai account, and whether an Owner toggle is needed.
- Whether company network policy allows the hosts in step 1.
- `SETUP.md` §6 step 2 still describes the desktop-app route; updating it to the CLI route is optional, after the test.
