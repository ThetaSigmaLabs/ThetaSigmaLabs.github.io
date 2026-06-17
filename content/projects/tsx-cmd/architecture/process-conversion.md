+++
title = 'Process conversion'
description = 'Migration status of every registered tsx_cmd process onto the driver-agnostic Manager layer.'
draft = true
+++

Every operation against a piece of hardware should flow through the
**Manager → Adapter → Process** stack. Today ~40 processes are
registered with the orchestrator and many still hardcode TheSkyX as
the backend (extending a TSX base class and/or importing the TSX
client directly), or only use a Manager conditionally and fall back to
a direct TSX call when it is absent — so they cannot run against
INDIGO, ASCOM, or the simulator.

This page tracks each registered process until it either:

1. routes through its corresponding `*Manager` (the Manager picks the
   active adapter — TheSkyX, INDIGO, ASCOM, or Simulator), or
2. is marked **N/A** because it is pure control-flow
   (orchestrator/profile selection) and never touches hardware.

## What "converted" means

A fully converted process:

* extends the plain `Process` base class — not a TSX base class,
* has zero imports of the TheSkyX client,
* gets the active adapter via its Manager and calls Manager methods
  only,
* has matching TheSkyX, Simulator, and (where applicable) INDIGO
  adapter implementations behind that Manager, so the same process
  works against any backend.

## Status

