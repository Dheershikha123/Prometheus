# PROMETHEUS – Repository Structure Steering

## Directory layout

```
d:\Prometheus\
├── .kiro/
│   ├── steering/           # Always-included agent/AI context
│   │   ├── product.md      # What PROMETHEUS is, the one proof, scope limits
│   │   ├── tech.md         # Stack, conventions, threading, hard rules
│   │   └── structure.md    # This file: layout and module map
│   ├── specs/
│   │   └── prometheus-mvp/ # Spec-driven development: requirements, design, tasks
│   │       ├── requirements.md
│   │       ├── design.md
│   │       └── tasks.md
│   └── hooks/
│       ├── contract-guard.json   # Warns when shared contract headers are edited
│       └── northstar-check.json  # Reminds agent to check NORTHSTAR before new work
├── CMakeLists.txt           # (Phase 0 — not yet present)
├── CMakePresets.json        # (Phase 0 — not yet present)
├── README.md
├── NORTHSTAR.md             # Thesis, success criteria, decision filter — settle arguments here
├── AGENTCONTEXT.md          # Rules for AI coding agents (source of truth; this .kiro mirrors it)
├── DEVCONTEXT.md            # Onboarding and workflow for human developers
├── CONTRIBUTING.md          # Branching, commits, PR checklist
├── prometheus-docs/         # Design documents
│   ├── architecture.md      # Subsystems, threading, update loop, dormancy, persistence
│   ├── interfaces.md        # Shared data contracts and change-control rules
│   ├── organism-model.md    # Variables, equations, state machine, adaptation
│   ├── benchmark-methodology.md
│   ├── testing-strategy.md
│   ├── roadmap.md           # Phases, 3-developer split, exit criteria
│   ├── demo-script.md       # The "Survival Test" demonstration
│   ├── hackathon-checklist.md
│   ├── risks.md
│   ├── decisions.md
│   └── glossary.md
├── src/                     # (Phase 0 — skeleton not yet present)
│   ├── core/                # Coordinator, scheduler, snapshot publication, shared types
│   ├── sensory/             # HostObservation collection and normalization
│   ├── platform/windows/    # Win32 PDH, GlobalMemoryStatusEx, optional NVML
│   ├── brain/               # OrganismState update, pressure estimate, Behaviour selection
│   ├── metabolism/          # ComputeBudget, QualityTier, hard limits, pacing
│   ├── body/                # Bounded particle simulation, runs within ComputeBudget
│   ├── memory/              # SQLite identity, events, checkpoints
│   ├── ui/                  # QML + thin ViewModel over immutable OrganismSnapshot
│   └── app/                 # Entry point, lifecycle
├── tests/                   # (Phase 0 — skeleton not yet present)
├── resources/               # QML assets, default config
├── scripts/                 # Build and benchmark helpers
└── experiments/             # Optional offline Python research (never in core build)
```

## Module ownership

| Module | Path(s) | Owner | Must NOT |
|---|---|---|---|
| Sensory | `src/sensory/`, `src/platform/windows/` | Dev A | Make decisions; render |
| Brain | `src/brain/` | Dev B | Include Qt or Windows headers; poll hardware |
| Metabolism | `src/metabolism/` | Dev B | Own visuals or persistence |
| Body | `src/body/` | Dev C | Read telemetry or touch DB |
| Memory | `src/memory/` | Dev A | Make real-time simulation decisions |
| Core | `src/core/` | Dev C | Contain biology rules or Win32 calls |
| UI | `src/ui/`, `src/app/` | Dev C | Hold mutable access to engine internals |

## Naming rules for new files

- File names match the primary C++ type they define (e.g. `OrganismState.h`, `Brain.cpp`).
- Test files live under `tests/` mirroring `src/` paths (e.g. `tests/brain/BrainTest.cpp`).
- Platform-specific files live exclusively under `src/platform/windows/`.
- No `Utils.h` catch-alls. Each file has one clearly named responsibility.

## What belongs in prometheus-docs/ vs .kiro/

`prometheus-docs/` contains **design documents** that all three developers own and edit collaboratively. They are the authoritative source for contracts, model, architecture, and methodology.

`.kiro/steering/` contains **agent-facing summaries** that distil those documents into always-available context for AI tools. When the two diverge, `prometheus-docs/` is correct; update the steering file to match.

`.kiro/specs/` contains **implementation specs** (requirements → design → tasks) used to drive Kiro's spec-driven workflow. They reference `prometheus-docs/` documents but do not duplicate them.
