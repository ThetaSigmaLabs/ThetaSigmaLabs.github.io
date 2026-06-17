+++
title = 'tsx_cmd'
description = 'A Meteor app for planning and automating multi-target astrophotography sessions across TheSkyX, INDIGO, ASCOM, and a built-in simulator.'
draft = true
+++

`tsx_cmd` is a Meteor app for planning and running a night at the
telescope. You stage one or more targets, lay them out on a per-night
gantt with target plots, and the app drives the whole observatory —
dome, mount, camera, filter wheel, dew heater, focuser, rotator,
guider, and more — through the session as a sequence of process
stages.

The equipment layer is driver-agnostic: the same plan runs whether the
hardware sits behind TheSkyX, INDIGO, ASCOM, or the built-in
simulator.

## Sections

* [Docs](docs/) — run the app, configure equipment, plan a night, run
  a session.
* [Architecture](architecture/) — Meteor stack, planner + scheduler,
  driver abstraction, future SDK.
* [Drivers](drivers/) — per-backend dev logs and checklists.

## Compatibility matrix

Equipment × backend. Legend: ✅ working · 🚧 in progress · 📋 planned ·
❌ not supported · — n/a

| Device class    | TheSkyX | INDIGO | ASCOM | Simulator |
| --------------- | :-----: | :----: | :---: | :-------: |
| Dome            |   📋    |   📋   |  📋   |    📋     |
| Mount           |   📋    |   📋   |  📋   |    📋     |
| Camera          |   📋    |   📋   |  📋   |    📋     |
| Filter wheel    |   📋    |   📋   |  📋   |    📋     |
| Focuser         |   📋    |   📋   |  📋   |    📋     |
| Rotator         |   📋    |   📋   |  📋   |    📋     |
| Dew heater      |   📋    |   📋   |  📋   |    📋     |
| Guider          |   📋    |   📋   |  📋   |    📋     |
| Plate solve     |   📋    |   📋   |  📋   |    📋     |

Per-driver detail, capability checklists, and gotchas live on the
[driver pages](drivers/).

## Planner & stages

A night is modelled as a list of targets, each broken into ordered
process stages — e.g. _slew → settle → plate solve → focus → rotate →
guide → capture sequence → meridian flip → re-acquire_. The gantt view
shows the planned timeline; the live view shows where each target
actually is.

_TODO: link to the stage catalogue once it lands._

## Status at a glance

* **Done:** _fill in as features ship._
* **In progress:** _current focus._
* **Planned next:** _short list of upcoming work._

## Roadmap

1. Lock the device-class command surface against the simulator driver.
2. Bring TheSkyX driver to parity (it's the reference backend).
3. Add INDIGO and ASCOM drivers behind the same interface.
4. Extract the driver layer into a standalone SDK so other apps (and
   third parties) can target the same abstraction.
