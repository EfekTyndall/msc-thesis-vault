---
date: 2026-11-17
tags: [report, compute, station]
---
# Station Jetson SD card check

Read-only check of the 256 GB SD card found in the station Jetson (10.36.1.124) during [[2026-11-17 Station Jetson setup and install plan]]. Martyn ran the commands over `ssh station-jetson` (own account) on 2026-11-17 around 15:35–15:40 CET. The card was mounted read-only (`mount -o ro`) to `/mnt/sdcard-check`. Nothing was written to the card, and no system configuration was changed.

## Result

The card is **empty**. It holds no files, only the `lost+found` folder that `mkfs.ext4` creates. The owner is still unknown, so Saad needs to be asked before the card is used.

| Item | Measured | Note |
|------|----------|------|
| Device | `/dev/mmcblk1` (model SN256), one partition `mmcblk1p1`, ext4, label `sdcard` | From the setup thread; not mounted at boot |
| Size, use | `df -h`: 234G size, 28K used, 222G available | |
| Contents | Top level: only `lost+found` (root, 16 KB, Jun 28 2025); `find -type f` printed nothing | No user files |
| Created | `tune2fs -l`: Filesystem created Sat Jun 28 14:42:30 2025 | |
| Last mount (read-write) | Tue Aug 26 17:23:31 2025 | Today's read-only mount does not update this field |
| Last write (superblock) | Mon Oct 26 15:03:07 2026 | Cause unknown (unverified). It is close to the last boot (up 21 days on 2026-11-16, [[2026-11-16 Station Jetson check]]). |
| Last mounted on | `<not available>` | No owner clue from the card itself |

## Owner

There is no direct evidence. `/media/saad` exists on the Jetson, and that is the folder the desktop auto-mounter creates for user `saad`. This points to Saad but does not prove it (unverified).

## Open

- Ask Saad whether the card is his and whether it may be used for recordings.
- If yes: run a read/write speed test that writes one new test file and then deletes it. This shows whether the card can hold recordings (SD cards are slower than an SSD; the required rate per episode is not measured yet).
- Whether the card was unmounted at the end of the check is unconfirmed (Martyn was asked to run `sudo umount /mnt/sdcard-check`).
