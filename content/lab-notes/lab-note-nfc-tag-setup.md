+++
date = '2026-05-27T19:24:05-03:00'
title = 'Lab Note: NFC Tag Setup for Filament and Location Tracking'
tags = ["NFC", "OpenPrintTag", "Filament Tracking", "The Moment", "Prusa Core One"]

[params]
  status = "Confirmed Working"
+++

## Overview

This note documents the process for initializing and writing NFC tags for use with The Moment and Prusa Core One filament tracking. It covers the correct tag types, apps required, and the initialization workaround needed for blank NFC-V stickers on iOS.

---

## Tag Types

Two tag types have been confirmed working:

- **NFC-V (ISO 15693 / ICODE SLIX2)** — Required for Prusa Core One native NFC reader. Longer read range. Use for filament spools on the Core One.
- **NTAG215** — Standard NFC tag. Confirmed working for app-based reading and writing. Suitable for location tags and non-Prusa filament use cases.

### Use Cases

| Use Case | Tag Type |
|---|---|
| Prusa Core One filament spool | NFC-V (ICODE SLIX2) |
| Location tags (printer stations, storage) | NTAG215 or NFC-V |
| App-only filament tracking (no printer reader) | NTAG215 |

---

## Apps Required

### NFC.cool

- **Platform:** iOS and Android
- **Cost:** Free for OpenPrintTag feature (version 6.3.0+)
- **Function:** Read and write OpenPrintTag records to NFC tags
- **Path in app:** NFC Apps → OpenPrintTag
- **Notes:** Cannot write to blank unformatted NFC-V tags. Tags must be initialized first (see below).

### Serialio NFC Read Write

- **Platform:** iOS only
- **Cost:** Free with in-app purchases
- **Function:** Can write NDEF records to blank, unformatted ISO 15693 (NFC-V) tags — something most iOS NFC apps cannot do
- **Use:** One-time initialization of blank NFC-V tags only

### NFC Tools

- **Platform:** iOS and Android
- **Cost:** Free (paid Pro version available)
- **Function:** Read and write NDEF tags after initialization. Confirmed working once a tag has been initialized.

---

## The Problem: Blank NFC-V Tags on iOS

Factory-blank NFC-V tags ship without an NDEF structure. iOS NFC apps (including NFC.cool and NFC Tools) cannot detect or write to them in this state. The tag appears either invisible or "read only."

This is an iOS limitation with unformatted ISO 15693 tags.

---

## Fix: Initialize with Serialio First

This is a one-time step per tag.

1. Open **Serialio NFC Read Write**
2. Tap **Write NDEF**
3. Select record type **Text**
4. Enter any value — `hello` or `hello world` is sufficient
5. Hold the blank NFC-V sticker flat against the top edge of your iPhone
6. Hold still for 2–3 seconds until the write confirms

The tag now has a valid NDEF structure.

---

## After Initialization

Once initialized, the tag works normally with both NFC.cool and NFC Tools.

**To write OpenPrintTag data:**
1. Open **NFC.cool**
2. Go to NFC Apps → OpenPrintTag
3. Fill in filament fields (material, brand, color, weight, etc.)
4. Tap NFC button — this overwrites the `hello` text record with OpenPrintTag data

**To read or update later:**
- NFC.cool, NFC Tools, or the Prusa app can all read the tag
- NFC.cool can update individual fields (e.g. remaining weight after a print)

---

## OpenPrintTag Fields (Reference)

| Field | Notes |
|---|---|
| Material type | PLA, PETG, ASA, TPU, etc. |
| Brand | e.g. Polymaker, Prusament |
| Color | Supports up to 6 colors per spool |
| Weight remaining | Update after each print |
| Batch / lot | Optional |
| Custom notes | Free text field |

---

## Integration Notes for The Moment

- NFC tag UID can serve as the spool identifier linking the physical tag to a Spoolman spool record
- Location tags (NTAG215 or NFC-V) can encode a plain text or URI record identifying the printer station or storage slot
- Tag scan events can be logged as print start / end markers in The Moment's print history
- Both NFC-V and NTAG215 are readable by iPhone without any additional hardware

---

## Known Issues and Tips

- If NFC.cool still says "read only" after Serialio initialization, try using NFC Tools → Other → Erase Tag, then retry with NFC.cool
- Hold the tag flat and still against the very top edge of the iPhone during any write operation
- NFC-V tags have a longer read range than NTAG tags — better for printer-mounted spool detection
- NTAG215 tags are cheaper and more widely available for location and general-purpose use

---

## References

- [OpenPrintTag standard](https://openprinttag.com)
- [NFC.cool OpenPrintTag support](https://www.nfc.cool/post/openprinttag-support-nfc-cool-ios)
- [Prusa Forum: How to write an OpenPrintTag](https://forum.prusa3d.com/forum/openprinttag/how-to-write-an-openprinttag/)
- [Serialio NFC Read Write iOS app](https://serialio.com/serialio-news/nfc-read-write-ios-app-ndef-raw-data-all-supported-nfc-chip-types/)
