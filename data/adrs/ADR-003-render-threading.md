# ADR-003: Render Threading

Date: 2019-08-20
Status: Accepted
Author: Engine Lead

## Context
We needed a render loop for the first title. The team was small and the launch hardware
had headroom, so we optimized for simplicity over parallelism.

## Decision
A single update/render thread. The game updates the `Actor` tree, then `STDRenderer` submits
draw calls immediately through `IVideoDriver` on the same thread, every frame.

## Rationale
- The simplest model there is: no synchronization, no race conditions.
- One thread was fast enough on the launch hardware.
- Easy to reason about: update, then draw, in order, on one core.

## Consequences

**Positive**
- No threading bugs in the renderer for the whole first title.
- Frame flow is trivial to debug; it is all sequential.

**Negative**
- The renderer uses one core while the others sit idle; the frame budget is tight on weaker hardware.
- Immediate GL submission can't be recorded and replayed, so there is no command buffer to hand to a worker.

**To watch**
- Do NOT move rendering onto worker threads before a command-buffer / render abstraction exists.
  The GL context is thread-affine and immediate calls can't be parallelized safely, so threading first will race.
- Do NOT refactor the entity model and the renderer in the same change. Two large migrations at once
  leaves the engine unshippable for months. Sequence them.
