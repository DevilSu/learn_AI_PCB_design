# <span style="color: #88C0D0;">**Open-Source AI PCB Design Landscape vs. Our KiCad 9 Setup**</span>

### <span style="color: #81A1C1;">**Filename:** `open_source_ai_pcb_comparison.md`</span>

| <span style="color: #EBCB8B;">**Field**</span> | <span style="color: #EBCB8B;">**Details**</span> |
| :--- | :--- |
| <span style="color: #EBCB8B;">**Author**</span> | devilsu |
| <span style="color: #EBCB8B;">**Created**</span> | 2026-11-19 17:24 PST |
| <span style="color: #EBCB8B;">**Modified**</span> | 2026-11-20 19:24 PST |
| <span style="color: #EBCB8B;">**Version**</span> | v1.1.0 |
| <span style="color: #EBCB8B;">**Status**</span> | <span style="color: #A3BE8C;">**Active Ecosystem Research**</span> |
| <span style="color: #EBCB8B;">**Scope**</span> | Open-Source / Non-Paid AI PCB Tools (2025–2026) vs. Native Ubuntu 26.04 KiCad 9 + `.venv` |

---

## <span style="color: #88C0D0;">**Table of Contents**</span>
- [<span style="color: #88C0D0;">**1. The Four Open-Source AI PCB Workflows on GitHub**</span>](#1-the-four-open-source-ai-pcb-workflows-on-github)
  - [<span style="color: #81A1C1;">1.1 Ecosystem Taxonomy Diagram</span>](#11-ecosystem-taxonomy-diagram)
  - [<span style="color: #81A1C1;">1.2 Paradigm A: Hardware-as-Code (HaC) Compilers</span>](#12-paradigm-a-hardware-as-code-hac-compilers)
  - [<span style="color: #81A1C1;">1.3 Paradigm B: KiCad MCP Servers & Native Action Plugins</span>](#13-paradigm-b-kicad-mcp-servers--native-action-plugins)
  - [<span style="color: #81A1C1;">1.4 Paradigm C: Multi-Agent Harnesses with Git & Pin Review</span>](#14-paradigm-c-multi-agent-harnesses-with-git--pin-review)
  - [<span style="color: #81A1C1;">1.5 Paradigm D: Open-Source Algorithmic / GPU Routing & CI Packaging</span>](#15-paradigm-d-open-source-algorithmic--gpu-routing--ci-packaging)
- [<span style="color: #88C0D0;">**2. Step-by-Step Process Comparison: Community vs. Our Setup**</span>](#2-step-by-step-process-comparison-community-vs-our-setup)
- [<span style="color: #88C0D0;">**3. Pros & Cons Comparison Matrix**</span>](#3-pros--cons-comparison-matrix)
- [<span style="color: #88C0D0;">**4. Bottom-Line Verdict & Borrowed Open-Source Upgrades**</span>](#4-bottom-line-verdict--borrowed-open-source-upgrades)

---

## <span style="color: #88C0D0;">**1. The Four Open-Source AI PCB Workflows on GitHub**</span>

### <span style="color: #81A1C1;">**1.1 Ecosystem Taxonomy Diagram**</span>

> [!IMPORTANT]
> **Key Takeaway Across Open-Source Repos (2025–2026)**
> Almost nobody in the open-source ecosystem lets an LLM emit raw `.kicad_pcb` or `.kicad_sch` S-expressions from scratch—raw coordinate hallucination causes broken pin connections and unreadable schematics. Instead, successful open-source workflows put a **deterministic abstraction layer** (a Python/DSL compiler, an MCP tool server, or `pcbnew` + an algorithmic autorouter) between the AI agent and KiCad, backed by `kicad-cli` DRC/ERC and rendered image feedback.

```mermaid
flowchart TD
    AI["LLM Coding Agent (Antigravity / Claude Code / Codex)"]

    subgraph P1 ["A. Hardware-as-Code Compilers"]
        HAC["atopile (.ato) / tscircuit (.tsx) / circuit-synth (.py)"]
    end

    subgraph P2 ["B. KiCad MCP & GUI Plugins"]
        MCP["Konnect (Rust) / kicad-copilot / mcp-kicad-sch-api"]
    end

    subgraph P3 ["C. Multi-Agent Review Harnesses"]
        HAR["kicad-agent / CircuitBot / Circuitron (Git + Web UI Pins)"]
    end

    subgraph OURS ["D. Our Direct Python + CLI Stack"]
        VENV["Python .venv (circuit-synth / kicad-sch-api / kiutils / SKiDL + pcbnew)"]
    end

    KICAD["Native KiCad Project (.kicad_sch + .kicad_pcb)"]
    ROUTE["Algorithmic Autorouters (Freerouting CPU / OrthoRoute GPU)"]
    VERIFY["Headless Verification & Fab (kicad-cli / KiBot)"]

    AI -->|"Writes DSL / Code"| HAC
    AI -->|"Calls 50-200+ MCP Tools"| MCP
    AI -->|"Runs Sub-Agent Pipeline"| HAR
    AI -->|"Executes Python Scripts"| VENV

    HAC -->|"Compiles Netlist & Syncs"| KICAD
    MCP -->|"Mutates via IPC / Files"| KICAD
    HAR -->|"Commits Atomic Edits"| KICAD
    VENV -->|"Manipulates API & Files"| KICAD

    KICAD <-->|"DSN / SES or IPC"| ROUTE
    KICAD -->|"ERC, DRC, PNG, Gerbers"| VERIFY
```

### <span style="color: #81A1C1;">**1.2 Paradigm A: Hardware-as-Code (HaC) Compilers**</span>

Engineers and AI agents write declarative code rather than drawing schematics; a compiler validates constraints/types, selects passives, and syncs to `.kicad_pcb`:

- <span style="color: #EBCB8B;">**`atopile` (merged with `faebryk`, MIT):**</span> Declarative `.ato` + Python constraint solver (`V_out = 3.3V +/- 5%`) that calculates resistor dividers/decoupling automatically and syncs layout with KiCad.
- <span style="color: #EBCB8B;">**`tscircuit` (MIT):**</span> React/TypeScript (`.tsx`) circuit components with built-in browser preview, WebAssembly `ngspice`, autorouting, and KiCad export.
- <span style="color: #EBCB8B;">**`circuit-synth` & `kicad-sch-api` (MIT, pure Python):**</span> Built specifically for AI agents; generates <span style="color: #EBCB8B;">**clean, Manhattan-wired, human-readable `.kicad_sch` sheets**</span> that stay bidirectionally synced with KiCad, rather than dumping raw netlists.
- <span style="color: #EBCB8B;">**`absmach/synth` (`SynthSpec`, Apache-2.0/MIT):**</span> Concise `.synth` text spec compiled deterministically into KiCad projects.

### <span style="color: #81A1C1;">**1.3 Paradigm B: KiCad MCP Servers & Native Action Plugins**</span>

Instead of writing one-off scripts, the AI talks to KiCad through structured Model Context Protocol (MCP) tool calls (similar in architecture to your existing `.vivado-mcp-server`):

- <span style="color: #EBCB8B;">**`mixelpixx/Konnect` & `biosshot/kicad-copilot`:**</span> Expose 50–200+ granular MCP tools (`place_symbol`, `add_wire`, `place_footprint`, `run_drc`, `export_gerbers`) over KiCad's IPC API and `kicad-cli`.
- <span style="color: #EBCB8B;">**`circuit-synth/mcp-kicad-sch-api`:**</span> Dedicated Python MCP server for schematic capture, hierarchical sheets, and orthogonal wire routing.
- <span style="color: #EBCB8B;">**`paul356/KiCad-AI-Assistant` & `aklofas/kicad-happy`:**</span> Embedded KiCad GUI chat panels and reusable agent skills for datasheet cross-checking, SPICE runs, and DFM audits.

### <span style="color: #81A1C1;">**1.4 Paradigm C: Multi-Agent Harnesses with Git & Pin Review**</span>

- <span style="color: #EBCB8B;">**`kicad-agent`, `CircuitBot` (LangGraph), `Circuitron`:**</span> Wrap KiCad inside multi-agent pipelines (*Planner -> Part Finder -> SKiDL/Code Generator -> Validator*). `kicad-agent` commits every layout step to Git (rolling back automatically if DRC worsens) and serves a local Three.js web viewer where the human clicks a trace/pad to drop a spatial review comment for the agent's next pass.

### <span style="color: #81A1C1;">**1.5 Paradigm D: Open-Source Algorithmic / GPU Routing & CI Packaging**</span>

Because LLMs are poor at raw geometric maze-solving across thousands of obstacles, virtually <span style="color: #EBCB8B;">**all**</span> open-source AI workflows hand off physical trace routing and fab packaging to deterministic backends:

- <span style="color: #EBCB8B;">**`freerouting/freerouting` (GPL/Open Source, Java):**</span> Industry-standard Specctra `.dsn`/`.ses` CPU autorouter for 2–6 layer boards.
- <span style="color: #EBCB8B;">**`bbenchoff/OrthoRoute` (Open Source, Python + CUDA/CuPy):**</span> GPU-accelerated Manhattan lattice autorouter using KiCad 9's IPC API—built specifically for dense BGA escapes, FPGA carrier boards, and high-channel-count arrays where CPU `Freerouting` stalls.
- <span style="color: #EBCB8B;">**`PCBRouteAgent` / `PCBWorld` (Research / Open Source):**</span> Multimodal LLM global planner driving KiCad's native push-and-shove router step-by-step via Gymnasium/RL wrappers.
- <span style="color: #EBCB8B;">**`INTI-CMNB/KiBot` & `InteractiveHtmlBom` (GPL/MIT):**</span> Single YAML config (`kibot.yaml`) that automates ERC/DRC pre-flight checks and packages exact manufacturer-formatted ZIPs (JLCPCB, PCBWay, OSH Park) plus interactive assembly HTML docs.

---

## <span style="color: #88C0D0;">**2. Step-by-Step Process Comparison: Community vs. Our Setup**</span>

| <span style="color: #EBCB8B;">**Design Stage**</span> | <span style="color: #EBCB8B;">**Typical Community HaC / MCP Process**</span> | <span style="color: #EBCB8B;">**Our Planned KiCad 9 + `.venv` Process**</span> | <span style="color: #EBCB8B;">**Key Difference**</span> |
| :--- | :--- | :--- | :--- |
| <span style="color: #EBCB8B;">**1. Architecture & Spec**</span> | Prompt agent -> writes `.ato`, `.tsx`, or calls MCP `create_project` | Align on `blueprints/` markdown spec (like `mic_arr_v2`), then run deterministic Python generators | Ours matches your repo blueprint/evidence workflow without learning a custom `.ato`/`.tsx` language |
| <span style="color: #EBCB8B;">**2. Part Sourcing**</span> | `atopile` registry, `tscircuit` registry, or MCP JLCPCB search tools | Stock `/usr/share/kicad` + `easyeda2kicad --lcsc_id=Cxxxx` inside `.venv` | Equivalent LCSC/JLCPCB coverage with zero cloud registry dependency |
| <span style="color: #EBCB8B;">**3. Schematic Capture**</span> | `circuit-synth` / `kicad-sch-api` or MCP draws neat orthogonal `.kicad_sch` sheets; `atopile` skips visual schematics | `SKiDL` (code netlist/ERC) + `kiutils` (raw `.kicad_sch` S-expression manipulation) | <span style="color: #EBCB8B;">**Gap in our initial plan:**</span> `SKiDL` + raw `kiutils` produce messy visual schematics unless paired with `kicad-sch-api` |
| <span style="color: #EBCB8B;">**4. Component Placement**</span> | Often manual in KiCad GUI, or LLM places parts one-by-one via MCP tool calls | Algorithmic Python `pcbnew` scripts compute exact coordinates (`mm`, angles, array pitch, decoupling offsets) | <span style="color: #EBCB8B;">**Our setup is stronger for sensor/mic arrays:**</span> Python math places 16–64 mic channels & mounting holes to sub-micron precision in 1 run |
| <span style="color: #EBCB8B;">**5. Copper Routing**</span> | Export to `Freerouting`, GPU `OrthoRoute`, or interactive KiCad GUI push-and-shove | Scripted `pcbnew` traces (for symmetric buses/arrays) + headless `Freerouting` CLI (`.dsn` -> `.ses`) | Identical core routing engine; dense FPGA boards may still need `OrthoRoute` or manual GUI push-and-shove |
| <span style="color: #EBCB8B;">**6. Audit & Fab Export**</span> | `kicad-cli` DRC/ERC + `KiBot` (`kibot.yaml`) for one-shot JLCPCB BOM/CPL/Gerber packaging | `kicad-cli` JSON ERC/DRC + `rsvg-convert`/3D PNG visual self-review + `kicad-cli` Gerber/STEP exports | Community `KiBot` automates JLCPCB CSV column formatting (`Comment`, `Designator`, `Footprint`, `LCSC Part #`) cleaner than raw `kicad-cli` |

---

## <span style="color: #88C0D0;">**3. Pros & Cons Comparison Matrix**</span>

| <span style="color: #EBCB8B;">**Approach**</span> | <span style="color: #EBCB8B;">**Pros**</span> | <span style="color: #EBCB8B;">**Cons**</span> |
| :--- | :--- | :--- |
| <span style="color: #EBCB8B;">**A. Hardware-as-Code (`atopile`, `tscircuit`)**</span> | - Clean Git diffs & reusable parametric subcircuits (LDOs, USB-C, buck converters)<br/>- Compiler catches voltage/pin-type mismatches before layout<br/>- LLMs write Python/TSX with very low syntax error rates | - <span style="color: #EBCB8B;">**Dependency friction on Ubuntu 26.04 (`Python 3.14`):**</span> `atopile` often pins older Python (`3.11`/`3.12`), and `tscircuit` requires Node.js/React<br/>- `atopile` treats schematics as code-only (no clean human-readable `.kicad_sch` PDF for review)<br/>- Harder to import legacy KiCad schematics |
| <span style="color: #EBCB8B;">**B. KiCad MCP Servers (`Konnect`, `kicad-copilot`)**</span> | - Zero custom boilerplate scripts needed for simple edits<br/>- Semantic guardrails prevent corrupting S-expression syntax<br/>- Live interaction with an open KiCad GUI window via IPC API | - Chatty token overhead (placing 64 decoupling caps via individual MCP tool calls is slow vs. a 5-line Python loop)<br/>- Several IPC-based MCP tools require an active GUI session rather than 100% headless CLI execution<br/>- Fast-moving repos (`Konnect` targets KiCad 10 / shifts packaging) |
| <span style="color: #EBCB8B;">**C. Multi-Agent Harnesses (`kicad-agent`, `CircuitBot`)**</span> | - Built-in Git rollback when routing/placement degrades DRC<br/>- Click-to-pin visual web UI makes spatial human feedback intuitive | - Heavy framework overhead (FastAPI, React, LangGraph, external API keys) duplicating what Antigravity + Git + image inspection already do natively |
| <span style="color: #EBCB8B;">**D. Our Native KiCad 9 + `.venv` Stack**</span> | - <span style="color: #EBCB8B;">**Zero stack bloat:**</span> Native `apt` KiCad 9.0.8 + your exact `setup_env.sh` (`Python 3.14` `.venv`)<br/>- <span style="color: #EBCB8B;">**100% Headless & Reproducible:**</span> Every board can be regenerated deterministically from a script (`RULE-EVID-001` friendly)<br/>- <span style="color: #EBCB8B;">**Unbeatable for parametric geometry:**</span> Exact polar/Cartesian math for mic arrays, RF/acoustic spacing, and stitching vias<br/>- Multimodal visual self-audit via `kicad-cli pcb render` & `rsvg-convert` | - <span style="color: #EBCB8B;">**Weakness 1 (Schematic Aesthetics):**</span> Writing `.kicad_sch` via raw `kiutils` or `SKiDL` makes ugly schematic diagrams unless we add `kicad-sch-api` / `circuit-synth`<br/>- <span style="color: #EBCB8B;">**Weakness 2 (JLCPCB Packaging):**</span> Raw `kicad-cli` needs custom CSV post-processing for turnkey PCBA BOM/CPL unless we add `KiBot`<br/>- <span style="color: #EBCB8B;">**Weakness 3 (High-Density FPGA Routing):**</span> CPU `Freerouting` struggles on tight BGA/high-speed differential pairs |

---

## <span style="color: #88C0D0;">**4. Bottom-Line Verdict & Borrowed Open-Source Upgrades**</span>

Our core architecture—<span style="color: #EBCB8B;">**native `apt` KiCad 9 + `setup_env.sh` (`--system-site-packages` `.venv`) + `pcbnew` parametric Python + `kicad-cli` JSON/PNG verification + `Freerouting`**</span>—is actually <span style="color: #EBCB8B;">**cleaner, faster, and more reproducible**</span> than heavy multi-agent harnesses (`CircuitBot`) or Node.js/pinned-Python DSLs (`tscircuit`, `atopile`), especially on `Python 3.14` and for your deterministic hardware projects (like microphone arrays and FPGA peripherals).

> [!TIP]
> **Three Zero-Cost Open-Source Upgrades Worth Borrowing from the Community**
> 1. **Add `kicad-sch-api` (and/or `circuit-synth`, MIT) to `requirements.txt`:** Solves our biggest gap—letting Python generate **clean, human-readable, orthogonally wired `.kicad_sch` schematics** instead of netlist-only `SKiDL` or messy hand-computed S-expression coordinates.
> 2. **Add `KiBot` (`kibot` / `InteractiveHtmlBom`, open source) for Fab Export:** Replaces ad-hoc export scripts with a single declarative `kibot.yaml` that runs pre-flight ERC/DRC and emits ready-to-upload JLCPCB/PCBWay Gerber + BOM + CPL archives and interactive HTML assembly maps.
> 3. **Keep `bbenchoff/OrthoRoute` in Mind for Future High-Density FPGA Boards:** Since this PC has an NVIDIA GPU, `OrthoRoute` (open-source CUDA autorouter over KiCad 9's IPC API) is a strong future add-on if you ever design dense multi-layer FPGA carrier or large-channel-count array boards that outgrow Java `Freerouting`.