Legend (matches the site-wide
[compatibility matrix](../../#compatibility-matrix)):
✅ converted or N/A · 🚧 in progress · 📋 not started

| Status | Count |
| ------ | :---: |
| ✅ Converted / N/A | 6 |
| 🚧 In progress | 18 |
| 📋 Not started | 18 |

### 📋 Not started — TSX-direct, no Manager

These talk to TheSkyX directly with no Manager involvement. Each needs
a Manager method (and adapter ops) created and wired, then the process
body rewritten as a thin Manager call.

| Process | Source | Target Manager | Notes |
| ------- | ------ | -------------- | ----- |
| `CameraTakeImage` | `processes/devices/camera/cameraTakeImage.js` | `CameraManager` | Wrap into `takeImage(opts)`; route through TSX / Simulator / INDIGO camera adapters. |
| `GuiderTakeImage` | `processes/devices/camera/cameraTakeImage.js` | `GuidingManager` | As above, for the autoguider camera. |
| `CameraImageSettingsProcess` | `processes/devices/camera/imageSettingsProcess.js` | `CameraManager` | Add `applyImageSettings(...)`. |
| `AutoguiderImageSettingsProcess` | `processes/devices/camera/imageSettingsProcess.js` | `GuidingManager` | Guider variant of image settings. |
| `ImageDetailsProcess` | `processes/devices/camera/ImageDetailsProcess.js` | `CameraManager` | Manager pulls last-image metadata. |
| `FocuserAutoFocusProcess` | `processes/devices/focuser/AutoFocusProcess.js` | `FocuserManager` | Add `runAutoFocus()`. |
| `RecheckFocusProcess` | `processes/devices/focuser/AutoFocusProcess.js` | `FocuserManager` | `recheckFocus()` — likely shares a method with auto-focus. |
| `DitherProcess` | `processes/devices/mount/DitherProcess.js` | `MountManager` / `GuidingManager` | Decide owner: mount-driven but coordinates with the guider. |
| `StopGuidingProcess` | `processes/devices/autoguiding/StopGuidingProcess.js` | `GuidingManager` | Wire `stop()` through the adapter. |
| `ResumeGuiderProcess` | `processes/devices/autoguiding/PauseGuiderProcess.js` | `GuidingManager` | `resume()`. |
| `PauseGuider` | `processes/devices/autoguiding/PauseGuiderProcess.js` | `GuidingManager` | `pause()`. |
| `FindGuideStarProcess` | `processes/devices/autoguiding/FindGuideStarProcess.js` | `GuidingManager` | `findGuideStar()`. |
| `CoolerManagement` | `processes/devices/coolerProcesses.js` | `CameraManager` | Cooler is part of the camera — add `setCoolerSetpoint` / `getCoolerStatus`. |
| `CloudCheckProcess` | `processes/weather/CloudCheckProcess.js` | `WeatherStationManager` | `getCloudCover()` (or a richer reading). |
| `CloudCheckProcess2` | `processes/weather/CloudCheckProcess2.js` | `WeatherStationManager` | Consolidate the three cloud variants behind one Manager call once converted. |
| `CloudCheckProcess3` | `processes/weather/CloudCheckProcess3.js` | `WeatherStationManager` | As above. |
| `DeviceConnectionVerifier` | `processes/devices/DeviceConnectionVerifier.js` | all `*Manager`s | Multi-device — call `.isConnected()` / `.getStatus()` per Manager instead of TSX scripts. |
| `SingleManualTarget` | `processes/orchestrator/singleManualTarget.js` | `MountManager` | Slew-to-target; MountManager already supports slew. |

### 🚧 In progress — uses a Manager, still TSX-direct in places

These already use a Manager but still import the TSX client or extend
a TSX base class. Each needs the residual TSX paths moved behind the
Manager and the TSX import deleted.

| Process | Source | Manager wired | Notes |
| ------- | ------ | ------------- | ----- |
| `SlewProcess` | `processes/devices/mount/slewProcess.js` | `MountManager` | Drop the TSX import; push remaining direct TSX calls into the Manager. |
| `CLSProcess` | `processes/devices/mount/slewProcess.js` | `MountManager` | Same file as `SlewProcess`. |
| `UnParkProcess` | `processes/devices/mount/slewProcess.js` | `MountManager` | Same file. |
| `ParkProcess` | `processes/devices/mount/slewProcess.js` | `MountManager` | Same file. |
| `SoftParkProcess` | `processes/devices/mount/slewProcess.js` | `MountManager` | Same file. |
| `TrackMountProcess` | `processes/devices/mount/slewProcess.js` | `MountManager` | Same file. |
| `HomeMountProcess` | `processes/devices/mount/slewProcess.js` | `MountManager` | Same file. |
| `MeridianFlipDetector` | `processes/devices/mount/meridianFlip.js` | `MountManager` | Replace TSX polling with `shouldFlip()` / `flip()`. |
| `DitherDetector` | `processes/devices/mount/dither.js` | `SleepManager` only | Add detection to `MountManager` / `GuidingManager`; SleepManager only covers the wait, not the read. |
| `FocusOrchestrationProcess` | `processes/devices/focuser/FocusOrchestrationProcess.js` | `FocuserManager` | Push remaining TSX calls behind the Manager (paired with auto-focus). |
| `CalibrateGuidingProcess` | `processes/devices/autoguiding/CalibrateGuidingProcess.js` | `GuidingManager` | `calibrate()` on the Manager. |
| `StartGuidingProcess` | `processes/devices/autoguiding/StartGuidingProcess.js` | `GuidingManager` | `start(opts)` on the Manager. |
| `CheckGuiderCalibrationProcess` | `processes/devices/autoguiding/CheckGuiderCalibrationProcess.js` | `GuidingManager` | `getCalibrationStatus()`. |
| `CameraOrchestrationProcess` | `orchestrator/CameraOrchestrationProcess.js` | `CameraManager` | Drop the TSX base class; depends on the camera processes above. |
| `CameraOrchestrationProcess2` | `orchestrator/CameraOrchestrationProcess2.js` | `CameraManager` | Drop the TSX import; depends on the camera processes above. |
| `IsDarkCheck` | `processes/orchestrator/isDarkCheck.js` | `SleepManager` only | Add `getSunAltitude()` to `WeatherStationManager` (or a new environment Manager); convert the TSX call. |
| `IsDarkDetector` | `processes/orchestrator/IsDarkDetector.js` | `SleepManager` only | As `IsDarkCheck`. |
| `IsSystemTimeDetector` | `processes/orchestrator/IsSystemTimeDetector.js` | `SleepManager` only | Reading system time needs no TSX at all — just delete the dependency; no new Manager method. |

### ✅ Converted or N/A — no conversion needed

Server-independent today (plain `Process`, no TSX import, Manager-only)
or pure orchestrator control-flow with no hardware to abstract.

| Process | Source | Why |
| ------- | ------ | --- |
| `SleepProcess` | `processes/utils/sleepProcess.js` | Extends `Process`, uses `SleepManager`, no TSX import. The reference for "done". |
| `SelectActiveProfileProcess` | `processes/orchestrator/SelectActiveProfileProcess.js` | Pure profile selection — no hardware. |
| `CheckProfileEligibilityProcess` | `processes/orchestrator/CheckProfileEligibilityProcess.js` | Pure control-flow — no hardware. |
| `SelectNextFrameTaskProcess` | `processes/orchestrator/SelectNextFrameTaskProcess.js` | Pure control-flow — no hardware. |
| `SimulateSelectActiveProfileProcess` | `processes/orchestrator/SimulateSelectActiveProfileProcess.js` | Test/sim helper — no hardware. |
| `MaestroPrepProcess` | `processes/orchestrator/MaestroPrepProcess.js` | Pure prep/control-flow — no hardware. |

> Re-verify the N/A rows when working a nearby row — confirm none have
> crept in a TheSkyX import since the last audit.

## Per-row recipe

For each 📋 / 🚧 row:

1. **Pick the Manager method.** Call it if it exists on `XxxManager`;
   otherwise add it — one method per device operation, returning
   `{ success, message, result }`.
2. **Make the adapter side work for every backend** the Manager
   supports: TheSkyX adapter (run the relevant TSX script),
   Simulator adapter (mutate adapter state), and the INDIGO adapter
   (REST call) where applicable.
3. **Rewrite the process body** as a thin wrapper: read the Manager
   from context, `await manager.<op>(...)`, return its result.
4. **Delete the TSX coupling**: remove the TheSkyX import, change the
   base class to plain `Process`, drop any script-process overrides.
5. **Add/extend the unit test** that mocks the Manager and asserts the
   process delegates to it.

## Verification

For each row marked converted:

1. Grep the process file for TheSkyX imports and TSX base classes —
   must return zero results.
2. Run the per-process unit test.
3. End-to-end smoke against the **Simulator** backend with no TheSkyX
   running — drive the orchestrator stage and confirm completion.
4. End-to-end smoke against the **TheSkyX** backend — behaviour must
   be unchanged.

When every 📋 and 🚧 row is at zero TSX imports and passes steps 2–4,
the conversion is complete.

_Last audited 2026-05-14._
