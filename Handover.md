---
tags: [handover]
---
# Handover of dex1-1-handheld (thesis end)

**When:** at the end of the thesis or of the Daimler contract, before `dex1-1-handheld` is shared with the company. It is listed in [[Waiting]] under "Thesis end".

**Kept current by:** the weekly cloud audit (Sundays), which runs the check below and adds new hits to section 2.

**Rule until then:** no new vault IDs, vault paths, thesis-repo paths or Claude mentions go into `dex1-1-handheld`. If one is unavoidable, add it to the list below.

## 1. Re-run the check (finds everything, also items added later)

```
cd ~/dex1-1-handheld
git grep -nEi '\bD[0-9]{2}\b|vault|Agent/|msc-thesis|thesis plan|thesis-plan|daily note|design notes|research gaps|PR #|claude|anthropic|experiments/' -- . ':!*.STL' ':!*.urdf'
git log --format='%h %an <%ae> | %s' | grep -i claude          # authored by Claude
git log --format='%h %B' | grep -i 'co-authored-by: claude'    # co-author lines
git ls-files -o --exclude-standard; git status --ignored --short # local-only files
```

## 2. Known items (snapshot 2026-11-19, from [[2026-11-19 Three-repo consistency audit]])

Vault and thesis references in tracked files:
- [ ] `hardware/controller-grip/README.md:3`: "(vault: D10)"
- [ ] `hardware/dex1-1-reference/README.md:3`: "(vault: D01, D02, D10)"; `:7` "(daily note 2026-11-03)"; `:8` "measurements in vault D10"
- [ ] `hardware/dex1-1-reference/from-unitree-ros/README.md:3`: "(vault: D01, `Agent/Reports/…`, step 2)"
- [ ] `hardware/pico-4-ultra-glock-shell/README.md:3`: "(vault: D10)"
- [ ] `hardware/pico-4-ultra-scan/README.md:3`: "(vault: D04, D10)"
- [ ] `software/xr/trigger_udp.py:8`: "(vault: D04, research gaps B7)"
- [ ] `hardware/PRINT-TEST-QUEUE.md`: the vault note path in the v19 result; "`docs/thesis-plan.md` §7" (§2 intro); "full note in the design notes" (plate 2b); "(vault PR #31)" (joint item); "(thesis plan, handheld design start plan step 3)" (§4); "`experiments/<YYYY-MM-DD>-<name>/`" (intro)

Internal values:
- [ ] `software/xr/trigger_udp.py` usage example: Mac IP `10.36.1.125`; replace with a placeholder
- [ ] Personal names in test notes ("Martyn", e.g. `PRINT-TEST-QUEUE.md`, `SLICER-SETTINGS.md`): keep or neutralise (company's choice)

Git history and GitHub:
- [ ] 34 commits authored by `Claude <noreply@anthropic.com>` and 55 `Co-Authored-By: Claude` lines on `main`; 57 co-author lines counting the open PR #12 branch (count of 2026-11-19, weekly audit; re-count with step 1)
- [ ] PR history: PRs #1–#12 came from `claude/…` and similar branches; PR titles and bodies stay on GitHub
- [ ] Decide: rewrite history (`git filter-repo` for authors and message lines) or hand over a fresh repository with one initial commit. A fresh repository drops both the history and the PR list

Local-only files on the Mac (never in Git, never in a zip of the folder):
- [ ] `CLAUDE.local.md`, `.claude/` (excluded via `.git/info/exclude`)

CAD and print files (not in Git while in development):
- [ ] `hardware/real-adapter/files/real-adapter-v9.f3d` and `real-adapter-v9.step` were tracked on `main` from b8ae783 (2026-11-16) until PR #12 removed them (merged 2026-11-19); they stay in the Git history, so the handover method must drop them. The step 1 grep reports "Binary file …real-adapter-v9.f3d matches" (compressed data, strings "D28" and "d05"; no real reference)
- [ ] Export only the latest version of each part (STEP, plus STL if wanted); no earlier versions
- [ ] Licences: Unitree STEP/URDF derivatives (dummy, dummy shell) have no stated licence: internal use only, not shared; PICO controller scan (CC BY) needs the author credit; Glock shell (CC BY 4.0) the same

## 3. Done

- 
