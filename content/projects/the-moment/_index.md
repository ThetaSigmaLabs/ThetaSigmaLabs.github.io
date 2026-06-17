+++
title = 'The Moment'
description = 'Automatic filament tracking, print history, and cost accounting — connected to Spoolman in real time.'
+++

{{< project-logo src="img/projects/the-moment-logo.png" alt="The Moment" >}}

Sit between your printers and [Spoolman](https://github.com/Donkie/Spoolman). When a print finishes, The Moment automatically deducts filament, logs the run with duration and cost, records which spool was on which toolhead, and stores a thumbnail if your slicer embedded one. You never touch Spoolman manually again.

Every gram used. Every minute spent. Every dollar it cost. Logged automatically, the moment a print finishes.

## Deploy

No git clone. No build step.

```bash
curl -O https://raw.githubusercontent.com/ThetaSigmaLabs/the-moment/main/docker-compose.yml
curl -O https://raw.githubusercontent.com/ThetaSigmaLabs/the-moment/main/.env.example && cp .env.example .env
docker compose up -d
```

Open `http://<your-server-ip>:5000` → Settings → Add Printer.

Spoolman is bundled in the compose file — both services start with those three commands. Add your filament types and spools in Spoolman before your first print; The Moment reads from Spoolman, it doesn't create records there.

## Features

### Filament & Spool Tracking

- Automatic weight deduction in Spoolman the moment a print finishes
- Multi-toolhead support — each toolhead tracks its own spool independently
- Filament-change tracking — spool swaps mid-print recorded as separate entries
- Location tracking — spools carry their location (printer toolhead or storage shelf); bidirectional sync with Spoolman

### Print History

- Every print logged: source printer, spool used, grams consumed, duration, cost breakdown
- Thumbnail extraction from PrusaSlicer and OrcaSlicer G-code
- Freeform notes on any history entry; searchable and deletable

### Cost Accounting

- Filament cost prorated from Spoolman's per-spool price
- Electricity: configurable kWh rate × print wattage × duration
- Preheat charge, high-temp surcharge (auto-applied for ABS, ASA, PA, PC)
- Maintenance, depreciation, and profit margin rates
- Per-printer overrides for wattage, preheat spec, and depreciation
- Configurable currency (USD, CAD, EUR, GBP, etc.)
- Quick calculator: test cost settings with arbitrary weight and time, no hardware needed

### NFC Tags

Tap a spool with your phone. Tap the printer slot. Done — spool assigned in The Moment and Spoolman simultaneously.

- ICODE SLIX2 tags programmed via NFC Tools Pro
- Dual-record NDEF: [OpenPrintTag](https://specs.openprinttag.org) CBOR + URL fallback
- One location tag per printer toolhead; tap spool then location within 5 minutes
- QR codes generated alongside every tag for NFC-free environments

## Supported Printers

| Printer | Interface | Multi-toolhead | Status |
| --- | --- | --- | --- |
| PrusaLink (CORE One, XL, MK4, Mini+) | PrusaLink API | Yes | Supported |
| OctoPrint (Ender, CR-10, Voron, etc.) | OctoPrint plugin | Single-head | Supported |
| Bambu X1C, P1S, A1, A1 Mini | MQTT over LAN | AMS slots → toolheads | Beta |
| INDX 8-head | TBD | 8 toolheads | Planned |

## Links

[GitHub →](https://github.com/ThetaSigmaLabs/the-moment)

Forked from [FilaBridge](https://github.com/needo37/filabridge) by needo37 — the archived original that pioneered the Spoolman bridge pattern. GPL-3.0.
