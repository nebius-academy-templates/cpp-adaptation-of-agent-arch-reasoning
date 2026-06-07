# Scenario 3 Adaptation Summary: Engine Internals (Render Abstraction + ECS)
#workshop/scenario-3 #adaptation

This is the C++ adaptation of the "Building an Architecture Reasoning Agent" workshop, scenario 3 from the *Adaptation variants* page: a game studio whose in-house engine uses OOP inheritance for entities (the `Actor` tree) and a single render backend (GLES2 only, behind a half-built `IVideoDriver` seam), both now blocking new platforms. It re-skins every authored artifact, swaps the reference codebase to a real C++ engine (Oxygine, as a git submodule), fixes a known data bug, and recomputes the per-block timings. Below is what changed and what a facilitator needs to do differently.

## Base-swap inventory

| Artifact | Was (Conduit) | Now (Scenario 3) |
|----------|---------------|------------------|
| `data/context/company.md` | Conduit publishing platform, 14k users | Northwind Studios, in-house C++ engine, 1 live title |
| `ADR-001` | Database Selection (solid) | **Entity Model**, OOP `Actor` inheritance (**thin**) |
| `ADR-002` | API Design Style (solid) | **Rendering API**, OpenGL for MVP (**solid / hinge**) |
| `ADR-003` | Deployment Architecture (outdated) | **Render Threading**, single immediate-mode thread (**outdated**) |
| `current-architecture.mmd` | Browser → VPS → Postgres | Game loop → `Actor` tree → STDRenderer / IVideoDriver, one thread |
| `team-lead/` | Alex Chen, real-time collab editing | **Sofia Lindqvist**, ECS migration |
| `team-lead-2/` | Marcus Webb, AI personalized feed | **Tariq Hassan**, render-backend abstraction |
| `team-lead-3/` | Priya Sharma, premium paywall | **Yuki Sato**, multithreaded renderer + job system |
| reference codebase | `conduit-realworld-example-app/` (Node, ~3.4k LOC) | `engine/` git submodule, Oxygine (C++, MIT), pinned at `1e9f923` |
| root template constraint | "7 people / ≤€5k" (the bug) | 9 engineers + no-freeze / incremental / 12-month port |

**Unchanged (carried over untouched):** the four `skills/*.py`, `.claude/settings.json`, the `output/` scaffolding, the `rfc_writer.md.template` scaffolding, the Mermaid MCP section, and the Step 0-4 flow. These are the parts the variants page says never change.

## Workshop corrections to follow the new scenario

### 1. The persona model changed shape

The original has three personas each requesting a *different product feature*. Scenario 3 keeps three personas but they request three *different engine-modernization features over one shared codebase, ADR trio, and diagram*. Each is anchored to one ADR:

| Persona (dir) | Feature requested | Anchored ADR |
|---------------|-------------------|--------------|
| Sofia Lindqvist (`team-lead/`) | Migrate entities to a data-oriented ECS | ADR-001 (thin: OOP inheritance) |
| Tariq Hassan (`team-lead-2/`) | Render-backend abstraction (RHI) for ports | ADR-002 (hinge: scattered GL) |
| Yuki Sato (`team-lead-3/`) | Multithreaded renderer + job system | ADR-003 (outdated: single thread) |

Students still pick one lead and interview them, exactly as before. The leads still answer in delivery language and push back on scope.

### 2. Step 0 now clones a real codebase

The reference code is a git submodule. Before running `claude`, run:

```bash
git submodule update --init     # checks out Oxygine at the pinned commit 1e9f923 into engine/
```

The clone is ~173 MB. **Treat it like the Mermaid install: do it on the break before the session, not live.** A cold clone over conference wifi will eat block time. The "real code" the agent reads is `engine/oxygine/src/`, the `Actor`/`Sprite` hierarchy under `src/oxygine/actor/` and the GL renderer (`STDRenderer`, with the OpenGL calls concentrated in `core/gl/`).

### 3. Facilitator-script reskins

Replace the Conduit/Medium framing in the Introduction and "Meet the team lead" prompts with the studio/engine framing. The two sample lists in the plan need swapping:

**New sample gaps (replaces the Conduit "Sample gaps" bullets):**
- ADRs are ~6 years old, written for the first title's MVP, assumptions no longer hold
- `Actor` inheritance trashes cache locality and makes new entity types slow to add
- the `IVideoDriver` seam has only a GLES2 backend (`VideoDriverGLES20`) and assumes GLES2, so no second API and no new platform
- Single immediate-mode render thread leaves cores idle; frame budget is tight on weaker hardware
- The console port can't be scoped because nobody knows where all the GL lives
- ADR-003 blocks the naive fix: don't thread the renderer before the abstraction exists, and don't run two migrations at once

