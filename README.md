# Practice: Building an Architecture Reasoning Agent

To get a head start, please clone the workshop repository before the session.

In this practice, you’ll work through the full architecture reasoning workflow:

- Gather and formalize requirements from a teamlead chatbot
- Evaluate an existing architecture against those requirements
- Design an agent that identifies gaps and proposes improvements
- Generate an architecture diagram using Mermaid and MCP
- Create a specialized subagent to write a structured RFC
- Compare the output of the main agent and the subagent.

The target codebase is **Northwind Studios' in-house C++ engine** (`engine/`, the open-source **Oxygine** framework, added as a git submodule), a C++11 game engine with an `Actor` OOP entity hierarchy, OpenGL ES rendering behind a single-backend `IVideoDriver` seam (`VideoDriverGLES20`), and a single update/render thread.

**You’ll work with a C++ engine during this workshop, but you don’t need to install it separately: everything is already included and adapted in the repository.**

------------⚠️ **Spoiler alert! Continue only during the workshop or while working independently after it.** -----------


Three team lead personas are provided, each requests a different engine-modernization feature. Pick one to start:

| Persona | Directory | Feature requested |
|---------|-----------|-------------------|
| **Sofia Lindqvist** | `team-lead/` | Migrate entities to a data-oriented ECS (off the `Actor` hierarchy) |
| **Tariq Hassan** | `team-lead-2/` | Render-backend abstraction so DX12 / Vulkan / Metal can sit under OpenGL |
| **Yuki Sato** | `team-lead-3/` | Multithreaded renderer + job system (off the single immediate-mode thread) |

---

## Repository layout

```
arch-reasoning-agent-practice/
├── engine/                                ← the real codebase under analysis (git submodule: Oxygine)
│   └── oxygine/src/                        ← C++ engine source: Actor/Sprite hierarchy + GL renderer
│
├── data/
│   ├── context/company.md             ← Northwind Studios: business context, pain points, constraints
│   ├── adrs/
│   │   ├── ADR-001-entity-model.md    ← minimal ADR (intentionally thin)
│   │   ├── ADR-002-rendering-api.md   ← well-structured ADR
│   │   └── ADR-003-render-threading.md ← outdated single-thread ADR
│   └── diagrams/
│       └── current-architecture.mmd   ← current monolithic-engine topology (Mermaid)
│
├── skills/                            ← COMPLETE, do not modify
│   ├── list_adrs/list_adrs.py         ← lists all ADRs as JSON
│   ├── read_adr/read_adr.py           ← reads one ADR by ID
│   ├── write_adr/write_adr.py         ← validates + saves new ADR to output/adrs/
│   └── save_diagram/save_diagram.py   ← validates + saves Mermaid to output/diagrams/
│
├── output/
│   ├── adrs/                          ← agent writes new ADRs here
│   └── diagrams/                      ← agent writes new diagrams here
│
├── team-lead/
│   └── CLAUDE.md.template             ← Sofia Lindqvist: ECS migration
├── team-lead-2/
│   └── CLAUDE.md.template             ← Tariq Hassan: render-backend abstraction
├── team-lead-3/
│   └── CLAUDE.md.template             ← Yuki Sato: multithreaded renderer + job system
│
├── .claude/agents/
│   ├── team_lead.md.template          ← Sofia Lindqvist subagent (used by main agent)
│   ├── team_lead_2.md.template        ← Tariq Hassan subagent (used by main agent)
│   ├── team_lead_3.md.template        ← Yuki Sato subagent (used by main agent)
│   └── rfc_writer.md.template         ← Step 4: build the RFC writer subagent
│
├── CLAUDE.md.template                 ← Step 3: build the architecture reasoning agent
└── README.md
```

---

## Step 0: Setup

Populate the engine submodule (the real code the agent reasons about), then verify the skills work:

```bash
git submodule update --init        # checks out the pinned Oxygine commit into engine/
python skills/list_adrs/list_adrs.py
python skills/read_adr/read_adr.py ADR-001
```

The submodule is pinned to a specific Oxygine commit for reproducibility, so everyone reads the same code.

