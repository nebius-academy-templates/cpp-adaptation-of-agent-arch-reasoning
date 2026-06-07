# Architecture Reasoning Agent Workshop: Engine Internals (Render-Backend Abstraction + ECS)

Build three AI agents that reason about a real C++ game engine: a team-lead chatbot, an architecture-reasoning orchestrator, and an RFC-writer sub-agent. You give them the studio context, the existing ADRs, and the real engine source; they turn a feature request into evaluated options, a target diagram, a new ADR, and an RFC.

This is the engine-internals adaptation (render-backend abstraction plus ECS) of the original, language-agnostic workshop. The reference codebase is the open-source Oxygine engine, pinned as a git submodule, so the agents reason about code that actually exists.

## Sources

- Original workshop (upstream, language-agnostic): https://github.com/nebius-academy-templates/agent-arch-reasoning-practice
- This fork: https://github.com/mrtoxfox/agent-arch-reasoning-practice
- This scenario lives on the branch `engine-render-abstraction-ecs`.
- Reference engine (the code under analysis): Oxygine, https://github.com/oxygine/oxygine-framework, pinned at commit `1e9f923`, MIT licensed.

What changed from the original: the Conduit Node.js sample app was removed; the company context, the three ADRs, the current-architecture diagram, and the three team-lead personas were rewritten for a C++ engine studio; and Oxygine was vendored as the real codebase. See `scenario-3-adaptation-summary.md` for the full inventory and the facilitator timings.

## The scenario in one paragraph

Northwind Studios ships a live C++ game on its in-house engine (Oxygine). Entities use a deep `Actor` inheritance tree, rendering is GLES-only behind a half-built `IVideoDriver` seam, and the renderer runs immediate-mode on a single thread. Three leads each want a different modernization: Sofia (ECS migration), Tariq (a real render-backend abstraction), Yuki (a multithreaded renderer). ADR-003 is the trap: do not thread before the abstraction exists, and do not refactor the entity model and the renderer in the same change.

## Repository layout

```
agent-arch-reasoning-practice/
├── engine/                         <- reference codebase (git submodule: Oxygine)
│   └── oxygine/src/                <- C++ engine source: Actor/Sprite tree + renderer
├── data/
│   ├── context/company.md          <- Northwind Studios: context, pains, constraints
│   ├── adrs/                       <- ADR-001 entity model, ADR-002 rendering API, ADR-003 render threading
│   └── diagrams/current-architecture.mmd (+ .md copy)
├── skills/                         <- 4 Python skills (list/read/write ADR, save diagram); complete, do not modify
├── output/                         <- agents write generated ADRs, diagrams, and the RFC here
├── team-lead/  team-lead-2/  team-lead-3/   <- the three persona CLAUDE.md templates
├── .claude/agents/                 <- subagent templates (team_lead*, rfc_writer)
├── CLAUDE.md.template              <- the architecture agent you build
└── scenario-3-adaptation-summary.md
```

## Setup

### Prerequisites
- Git (with submodule support)
- Python 3.8 or newer
- Claude Code (recommended) or Cursor

### macOS
```bash
# 1. Install prerequisites (Homebrew)
brew install git python

# 2. Clone with the engine submodule
git clone --recurse-submodules https://github.com/mrtoxfox/agent-arch-reasoning-practice.git
cd agent-arch-reasoning-practice
git checkout engine-render-abstraction-ecs

# If you already cloned without --recurse-submodules:
git submodule update --init

# 3. Smoke-test the skills
python3 skills/list_adrs/list_adrs.py
python3 skills/read_adr/read_adr.py ADR-001
```

### Windows
Use Git Bash (ships with Git for Windows) or PowerShell. Install Python from python.org and tick "Add python.exe to PATH".
```powershell
# 1. Clone with the engine submodule
git clone --recurse-submodules https://github.com/mrtoxfox/agent-arch-reasoning-practice.git
cd agent-arch-reasoning-practice
git checkout engine-render-abstraction-ecs

# If you already cloned without --recurse-submodules:
git submodule update --init

# 2. Smoke-test the skills (use `py`, or `python` if that is your launcher)
py skills\list_adrs\list_adrs.py
py skills\read_adr\read_adr.py ADR-001
```
In Git Bash on Windows, forward slashes and `python3` also work. The submodule checkout is about 173 MB, so do it before a session, not live.

## Running the workshop

1. Team lead. Pick a persona, copy its template to `CLAUDE.md`, and interview it:
   ```bash
   cp team-lead/CLAUDE.md.template team-lead/CLAUDE.md   # Sofia (ECS); or team-lead-2 (Tariq), team-lead-3 (Yuki)
   cd team-lead && claude
   ```
   Turn the conversation into a `requirements.md`.
2. Architecture agent. Copy `CLAUDE.md.template` to `CLAUDE.md`, fill the `[TODO]`s (keep both guardrail gates), then run `claude` and send `analyse the architecture`. It reads `company.md`, the three ADRs, the diagram, and the real engine source, proposes options, and waits.
3. Target diagram. After you select an option, it writes a Mermaid target diagram to `output/diagrams/`.
4. ADR. Say `write adr` and it saves ADR-004 via the `write_adr` skill.
5. RFC. It hands off to the `rfc_writer` sub-agent to assemble `output/rfc.md`.

Keep the engine scan small: point the agent at `engine/oxygine/src/oxygine/actor/` and the renderer (`STDRenderer`, `core/gl/`), not the whole engine.

## Mermaid diagrams (optional)

The agent can write Mermaid into a `.md` file directly, no extra setup. For live previews, install the `claude-mermaid` plugin from the Claude Code marketplace, then restart Claude Code.

## Credits and license

Workshop structure and the Python skills come from the upstream original (nebius-academy-templates). The reference engine is Oxygine (MIT). This fork adapts the workshop to engine internals.