**New sample team-lead Q&A (replaces the Conduit chatbot questions), Sofia as example:**
- *"Who are you?"* → Sofia Lindqvist, engine lead at Northwind Studios.
- *"What's the business?"* → a small studio with one shipped live title and a second greenlit, on our own C++ engine.
- *"What's blocking you?"* → the `Actor` hierarchy is slow to extend and trashes cache; we want to move to an ECS.
- *"How fast do you need this?"* → incrementally, without freezing the live title; console port is committed for ~12 months.
- *"Turn this into a requirements.md file."*

The sample requirements shift from orders/availability/PCI to: target frame budget (60 fps), console-port deadline (~12 months), no feature freeze, incremental migration only, team of nine.

### 4. The data bug is fixed

The old root `CLAUDE.md.template` said "7 people / ≤€5k" while `company.md` said "4 people / €1,200." Both now read **9 engineers** plus the constraint set (no feature freeze, incremental migration only, 12-month console port). No cohort cleanup needed before running.

### 5. Listen for the ADR-003 cross-cutting trap

Whichever feature a student picks, ADR-003 constrains the naive approach: you cannot thread the renderer before a render abstraction exists, and you should not refactor the entity model and the renderer in the same change. A strong agent surfaces that ordering (Yuki's threading feature is effectively blocked on Tariq's abstraction). This is the richest discussion point of the scenario, the optional discussion prompts work well here.

## Recalculated step timings

The codebase decision changes the timing profile. With Conduit-as-design-doc, scenario 3 had no live code scan. Vendoring Oxygine as a real submodule means **Build & test now includes a live scan of unfamiliar C++**, which pushes scenario 3 from its old design-doc profile toward scenario 5's real-code profile.

Assumptions are unchanged from the original report: full student time = agent wall-clock + human work, clean single pass, all adaptation files pre-built. `A` = agent wall-clock (min), `H` = human (typing/reading/posting).

### Changed steps vs the original scenario-3 model

| Step | Original Sc. 3 | Change | New |
|------|----------------|--------|-----|
| 5.5 Analyze (now a **live Oxygine scan** + read unfamiliar C++) | 5.2 (A 1.7 / H 3.5) | A +3.0, H +1.0 | **9.2 (A 4.7 / H 4.5)** |
| 7.3 / 7.5 RFC read (now references real engine code) | 5.0 / 5.0 | H +0.5 each | 5.5 / 5.5 |

All other steps are unchanged from the original scenario-3 numbers (dense engine ADRs already accounted for in Evaluate-by-hand and the recommendation step).

### Per-block fit

| Block | Budget | Original Sc. 3 | Now (with Oxygine scan) | Fit |
|-------|--------|----------------|-------------------------|-----|
| Meet team leads | 15 | 13.0 | 13.0 | ✅ Comfortable |
| Evaluate by hand | 10 | 10.0 | 10.0 | ⚠️ At budget (dense engine ADRs) |
| Skills brainstorm | 10 | 9.0 | 9.0 | ✅ |
| Build & test | 25 | 17.7 | **21.7** | ⚠️ Tightest block, the binding constraint |
| Mermaid MCP | 20 | 11.6 | 11.6 | ✅ (restart is the real cost) |
| RFC sub-agent | 20 | 16.4 | 17.4 | ✅ Thin margin |
| Reflection + Feedback | 15 | 15.0 | 15.0 | ✅ |

**Whole workshop: ~101.7 of 120** (was ~97.7 as a design-doc). Slack drops to ~18 min on a clean pass.

### What this means for the room

- **Build & test is now the binding block** (~21.7 / 25), the same shape as scenario 5. The live scan plus a human reading both an unfamiliar engine summary and the recommendation eats most of 25 minutes. One reprompt and it overruns.
- **Keep the scan small.** Point the agent at `engine/oxygine/src/oxygine/actor/` and `STDRenderer` / `core/gl/` rather than the whole 20k-line engine. That keeps the analyze step near 2.5-3 min instead of 5+.
- **Pre-clone the submodule** (the 173 MB Oxygine checkout) before the session, alongside the Mermaid install. Both are pre-work, not live-time work.
- **Evaluate-by-hand sits exactly at 10/10** because the engine ADRs read longer than Conduit's. If a cohort is slow, trim the RFC compare step (run only the sub-agent RFC) to reclaim ~5 min.

---
*Generated: 2026-06-06*
*Based: Notion "Adaptation variants" (scenario 3), "Building an Architecture Reasoning Agent" workshop plan, the adaptation-timing-report baseline, and the Oxygine repo (pinned 1e9f923)*
*Prompt: Adapt the workshop to scenario 3 (C++ engine internals); replace data + codebase; humanize; write corrections + recalculated timings*
