# ADR-002: Rendering API

Date: 2019-09-03
Status: Accepted
Author: Graphics Lead

## Context
The first title targeted PC and we needed pixels on screen quickly. We had to pick a
graphics API and decide how draw calls get issued from the engine.

## Options Considered

1. **OpenGL directly.** Runs everywhere we ship today, every engineer knows it, nothing to build first.
2. **A render abstraction layer now.** An interface over GL/DX/Vulkan up front; more code before any pixels, slower MVP.
3. **A third-party render library (e.g. bgfx).** Multiple backends out of the box, but a dependency to learn and integrate.

## Decision
Use OpenGL ES. Issue draw calls through one thin `IVideoDriver` interface, but write only the
single backend GLES needs (`VideoDriverGLES20`) and shape the interface around GLES2 concepts.
No general multi-API abstraction for the MVP.

## Rationale
- Fastest path to a shippable build for a single platform.
- A thin, GLES-shaped driver is just enough to ship; no multi-backend abstraction to design up front.
- One graphics API was all the launch platform needed.

## Consequences

**Positive**
- Shipped the first title on time with no rendering middleware.
- One backend is easy to trace when a single platform misbehaves.

**Negative**
- The `IVideoDriver` interface is shaped around GLES2 (string-id shader uniforms, GLES blend and
  state enums), and `STDRenderer` plus the shader programs assume that backend, so GLES specifics
  leak through the seam.
- Only `VideoDriverGLES20` was ever written, so the interface has never been proven against a second backend.

**To watch**
- Any second graphics API (DX12, Vulkan, Metal) means reshaping `IVideoDriver` into a real
  multi-backend seam, not just adding a class behind today's GLES-shaped interface. Do that
  *before* the console/mobile port starts, not during it.
