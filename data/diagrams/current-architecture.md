# Current Architecture (Oxygine engine)

Detailed view of the current engine. The three highlighted zones are what the workshop's three features would change; everything else stays as is.

- 🟢 **green** = entity model the ECS migration replaces (Sofia / ADR-001)
- 🔵 **blue** = render seam the RHI abstraction reshapes (Tariq / ADR-002)
- 🟠 **orange** = single-thread + immediate-mode submission the multithreading targets (Yuki / ADR-003)

```mermaid
graph TD
    subgraph Legend["Workshop zones"]
        direction TB
        L1["ECS migration (Sofia / ADR-001)"]
        L2["Render abstraction, RHI (Tariq / ADR-002)"]
        L3["Multithreaded renderer (Yuki / ADR-003)"]
        L1 ~~~ L2 ~~~ L3
    end

    Assets["Assets load up front<br>(synchronous, at scene start)"] --> Tree

    subgraph Loop["Game loop, one thread does everything"]
        direction TB
        Update["Update: walk the Actor tree each frame<br>(Stage.update calls update() on every Actor)"]
        Render["Render: walk the same tree and draw<br>(Stage.render calls render() on every Actor)"]
        Update --> Render
    end

    subgraph Entities["Entities: one deep inheritance tree"]
        Tree["Everything is an Actor<br>Object → Actor → VStyleActor → Sprite<br>UI: Button, TextField · game: Player, Enemy, Pickup"]
    end

    subgraph RenderStack["Drawing path (immediate-mode)"]
        direction TB
        Delegate["RenderDelegate (STDRenderDelegate)"]
        STD["STDRenderer<br>submits draw calls right away"]
        Driver["IVideoDriver<br>the render interface, shaped for GLES2"]
        Backend["VideoDriverGLES20<br>the only real backend (plus a null stub)"]
        Delegate --> STD
        STD --> Driver
        Driver --> Backend
    end

    Update --> Tree
    Render --> Tree
    Tree -->|"doRender()"| Delegate
    Backend --> GPU[("OpenGL ES 2.0")]

    Legend ~~~ Assets

    classDef ecs fill:#e7f5e0,stroke:#2e7d32,stroke-width:3px;
    classDef rhi fill:#e3f0fb,stroke:#1565c0,stroke-width:3px;
    classDef thread fill:#fdeccd,stroke:#e65100,stroke-width:3px;

    class Tree,Update,L1 ecs;
    class Driver,Backend,L2 rhi;
    class STD,L3 thread;
    style Loop stroke:#e65100,stroke-width:2px,stroke-dasharray:5 3;
```
