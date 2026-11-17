---
date: 2026-11-17
tags: [research, printer, bambu, firmware]
---
# Bambu Studio and H2S firmware match

Question: Flow Rate calibration started from Bambu Studio's Calibration tab on the Mac gives "please update Bambu Studio and try again". How to check that Studio and the H2S (with AMS 2 Pro) firmware match, which Studio the current firmware needs, how to update, how long it takes, and when it is safe.

Method: official pages read in a browser on 2026-11-17: [H2S firmware download page](https://bambulab.com/en/support/firmware-download/h2s), [H2S firmware release history](https://wiki.bambulab.com/en/h2s/manual/h2s-firmware-release-history), [H2 firmware update guide](https://wiki.bambulab.com/en/h2/software/firmware-update), [H2 offline firmware update guide](https://wiki.bambulab.com/en/h2d/manual/offline-firmware-update), [Bambu Studio releases on GitHub](https://github.com/bambulab/BambuStudio/releases). Nothing was checked on the printer or the Mac.

## Current versions (2026-11-17)

| Item | Latest | Source |
|---|---|---|
| H2S printer firmware | **01.03.00.00, released 2026-11-16** (yesterday) | [download page](https://bambulab.com/en/support/firmware-download/h2s) |
| Previous H2S firmware | 01.02.00.00 (2026-03-31), 01.01.02.00 (2026-02-09) | [release history](https://wiki.bambulab.com/en/h2s/manual/h2s-firmware-release-history) |
| Bambu Studio (public) | **02.09.01.70 (2.9.1), 2026-11-03**; 2.8.4 and 2.8.3 were betas | [GitHub releases](https://github.com/bambulab/BambuStudio/releases) |
| Studio on Martyn's Mac | 02.09.01.70 as of 2026-11-10 (BambuStudio.conf); not re-checked today | project memory |
| AMS 2 Pro firmware | latest number not published on the pages read; 01.03.00.00 says remaining-filament estimation "requires … AMS 2 Pro 05.00.22.22 or later" | [download page](https://bambulab.com/en/support/firmware-download/h2s) |

## Which Studio the firmware needs

Bambu does not publish one minimum Studio version per firmware. It names a minimum per feature:

- 01.03.00.00: E3D High-Flow hotend "Requires Bambu Studio v2.8.3.66 or later"; heatbed/aux-fan fix "requires Bambu Studio 2.8.4 or later".
- 01.02.00.00: Purify Air and filtering mode need Studio 2.5.0.64+; nozzle clumping detection by probing needs 2.5.2.65+.
- 01.01.00.00: build plate position detection needs 2.2.1.58+.

So Studio 2.9.1.70 meets every requirement listed for the current H2S firmware. Nothing in the release notes mentions flow-rate or flow-dynamics calibration needing a newer Studio.

## Is the error a known H2S issue?

**Not found.** No official page (wiki, release notes) and no forum thread found mentions "please update Bambu Studio and try again" for the H2S calibration tab. Older forum threads report broken Studio calibration after firmware updates on other printers (e.g. [X1 firmware 01.05.00.00](https://forum.bambulab.com/t/latest-01-05-00-00-firmware-broke-calibration-from-bambu-studio/46303), [H2D flow dynamics bug](https://forum.bambulab.com/t/bambustudio-flow-dynamics-h2d-bug/169364); forum reports, not checked).

Possible causes, all **unverified**:
1. The Mac is no longer on 2.9.1.70 or the Studio network plugin is outdated (Studio may ask to update it separately on start).
2. The printer already took firmware 01.03.00.00 (out 2026-11-16) and the calibration call expects something 2.9.1 does not have.
3. The printer is still on an older firmware and the message is misworded.

Step one is to read both versions (below); that tells which case applies.

## How to check both versions

- **Printer firmware, on the screen:** Settings > Firmware (path named in the [offline update guide](https://wiki.bambulab.com/en/h2d/manual/offline-firmware-update)). The update page lists all components, printer and AMS ("This will update all the necessary components in the list", [H2 update guide](https://wiki.bambulab.com/en/h2/software/firmware-update)).
- **Printer firmware, in Studio:** Device tab > Update shows the installed printer and AMS versions (unverified, not on an official page read).
- **Studio version on the Mac:** menu Bambu Studio > About Bambu Studio (unverified, standard macOS layout). Compare with the latest tag on [GitHub releases](https://github.com/bambulab/BambuStudio/releases).
- **Match:** printer firmware = 01.03.00.00 and Studio ≥ 2.8.4 (today 2.9.1.70) means both are current.

## Update steps

1. Studio: download the Mac build from [bambulab.com/en/download/studio](https://bambulab.com/en/download/studio) (or accept Studio's own update prompt) and reinstall; restart Studio and accept any network-plugin update.
2. Printer: when online and bound to the account, the screen shows an update prompt; Confirm, then **Update** in the Update menu. Do not cut power; the printer restarts and shows a confirmation ([H2 update guide](https://wiki.bambulab.com/en/h2/software/firmware-update)).
3. Offline alternative: FAT32 USB drive, package from the download page, Settings > Firmware > Update Offline ([offline guide](https://wiki.bambulab.com/en/h2d/manual/offline-firmware-update)).
4. Note from the release history: after 01.02.50.00+ "external spool filament information will be cleared. Please set it up again." 01.02.00.00 also asked for a motor noise recalibration after updating.

## How long

- Online update: "This process can take a few minutes" ([H2 update guide](https://wiki.bambulab.com/en/h2/software/firmware-update)).
- Offline update: "The upgrade takes 20-30 minutes" ([offline guide](https://wiki.bambulab.com/en/h2d/manual/offline-firmware-update)).
- AMS 2 Pro: updated in the same run; no separate time published (unverified). Plan **30 min** of printer downtime, plus a few minutes for Studio.

## When it is safe

- Printer idle: no print running or queued, no calibration running (the update page is reached only from the idle menu; doing it mid-print is not described anywhere, so do not).
- Power stays on for the whole update ("do not disconnect the power supply", [H2 update guide](https://wiki.bambulab.com/en/h2/software/firmware-update)).
- Ideally the end of a lab day or a gap of ≥ 30 min; rerun Flow Rate calibration afterwards.

## Open

- Read the actual printer firmware and Studio version on the Mac/printer to tell which cause applies.
- AMS 2 Pro installed firmware vs 05.00.22.22.
