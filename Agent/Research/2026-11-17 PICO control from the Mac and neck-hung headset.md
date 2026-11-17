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

## Results 2026-11-17 evening (Steps 1 to 3)
- Step 1: Homebrew installed scrcpy 5.0.1 and adb 1.0.41 (platform-tools 37.0.1) on the Mac.
- Step 2: PICO listed by `adb devices` over USB (model A9210, Android 14, PICO OS build 5.15.7). Screen 4320×2160, both eyes side by side; `--crop=2160:2160:0:0` shows the left eye. The picture shows in the Mac window.
- Picture goes black after a while with the headset off the head. Read-only readings: `screen_off_timeout` 60000 ms; `dumpsys power` shows a PICO "PvrManagerProximityPowerPolicyController" taking wake locks; props `persist.psensor.sleepmode=2`, `persist.pvr.sleep_by_static=1`. Likely PICO's proximity/static sleep (unverified). `--stay-awake` set `stay_on_while_plugged_in=7` (reset by scrcpy on exit); effect on the blackout not yet confirmed.
- Step 3 fails: mouse clicks in the scrcpy window do not press XRoboToolkit buttons (Martyn, 19:04).
- XRoboToolkit app ([README](https://github.com/XR-Robotics/XRoboToolkit-Unity-Client)): package `com.xrobotoolkit.client`; can be launched with `adb shell monkey -p <package> -c android.intent.category.LAUNCHER 1`; "Data & Control remembers the last valid IPv4 confirmed for PC Service" but "never persists ... an automatic-connect instruction". No intent extras or remote command for Tracking/Send found. Source folder not readable (robots.txt).
- Second try `--mouse=uhid` (Mac mouse as USB mouse) not confirmed. Martyn's choice (19:07): press XRoboToolkit buttons with the controller; scrcpy only to watch. Open: Step 4 (Wi-Fi), Step 5 (neck test, proximity sleep).
- Confirmed (Martyn, 19:07): picture still goes black off the head with `--stay-awake` on, so the blackout is the PICO proximity sleep, not the Android timeout. Next option: tape over the sensor (needs Martyn's OK).
- Sensor location: small oval window inside the headset, centred between the lenses above the nose cut-out ([PICO 4 Ultra user guide EU, p. 8, item 12 "Proximity Sensor"](https://p16-platform-static-va.ibyteimg.com/tos-maliva-i-jo6vmmv194-us/pico4-ultra-user-guide-eu.pdf)). Note: `persist.pvr.sleep_by_static=1` suggests it may also sleep when not moving (unverified).
- Tape test (Martyn, 19:23): opaque tape over the proximity sensor, headset off the head, scrcpy with `--stay-awake` over USB: picture did not go black. Not yet tested without the USB cable or with the headset hung on the neck (Step 5).
- Neck test, short (19:27, Step 5 variant without sim): headset on the neck with sensor taped, connected to the station Jetson PC Service; SDK log at 30 Hz for 120 s (/tmp/neck_test_2026-11-17.csv on the Jetson, not kept yet). 3600 samples, SDK timestamp spans 119.98 s; timestamp did not advance in 16 of 3599 samples (0.4 %), longest stall 0.27 s; no zero poses. One position step > 0.05 m (0.077 m at t = 34.5 s, fast move or tracking jump, unknown); all others < 0.04 m. Range x 0.30, y 0.31, z 0.48 m (SDK frame). Trigger 0 to 1. SDK needed 44 s to "device found" (6.5 s in the B2 test). Partial pass: continuity only, not accuracy; 10 min run not done.
