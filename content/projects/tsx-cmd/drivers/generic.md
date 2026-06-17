+++
title = 'Generic / cross-driver notes'
description = 'Behaviour and conventions that apply across all tsx_cmd drivers.'
draft = true
+++

Things that aren't owned by a single driver — they shape the shared
interface or show up in every backend.

## Capability discovery

_TODO: how a driver advertises which features it supports, and how the
CLI degrades gracefully when something isn't available._

## Error taxonomy

_TODO: shared error categories (connection, protocol, device, command,
timeout) and how each driver maps its native errors into them._

## Units & coordinates

_TODO: degrees vs radians, JNow vs J2000 vs ICRS, time/UTC handling,
where conversions happen._

## Threading & async model

_TODO: blocking vs non-blocking commands, slew completion, long-running
operations, cancellation._

## Conformance tests

_TODO: the test suite every driver runs against. The simulator is the
canonical pass._
