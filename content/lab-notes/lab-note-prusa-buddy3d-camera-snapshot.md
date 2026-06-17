+++
date = '2026-06-03T11:00:00-03:00'
title = 'Lab Note: Prusa Buddy3D Camera — Local Snapshot Feasibility'
Status = "In Progress"
tags = ["PrusaLink", "Buddy3D Camera", "The Moment", "Prusa Core One", "Camera API"]
+++

## Overview

Investigation into whether the Prusa Buddy3D Camera can be used to capture print-end snapshots locally via PrusaLink for The Moment print history. Triggered by implementing a camera snapshot feature that correctly handled the API response but produced no images on a real print.

<!--more-->

---

## Setup

| Component | Detail |
| --- | --- |
| Printer | Prusa Core One L |
| Printer firmware | 6.5.3+12780 |
| PrusaLink API version | 2.0.0 |
| Camera | Buddy3D Camera (Guardian PR1), firmware 3.1.3 |
| Camera connectivity | Wi-Fi 2.4 GHz, own IP address on LAN |

---

## What Was Implemented

The Moment's camera snapshot feature queries `GET /api/v1/cameras` on the printer, then calls `GET /api/v1/cameras/{id}/snap` to capture a JPEG. This endpoint exists in the [PrusaLink OpenAPI spec](https://github.com/prusa3d/Prusa-Link-Web/blob/master/spec/openapi.yaml) and was confirmed in the spec for the Raspberry Pi-based PrusaLink (used on MK3/MK4/MK3.9 printers).

---

## Findings

### `/api/v1/cameras` returns 404 on Core One L

The endpoint is not implemented in the Core One L's Buddy board firmware. The Buddy board exposes a PrusaLink-compatible API, but camera management is not part of it because the Buddy3D Camera is a **separate network device**, not a USB camera attached to the printer's board.

```text
GET http://<printer-ip>/api/v1/cameras
→ 404 Not Found
```

This is not a bug in The Moment. The code correctly treats 404 as "no camera" and silently no-ops. No crash, no error, no snapshot.

### Buddy3D Camera architecture

The Buddy3D Camera (Guardian PR1) is a standalone Wi-Fi camera with its own IP address. It is not proxied through the printer's PrusaLink API.

| Access method | Available | Notes |
| --- | --- | --- |
| `GET /api/v1/cameras` on printer | ❌ No | Printer firmware does not implement it |
| HTTP snapshot endpoint on camera | ❌ No | No HTTP API on the camera itself |
| RTSP stream | ✅ Yes | `rtsp://<camera-ip>/live`, port 554, LAN only |
| PrusaConnect cloud snapshots | ✅ Yes | Automatic, every 10–90 s, requires cloud auth |
| ONVIF / REST API on camera | ❌ None native | go2rtc or similar proxy required |

### RTSP stream

The camera exposes an unencrypted RTSP stream on the local network when enabled via the Prusa app:

```text
rtsp://<camera-ip>/live
rtsp://admin:admin@<camera-ip>:554/live   (alternate format seen in the wild)
```

A still frame can be captured from this stream using ffmpeg or a Go RTSP library, but requires knowing the camera's separate IP address.

---

## Options Going Forward

### Option A — Add configurable snapshot URL per printer *(recommended)*

Add an optional `CameraSnapshotURL` field to the printer config. If set, The Moment fetches from that URL instead of (or in addition to) trying `/api/v1/cameras`. Supports:

- Any HTTP/HTTPS endpoint returning a JPEG
- RTSP via ffmpeg exec or Go RTSP library

Users with the Buddy3D Camera configure the RTSP URL. The existing PrusaLink camera API fallback remains active for Pi-based printers.

### Option B — Wait for firmware update

Prusa may add `/api/v1/cameras` support to Buddy board firmware in a future version. If/when they do, The Moment's existing code activates automatically. No action required today.

---

## References

- [PrusaLink OpenAPI spec — cameras](https://github.com/prusa3d/Prusa-Link-Web/blob/master/spec/openapi.yaml)
- [Buddy3D Camera Knowledge Base](https://help.prusa3d.com/article/buddy3d-camera_821264)
- [Buddy3D Camera firmware 3.1.0 forum thread](https://forum.prusa3d.com/forum/buddy3d/buddy3d-camera-firmware-3-1-0/)
- [Buddy3D Camera REST API discussion](https://forum.prusa3d.com/forum/buddy3d/rest-api-for-buddy-core-one-version-available/)
- [Prusa Buddy3D Cameras and UniFi Protect](https://andrewmzhang.com/blog/2026/prusa-cameras/)
