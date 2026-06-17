# Setting Up a Single Extruder as a Multi-Extruder in PrusaSlicer

**Printer:** Prusa Core One L
**Slicer:** PrusaSlicer
**Use case:** Flush inlay lettering (black body with white letters) using a manual filament swap
**Status:** Verified working

---

## Purpose

PrusaSlicer has two workflows for two-color prints on a single-nozzle printer:

1. **Color Change** — uses the layer slider and the `+` icon. Inserts `M600` directly. Best when every layer is one color only.
2. **Single Extruder Multi Material (multi-extruder mode)** — assigns parts of the model to "extruder 1" and "extruder 2". This is needed when a feature must be split out by object or modifier, not just by Z height.

This note covers **Option 2**, because the lettering inlay was set up as a separate object/modifier and assigned to a second extruder.

Out of the box, Option 2 does not work on a single-nozzle printer:

- The slicer outputs `T0` / `T1` tool change commands, which the Core One firmware ignores
- A purge tower is generated and printed, wasting time and filament
- The printer never pauses to ask for a filament swap

The steps below fix all three problems.

---

## Model Design Requirements

- **First layer height:** 0.2 mm
- **Insert depth:** 2 layers (0.4 mm total at 0.2 mm layer height)
- The inlay region must be modeled flush with the surrounding surface so the white filament fills the recess

---

## PrusaSlicer Setup Steps

Do these in order. Each one matters.

### 1. Configure the printer for two extruders

- Open **Printer Settings** tab
- Go to **General** in the left sidebar
- Set **Extruders** = `2`
- Check **Single Extruder Multi Material**

### 2. Add M600 to the Tool change G-code

- Still in **Printer Settings**, go to **Custom G-code** in the left sidebar
- In the **Tool change G-code** field, type:

```
M600 ; Filament change
```

Important: this must go in **Tool change G-code**, not in **Color Change G-code**. The Color Change field only fires when an "Add color change" marker is placed via the layer slider. The Tool change field is what fires when the slicer emits a `T0` or `T1`.

### 3. Fix the Single Extruder MM setup numbers

- In **Printer Settings**, go to **Single extruder MM setup**
- Set:
  - **Cooling tube position** = `10`
  - **Filament parking position** = `0`
  - **Extra loading distance** = `0`

The default values in this section are tuned for a real MMU and cause a large blob during manual swaps. These numbers stop that.

### 4. Disable the wipe tower

- Open **Print Settings** tab
- Go to **Multiple Extruders** in the left sidebar
- Under **Wipe tower**, uncheck **Enable**
- Also uncheck **Prime all printing extruders** if it is ticked

### 5. Save the profile

- Click the small **disk icon** next to the printer profile name at the top
- Save as a custom name (for example `Core One L — manual two-color`)
- Prusa system profiles cannot be overwritten, so this must be a custom profile

- Repeat for the print profile if any Print Settings were changed

### 6. Force a re-slice

- Click **(Re)Slice Now** in the bottom-right of the Plater
- Do not rely on the cached slice — settings changes are not always reflected until the slice is forced

### 7. Assign filaments

- In the Plater, the right-hand filament dropdowns now show two slots
- Set both to the filament type you are printing (for example PLA + PLA)
- Set the color of each slot to match the real filament you will load
- Verify your object or modifier is assigned to extruder 2 for the inlay region

---

## Verifying the G-code (Debug Step)

The first time the project is set up, switch to text G-code so the file can be opened in a text editor.

### Switch to text G-code

Two options:

- **Per profile:** Printer Settings → General → uncheck **Supports binary G-code**, then save the profile
- **Globally:** Configuration → Preferences → Other → uncheck **Use binary G-code when the printer supports it**

Re-slice. The export will now be `.gcode` instead of `.bgcode`.

### What to search for in the file

| Search term | What it means | Expected count for a single inlay |
|---|---|---|
| `M600` | Filament change pause | 1 or 2 (one swap to white, optional swap back) |
| `T0`, `T1` | Tool change commands | Same count as M600 — each should sit right before an M600 |
| `WIPE_TOWER` or `CP TOOLCHANGE` | Wipe tower comments | Should be **zero** if the wipe tower is properly disabled |

### Sanity check in the preview

- In PrusaSlicer after slicing, the **layer slider** on the right shows pause icons at each `M600`
- Confirm the icons sit at the Z heights where the filament should swap

### Switch back to binary G-code

Once the file is verified, re-enable binary G-code:

- Per profile: re-check **Supports binary G-code**, save profile
- Globally: re-check **Use binary G-code when the printer supports it**

Binary G-code is smaller and uploads faster to the printer.

---

## Known Quirks on the Core One

- **Door interlock during M600:** The print head will not move while the door is open. The standard "hold the purge string and let the nozzle pull it off" trick from MK3/MK4 days does not work. Close the door before resuming. There will be a small ooze blob on the next move.
- **Possible "bonus" M600 pauses:** Some users on GitHub report extra filament-change prompts at unexpected points. Verify with the G-code search before a long print.
- **Open feature request (March 2026):** Prusa is considering a proper single-extruder purge tower for M600 prints. May simplify this workflow in a future PrusaSlicer release.

---

## When to Use Color Change Instead (Option 1)

If the inlay design has **every layer as one color only** (no layer has both black and white), the simpler workflow is:

1. Slice the model with one filament
2. Drag the layer slider to the layer where white should start
3. Click the orange `+` icon → **Add color change**
4. Repeat at the layer where black should resume

No multi-extruder setup, no wipe tower, no Tool change G-code edit. Use this when the geometry allows it.

---

## Sources

- [Prusa Knowledge Base — Color change](https://help.prusa3d.com/article/color-change_1687)
- [Prusa Knowledge Base — Binary G-code](https://help.prusa3d.com/article/binary-g-code_646763)
- [Prusa forum — Having trouble with single extruder multicolor printing (Core One)](https://forum.prusa3d.com/forum/prusa-core-one-how-do-i-print-this-printing-help/having-trouble-with-single-extruder-multicolor-printing/)
- [Prusa forum — Filament Towers (explains T0/T1 vs M600)](https://forum.prusa3d.com/forum/prusaslicer/filament-towers/)
- [GitHub Issue #14694 — Single Extruder Multi Material M600 ignored on Core One](https://github.com/prusa3d/prusaslicer/issues/14694)
- [GitHub Issue #15278 — Feature request: purge tower for single-extruder M600](https://github.com/prusa3d/PrusaSlicer/issues/15278)
- [Printables — Prusa Tool Box (Core One L) with multi-color manual swap instructions](https://www.printables.com/model/545197-prusa-tool-box-core-one-core-one-l-mk4s-mk4-xl-mk3)
