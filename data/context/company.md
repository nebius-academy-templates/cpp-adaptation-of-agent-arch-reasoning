# Northwind Studios: Company Context

## What We Do
Northwind Studios builds stylized action games on our own C++ engine, kept in-house
since 2019. We shipped one title, run it as a live game with monthly content, and have
a second title greenlit on the same engine.
Think of us as a small studio that owns its tech instead of licensing Unity or Unreal.
Founded in 2019, PC-first, now being pushed toward console and mobile.

## Current Numbers
- 1 shipped title, ~180,000 owners, live for 14 months
- ~9,000 daily active players, peaks near 22,000 on patch days
- "Very Positive" store rating (88%)
- Monthly content patch cadence; the live title never stops shipping
- Frame budget on target hardware: 60 fps (16.6 ms/frame), and headroom is thin

## Roadmap
- Second title is greenlit and shares the engine
- Publisher wants the live title on console and mobile within ~12 months
- The engine team cannot pause live-ops to do it

## Current Technology Stack
- Engine: in-house C++17 engine ("Oxygine"), maintained since 2019
- Entities: OOP inheritance on the engine's `Actor` base class, deep subclass trees (`Actor` → `VStyleActor` → `Sprite`), with our `Player` / `Enemy` / `Pickup` types layered on top
- Rendering: OpenGL ES only, behind a single-backend `IVideoDriver` interface (`VideoDriverGLES20`); `STDRenderer` submits draw calls immediate-mode
- Threading: a single update/render thread, immediate-mode draw submission
- Assets: loaded synchronously at scene start
- Build: CMake, full rebuild ~22 min, incremental link ~90 s

## Engineering Team
- 3 engine engineers (the core C++ systems)
- 4 gameplay engineers
- 2 graphics / tools engineers
- 1 producer
- No dedicated console/platform specialist yet

## Hard Constraints
- No feature freeze: the live title keeps shipping monthly content through any refactor.
- Incremental migration only: a multi-month big-bang rewrite is off the table.
- First console port is due in ~12 months, a fixed publisher commitment.

## Known Pain Points (Business Impact)

1. **The entity hierarchy is slowing everyone down.** Adding a new kind of object means
   editing a shared base class, and a few diamond-inheritance knots have already appeared.
   New behaviour lands slower every quarter.

2. **Cache misses in the update loop.** Object data is scattered across the class tree,
   so iterating thousands of entities thrashes the cache. Big scenes drop frames.

3. **Locked to one graphics API.** There is an `IVideoDriver` interface, but the only backend
   ever written is `VideoDriverGLES20`, and both the interface and `STDRenderer` assume GLES2.
   Finishing that seam is the precondition for any second API or new platform.

4. **The renderer uses one core.** Immediate-mode submission on the main thread leaves
   the other cores idle. The frame budget is tight on weaker target hardware.

5. **Designers wait on engineers.** Behaviour changes need a C++ edit and a recompile,
   and a full rebuild is ~22 min. Iteration is painful.

6. **The port can't be scoped.** Because GL is everywhere, nobody can give the publisher
   an honest estimate for the console port. That is now a business risk.

## Timeline
- Engine modernization decision: now, before the second title's vertical slice.
- Second title vertical slice: ~6 months.
- First console port: ~12 months, hard publisher deadline.
