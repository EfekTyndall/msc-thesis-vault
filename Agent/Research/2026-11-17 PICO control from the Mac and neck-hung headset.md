---
date: 2026-11-17
tags: [research, pico, xrobotoolkit, scrcpy, plan]
---
# PICO control from the Mac and neck-hung headset

Question (Martyn, 2026-11-17): (1) Can the XRoboToolkit app on the PICO 4 Ultra Enterprise be seen and operated from the Mac, so there is no need to switch between devices? (2) Can the headset hang around the neck and still track the controller?

Timing: plan only. Nothing is installed or tested until install items B1 and B2 on the station Jetson are done (see Agent/Reports/2026-11-17 Station Jetson setup and install plan.md) and Martyn says go.

Method: official docs read on 2026-11-17: [scrcpy README](https://github.com/Genymobile/scrcpy), its [macOS](https://github.com/Genymobile/scrcpy/blob/master/doc/macos.md), [connection](https://github.com/Genymobile/scrcpy/blob/master/doc/connection.md), [device](https://github.com/Genymobile/scrcpy/blob/master/doc/device.md) and [video](https://github.com/Genymobile/scrcpy/blob/master/doc/video.md) pages; [PICO 4 Series user guide (PDF)](https://pico-web-tob.oss-cn-beijing.aliyuncs.com/20230825/document/1695015503416348672.pdf); [PICO 4 Ultra specs](https://www.picoxr.com/nl/products/pico4-ultra/specs); [picoxr/pico-power-manager](https://github.com/picoxr/pico-power-manager); [PICO developer site](https://developer.picoxr.com/) (home page only, nothing found there). No paper in `msc-thesis/thesis/literature.bib` covers this. Nothing was tried on the headset.

## Short answers

1. **Mac control: yes, very likely.** The PICO runs an Android-based OS, and developer mode is already on. The free tool **scrcpy** (Apache-2.0) shows an Android screen in a Mac window and sends mouse clicks and keys back. Needs Android 5.0 or newer and USB debugging ([README](https://github.com/Genymobile/scrcpy)). It works over the USB-C cable or over Wi-Fi. Unverified on the PICO 4 Ultra: whether a mouse click in the window reaches buttons inside the XRoboToolkit VR panel (scrcpy sends touch events; VR apps normally expect the controller ray).
2. **Neck-hung: unknown, needs a test. Two catches:**
   - **Proximity sensor.** The headset "sleeps when VR headset is taken off" and wakes when put on; a while later it enters deep sleep and the controllers shut down ([user guide, p. 6 and p. 8](https://pico-web-tob.oss-cn-beijing.aliyuncs.com/20230825/document/1695015503416348672.pdf)). On the neck the sensor sees no face, so the headset will probably sleep and tracking stops. No official setting to disable this was found (user guide, developer site, pico-power-manager README). scrcpy's `--stay-awake` changes the normal Android screen timeout, not the proximity sleep, and does nothing over Wi-Fi only ([device.md](https://github.com/Genymobile/scrcpy/blob/master/doc/device.md)). Common workaround in the VR community: a small piece of opaque tape over the sensor between the lenses (no official source; unverified on PICO 4 Ultra; fully reversible).
   - **Camera view.** Controllers are tracked by the headset's 4 environment cameras (inside-out, [specs](https://www.picoxr.com/nl/products/pico4-ultra/specs)); the guide says "Tracking Cameras / Do not block during use". On the neck the cameras point forward or down from chest height, so the controller must stay in front of the chest, and the headset's own world tracking sees mostly body and floor. Whether the controller pose stays stable is unverified. This is a measurement question for the thesis too: VR data recorded with a neck-hung headset is not the same set-up as worn VR teleoperation.

## Plan (after B1 + B2, on Martyn's go)

**Step 0, OK needed.** Installing scrcpy and adb on the Mac is a new dependency (Homebrew). Ask Martyn first.

**Step 1, install on the Mac (2 commands, ~2 min):**
```
brew install scrcpy
brew install --cask android-platform-tools
```
(from [macos.md](https://github.com/Genymobile/scrcpy/blob/master/doc/macos.md); `android-platform-tools` gives `adb`, the tool that talks to Android devices.)

**Step 2, first link over USB.** Plug the PICO into the Mac with a USB-C data cable. Put the headset on once: a prompt "Allow USB debugging?" should appear; tick "always allow" and accept. Then on the Mac:
```
adb devices          # PICO must be listed as "device"
scrcpy --no-audio
```
A window shows both eyes side by side. To show one eye only, read the size with `adb shell wm size` and crop to the left half, e.g. `scrcpy --no-audio --crop=W/2:H:0:0` (the docs use this for the Oculus Go; [video.md](https://github.com/Genymobile/scrcpy/blob/master/doc/video.md)).

**Step 3, check control.** In the scrcpy window, open XRoboToolkit and try its buttons with the mouse (Tracking → Controller On, Data & Control → Send On). Record whether clicks work.

**Step 4, Wi-Fi (no cable).** With the cable still in: `scrcpy --tcpip --no-audio`. It switches the PICO to Wi-Fi debugging and connects; then unplug ([connection.md](https://github.com/Genymobile/scrcpy/blob/master/doc/connection.md)). Mac and PICO must be on the same network. After a headset reboot this step is needed again (Android behaviour, unverified on PICO).

**Step 5, neck test (~10 min).** With the MuJoCo Dex1-1 sim running (`software/sim/run_dex1_1.sh`) and scrcpy open:
1. Hang the headset on the neck, wait 2 min. Does the scrcpy picture go black (sleep)? Does the sim still get controller pose and trigger?
2. If it sleeps: try a small piece of masking tape over the proximity sensor (only with Martyn's OK), repeat.
3. Move the controller in a typical work area in front of the chest and to the side. Note where tracking is lost.
Pass if: the headset stays awake for 10 min on the neck and the sim follows the controller in the work area without jumps.

**Fallback if the neck test fails:** wear the headset on the forehead (visor up) or on the head, and use only the scrcpy window for app control. Step 1 to 4 are useful either way.

## Open
- Clicks into the XRoboToolkit VR panel via scrcpy: unverified until Step 3.
- Proximity-sensor sleep on the neck and any disable option: unverified; PICO Business device management (Enterprise headset) may offer more settings, not checked.
- Effect of a neck-hung headset on pose quality and on comparability with worn VR teleop: to discuss before using it for recordings.
