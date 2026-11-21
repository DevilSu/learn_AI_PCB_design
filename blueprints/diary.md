# learn_AI_PCB_design: Project Diary & Engineering Narrative

> **Rule Reference**: `RULE-DIARY-001` (`dvs_skills/rules/project_diary.md`)  
> **Status**: **Append-Only & Strictly Immutable** (Historical Record)  
> **Target Media**: Source Material for the *Acquired*-Style Project Retrospective Podcast  

---

## Day 1 — 2026-11-20: Origins — Architecting the Open-Source AI PCB Pipeline & Progressive Testbed

### 1. The Goal & Hypothesis
The goal of `learn_AI_PCB_design` is to build a 100% local, open-source, script-reproducible AI-assisted PCB design and JLCPCB PCBA manufacturing pipeline centered on KiCad 9 on Ubuntu 26.04 LTS, then stress-test that pipeline progressively on real hardware boards—starting with a 2-layer `pcb_breadboard`, advancing to a parametric SMT `led_array`, and culminating in a mixed-signal/power `buck_12v_to_5v` converter board before applying the workflow to complex FPGA/sensor arrays.

Our working hypothesis is that an AI agent fails when asked to one-shot raw KiCad S-expressions or guess trace coordinates, but succeeds reliably when constrained behind deterministic Python schematic/layout generators (`kicad-sch-api`, `circuit-synth`, `pcbnew`), algorithmic routing (`Freerouting`), automated pre-flight validation (`kicad-cli` ERC/DRC + 3D PNG inspection + `KiBot`), and strict human verification gates for pinouts, 3D mechanical clearance, and vendor SMT placement previews.

### 2. Actions & Technical Decisions
* Audited the host workstation (`Ubuntu 26.04.1 LTS`, `Python 3.14.4`) and verified that `kicad 9.0.8+dfsg-1`, `kicad-libraries`, `kicad-packages3d`, `librsvg2-bin`, and `default-jre-headless` are all available in Ubuntu's native `universe` and `main` repositories without PPAs.
* Examined existing workspace Python conventions (`mic_arr_spatial`, `realtime_anc`, `mic_arr_v2`, `gentoo_llwvs`) and standardized on `setup_env.sh` + `python3 -m venv --system-site-packages .venv` + `requirements.txt` rather than Conda or `uv`.
* Surveyed active open-source AI PCB design tools on GitHub (`atopile`, `tscircuit`, `circuit-synth`, `kicad-sch-api`, `Konnect`, `kicad-copilot`, `kicad-agent`, `Freerouting`, `OrthoRoute`, `KiBot`) and compared them against our native KiCad 9 + `.venv` stack in [open_source_ai_pcb_comparison.md](research/open_source_ai_pcb_comparison.md).
* Selected `kicad-sch-api` + `circuit-synth` (clean orthogonal schematic generation), `kiutils` + `SKiDL` + `kicad-python` (layout scripting, Python ERC, and IPC API), `easyeda2kicad` (LCSC part import), `Freerouting` (headless Specctra routing), and `KiBot` + `InteractiveHtmlBom` (turnkey JLCPCB packaging), documented in [kicad_setup_guide.md](research/kicad_setup_guide.md).
* Codified a 7-stage gated practitioner workflow with concrete prompt templates and 3 mandatory human verification gates in [ai_pcb_design_playbook.md](research/ai_pcb_design_playbook.md).
* Scaffolded the tripartite repository structure (`blueprints/`, `hw/`, `sw/`) under `learn_AI_PCB_design` in compliance with `RULE-REPO-001`.

### 3. Failures, Anomalies & Rabbit Holes
* **The Sandboxed Package Trap**: Installing KiCad via Snap or Flatpak isolates the runtime filesystem and traps the compiled C++ `pcbnew` module away from external Python virtual environments.
* **The Conda ABI Mismatch**: Mixing `apt`-installed KiCad (`libpython3.14` + system `wxWidgets`) with a Conda Python binary causes ABI and symbol resolution failures; our native `--system-site-packages` `.venv` avoids this cleanly.
* **The Schematic Readability Gap**: Relying solely on `SKiDL` netlists and raw `kiutils` S-expression edits produces functional netlists but messy, unreadable schematic diagrams.
* **The JLCPCB SMT Rotation Trap**: Default KiCad footprint zero-rotation definitions frequently differ by 90 degrees or 180 degrees from JLCPCB SMT tape orientations, risking rotated QFN/SOT-23/diode assembly if raw `kicad-cli` position files are uploaded uncorrected.

### 4. Root Causes & Breakthrough Moments
* **Ecosystem Synthesis**: Adding `kicad-sch-api` / `circuit-synth` solves schematic readability with byte-preserving orthogonal routing, while adopting `KiBot` automates JLCPCB BOM column mapping and `JLC-rotations` pick-and-place correction in a single YAML pass.
* **Tier Storage Insight**: Breaking down installation footprints showed that `easyeda2kicad`, `KiBot`, and `Freerouting` total under 220 MB combined—nearly 85% of the ~5.75 GB Full Autonomous footprint comes solely from `kicad-packages3d` (~4.8 GB unpacked).

### 5. Daily Verdict & Key Metrics
* **Status**: Stage 0 (Ecosystem Research & Repository Scaffolding) completed; zero host packages installed yet (`Stage 1` next).
* **Artifacts Completed**: 3 Nord-formatted research blueprints ([kicad_setup_guide.md](research/kicad_setup_guide.md), [ai_pcb_design_playbook.md](research/ai_pcb_design_playbook.md), [open_source_ai_pcb_comparison.md](research/open_source_ai_pcb_comparison.md)), root [readme.md](../readme.md), `.gitignore`, and immutable project diary.
* **Target Testbed Roadmap**: `toolchain_setup` -> `pcb_breadboard` -> `led_array` -> `buck_12v_to_5v`.