Read `data/context/company.md` to understand the scenario, then browse `engine/oxygine/src/` to see the actual code the agent will reason about, the `Actor`/`Sprite` class hierarchy and the renderer (`STDRenderer` → `IVideoDriver`, with the GLES backend in `core/gl/`).

---

## Step 1: Talk to your Team Lead

Three personas are available. Pick one (or talk to all three for extra practice).

| Persona | Feature | Templates to copy |
|---------|---------|-------------------|
| Sofia Lindqvist | ECS migration | `team-lead/CLAUDE.md.template` → `team-lead/CLAUDE.md`<br>`.claude/agents/team_lead.md.template` → `.claude/agents/team_lead.md` |
| Tariq Hassan | Render-backend abstraction | `team-lead-2/CLAUDE.md.template` → `team-lead-2/CLAUDE.md`<br>`.claude/agents/team_lead_2.md.template` → `.claude/agents/team_lead_2.md` |
| Yuki Sato | Multithreaded renderer | `team-lead-3/CLAUDE.md.template` → `team-lead-3/CLAUDE.md`<br>`.claude/agents/team_lead_3.md.template` → `.claude/agents/team_lead_3.md` |

Test the chatbot for your chosen persona:

```bash
cd team-lead        # or team-lead-2 / team-lead-3
claude
```

**Good opening questions by persona:**
- Sofia: *"What's the smallest subsystem we could move to ECS first to prove it works?"*
- Tariq: *"`IVideoDriver` only has a GLES2 backend, how do we turn it into a real multi-backend seam?"*
- Yuki: *"What breaks if we move rendering onto worker threads the naive way?"*

The team lead should answer in engineering-delivery language and push back when proposals sound like a big-bang rewrite or threaten the live game.

---

## Step 2: Read and analyse the existing ADRs

Before building the agent, spend 10 minutes reading the three ADRs in `data/adrs/` manually.

