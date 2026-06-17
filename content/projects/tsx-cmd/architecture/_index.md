+++
title = 'Architecture'
description = 'Meteor stack, planner, stage engine, driver abstraction, and SDK roadmap for tsx_cmd.'
draft = true
+++

How `tsx_cmd` is put together: a Meteor app on top of a driver layer
that abstracts the observatory hardware.

## Goals

* One plan, multiple backends — the same night plan runs whether the
  hardware is behind TheSkyX, INDIGO, ASCOM, or the simulator.
* The night is a first-class object: targets, stages, and the gantt
  are the model, not bolt-ons.
* Drivers are pluggable and testable in isolation.
* The simulator is a first-class driver — the reference for what
  "correct" behaviour looks like and the substrate for rehearsing a
  plan before sky time.

## Layers

```text
┌─────────────────────────────────────────┐
│  Client (Meteor / React)                │  plan editor, gantt,
│                                         │  target plots, live view
├─────────────────────────────────────────┤
│  Meteor server                          │  methods, pub/sub, auth
├─────────────────────────────────────────┤
│  Planner & stage engine                 │  targets, stages, gantt,
│                                         │  scheduling, recovery
├─────────────────────────────────────────┤
│  Driver interface (future SDK)          │  Dome · Mount · Camera ·
│                                         │  FilterWheel · Focuser ·
│                                         │  Rotator · DewHeater ·
│                                         │  Guider · Solver
├─────────────────────────────────────────┤
│  Driver implementations                 │  TheSkyX · INDIGO · ASCOM ·
│                                         │  Simulator
└─────────────────────────────────────────┘
```

## Planner & stage engine

_TODO: data model for plans, targets, and stages; how the gantt is
computed; how the stage engine sequences a target (slew, settle, plate
solve, focus, rotate, guide, capture, meridian flip, re-acquire);
pause / resume / skip / recover semantics._

## Driver interface

_TODO: define the interfaces for each device class. Keep the surface
minimum-viable and grow it deliberately — every method has to be
implementable on every backend or it doesn't belong on the
interface._

[Process conversion](process-conversion/) tracks the migration of every
registered process onto the driver-agnostic Manager layer.

## SDK plan

* Phase 1 — interface stabilises inside the `tsx_cmd` repo, exercised
  by the app and simulator.
* Phase 2 — extract into its own package so third-party tools can
  consume it independently of Meteor.
* Phase 3 — publish reference drivers + a conformance test suite that
  any new driver must pass.

## Open questions

_TODO: capture design questions as they come up — async vs sync
command model, error taxonomy, capability discovery, units (degrees vs
radians, JNow vs J2000), connection lifecycle, how the stage engine
maps onto Meteor methods vs. background jobs, persistence model for
in-flight sessions, etc._