- What was decided and when? (All three date to 2019, written for the first title's MVP.)
- Which ADR explicitly flags that a second graphics API is blocked until `IVideoDriver` becomes a real multi-backend seam (today it has only a GLES2 backend)?
- What does ADR-003 warn about for threading the renderer, and about refactoring the entity model and the renderer at the same time?

This shapes the requirements section of your agent.

---

## Step 3: Build the Architecture Reasoning Agent

This is the main agent. It reads ADRs, identifies gaps, proposes options, generates diagrams, and writes new ADRs.

1. Copy the template:

```bash
cp CLAUDE.md.template CLAUDE.md
```

2. Fill in every `[TODO]`. Required sections:

| Section | What to write |
|---|---|
| **Your role** | One-sentence identity |
| **Purpose** | Full scope: what the agent reads, analyses, produces |
| **Trigger** | Exact phrases that start the workflow |
| **Workflow** | ≥ 10 numbered steps (see hints in template) |
| **Output format** | Gap analysis table + option structure with costs/risks/timelines |
| **Guardrails** | Must include the ADR write gate and diagram gate |
| **Failure handling** | What to do when each tool or subagent fails |

### Required guardrails

Your `CLAUDE.md` **must** include both of these rules:

> `write_adr` must **not** be called unless the user's message contains **"write adr"**, **"save adr"**, or **"publish"**.

> `save_diagram` must **not** be called before the user has selected a specific option.

3. Run the agent:

```bash
claude          # from the arch-reasoning-agent-practice/ directory
```

Send: `analyse the architecture`

Confirm it:
- Reads `company.md` and all 3 ADRs
- Identifies at least 3 architectural gaps related to your chosen engine feature
- Proposes 2-3 options with cost estimates and timelines
- **Waits.** Does not call `write_adr` or `save_diagram` yet

4. Drive it through the full flow:
   - Select an option → diagram should be saved to `output/diagrams/`
   - Say `write adr` → ADR should be saved to `output/adrs/`

---

## Step 4: Add the RFC writer subagent

After the agent writes an ADR, it should produce a complete RFC document that combines everything: requirements, architecture, ADRs, diagram, and migration plan.

1. Copy the template:

```bash
cp .claude/agents/rfc_writer.md.template .claude/agents/rfc_writer.md
```

2. Fill in every `[TODO]`:

| Section | What to write |
|---|---|
| **Your role** | One-sentence identity |
| **Purpose** | What document it produces and who reads it |
| **RFC sections** | Ordered list of ≥ 8 sections the RFC must contain |
| **Output format** | How each section is structured |
| **Guardrails** | What the RFC writer must never do |

3. Update your `CLAUDE.md` to invoke `rfc_writer` after the ADR is written. The subagent receives:
   - Full company context
   - All ADR content (existing + new)
   - The selected architecture option
   - Diagram path in `output/diagrams/`
   - Requirements summary from the team lead session

4. Re-run the full workflow and verify `output/rfc.md` is created.

---

## Setting up the Mermaid diagram plugin

The architecture agent can render and live-preview Mermaid diagrams in your browser using the **claude-mermaid** Claude Code plugin. This is optional but recommended, without it the agent can still write `.mmd` files, but you won't get live previews.

The plugin is hosted on GitHub and must be registered as a custom marketplace source before installation.

### 1. Register the marketplace source

Add the following to your **user-level** Claude Code settings (`~/.claude/settings.json`). Create the file if it doesn't exist; merge with your existing config if it does.

```json
{
  "extraKnownMarketplaces": {
    "claude-mermaid": {
      "source": {
        "source": "github",
        "repo": "veelenga/claude-mermaid"
      }
    }
  }
}
```

### 2. Install the plugin

Open Claude Code (any project) and run:

```
/plugin install claude-mermaid
```

Claude Code will download the plugin from the GitHub repo and cache it locally. You will see a confirmation message when it is ready.

### 3. Enable the plugin

The install command enables the plugin automatically. You can verify it is active by checking that your `~/.claude/settings.json` now contains:

```json
"enabledPlugins": {
  "claude-mermaid@claude-mermaid": true
}
```

### 4. Verify it works

From this project's root directory, open Claude Code and ask:

```
draw a simple mermaid flowchart with three boxes
```

A browser tab should open with a live-preview of the diagram. The tab auto-refreshes whenever the diagram is updated.

### Permissions

The `.claude/settings.json` in this repo already pre-approves the two mermaid tool calls (`mermaid_preview` and `mermaid_save`) so you will not be prompted to allow them during the exercise.

---

## Submission checklist

- [ ] At least one team lead `CLAUDE.md` exists in `team-lead/`, `team-lead-2/`, or `team-lead-3/`
- [ ] Matching `.claude/agents/team_lead*.md` file exists for your chosen persona
- [ ] Team lead chatbot responds in engineering/delivery language (test: propose a big-bang rewrite, they should push back on risk or timeline)
- [ ] `CLAUDE.md` exists, no `[TODO]` tokens remain
- [ ] ADR write gate present in `CLAUDE.md`
- [ ] Diagram gate present in `CLAUDE.md`
- [ ] Agent reads all 3 ADRs before proposing options
- [ ] Agent presents 2-3 options and **waits** before writing anything
- [ ] `output/diagrams/` contains a generated diagram
- [ ] `output/adrs/` contains a generated ADR
- [ ] `.claude/agents/rfc_writer.md` exists, no `[TODO]` tokens remain
- [ ] `output/rfc.md` exists after full workflow run

---

## Tips

- Read `data/context/company.md` carefully, your agent's gap analysis should map directly to the pain points described there, especially the GLES-only `IVideoDriver` seam and the single render thread that together block any new platform.
- Browse `engine/oxygine/src/`, the agent should be able to reason about what the real code actually does (the `Actor`/`Sprite` inheritance tree, the GLES backend behind `IVideoDriver`), not just the company doc.
- The write gate is the most important guardrail. Test that the agent does **not** call `write_adr` when you say "looks good" or "I approve".
- If the agent skips a step, add more explicit ordering language to the workflow section ("do not proceed to step N until step N-1 is complete").
- Watch for the cross-cutting trap in ADR-003: threading the renderer (Yuki's feature) is blocked until the render abstraction (Tariq's feature) exists, and the two big migrations should not run at the same time. A good agent surfaces that ordering.
- For Cursor or Windsurf: use the same template content, saved to `.cursor/rules/architecture-agent.mdc` or `.windsurfrules` respectively. For the Team Lead, open `team-lead/` as a separate workspace root.
