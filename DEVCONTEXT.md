# DEVCONTEXT

Onboarding and working agreement for the three developers on PROMETHEUS. AI agents should read [AGENTCONTEXT.md](AGENTCONTEXT.md) as well.

## 1. Read these first (about 30 minutes)

1. [NORTHSTAR.md](NORTHSTAR.md): why we are building this and what "done" means
2. [docs/architecture.md](docs/architecture.md): how the pieces fit
3. [docs/interfaces.md](docs/interfaces.md): the contracts we share
4. [docs/roadmap.md](docs/roadmap.md): phases and who does what

## 2. Roles

| | Dev A: Senses & Memory | Dev B: Brain & Metabolism | Dev C: Body, UI & Runtime |
|---|---|---|---|
| Owns | `sensory/`, `platform/windows/`, `memory/`, benchmark tooling | `brain/`, `metabolism/` | `app/`, `core/`, `body/`, `ui/`, CMake/Qt |
| Work type | Windows APIs, SQLite, profiling | Pure C++ logic, determinism | Qt/QML, rendering, threading, integration |
| Test style | Fault injection, process measurement | Unit tests, property tests | Integration tests, visual checks |
| First deliverable | **Synthetic telemetry** (unblocks Dev B) | State equations + EMA + pressure | Skeleton + transparent window |

Dev C is the **integration lead** and runs the end-of-phase merge check. Ownership means *primary responsibility*, not exclusivity; cross-module PRs need the owner's review.

## 3. Environment setup (Windows)

> Exact versions are pinned in Phase 0. The list below is the intended baseline.

| Tool | Notes |
|---|---|
| Windows 10/11 x64 | Primary target. Record your version for benchmark notes. |
| Visual Studio 2022 Build Tools (MSVC, C++20) | Or full VS 2022 |
| Windows SDK | PDH, memory/power APIs, desktop integration |
| CMake (recent 3.x) + Ninja | Presets in `CMakePresets.json` |
| Qt 6 (LTS), with Qt Quick / QML / Quick Controls | Install only the modules we use |
| SQLite | Source or package manager; decided in Phase 0 |
| GoogleTest | Via package manager or FetchContent |
| Git | Short-lived feature branches |
| Optional: Python 3 | Offline experiments only (`experiments/`) |
| Optional: NVIDIA driver with NVML | Only if testing GPU telemetry |

Measurement tools (Dev A, but everyone should know them): Windows Performance Monitor, Process Explorer, Windows Performance Recorder.

### Verify your setup

```powershell
cmake --preset windows-msvc-debug      # configure
cmake --build --preset windows-msvc-debug
ctest --preset windows-msvc-debug      # all tests green
.\build\windows-msvc-debug\src\app\prometheus.exe --synthetic   # (planned flag)
```

These work once the Phase 0 skeleton merges.

## 4. Workflow

- `main` always builds and passes tests. Never push directly.
- Branches: `feat/<module>-<short-name>`, `fix/...`, `docs/...`. Keep them short-lived (days, not weeks).
- Commits: imperative, scoped, e.g. `brain: add EMA pressure filter`.
- PRs: small, one concern, linked to a roadmap item; at least one review. Use the checklist in [CONTRIBUTING.md](CONTRIBUTING.md).
- **Shared headers** (`HostObservation`, `OrganismState`, `ComputeBudget`, `OrganismSnapshot`, `Behaviour`, `QualityTier`): any change needs approval from **both other devs**. See [docs/interfaces.md](docs/interfaces.md).
- End of each phase: 30-minute integration check; everyone merges and runs the full app.

## 5. Day-to-day rules of thumb

- Keep `main` runnable with stubs; replace stubs with real implementations incrementally.
- Develop against **synthetic telemetry** first; wire real sensors after.
- Write the test that proves your module's causal claim *before* tuning numbers.
- Every tuned constant gets a comment: "initial, to be tuned" until a measurement backs it.
- If you are blocked on another dev's module, code against the interface and a stub; do not wait.

## 6. Known pitfalls

| Area | Pitfall |
|---|---|
| CPU telemetry | Utilization needs an **interval** (two samples); a single read is meaningless. PDH counters need a priming query. |
| Missing data | A failed/absent sensor must be `valid=false`, never `0`. Zero reads as "no pressure". |
| GPU | NVML may be absent; the app must work fully without it. |
| Qt threading | QObjects belong to one thread. Cross-thread updates via queued signals or immutable snapshots. |
| Click-through | Hit-testing is subtle; do not make the whole window permanently click-through. |
| Rendering | A frame must not trigger a Brain/sim update. |
| Persistence | No DB work on the GUI/render path; use transactions and atomic checkpoints. |
| Benchmarking | Process CPU ≠ total system CPU. Normalize by logical cores; record the machine. |
| Oscillation | Without hysteresis and dwell times, states flap on noisy samples. |
| Controller cost | The controller must be cheaper than what it controls. Avoid per-tick allocation. |

## 7. Debugging aids (to build early)

- **Synthetic telemetry mode** with scripted pressure curves (ramp, spike, plateau, stale, drop-out).
- **Observer/dev panel** showing measured host values separately from simulated organism variables.
- Rate-limited structured logs of state transitions and budget changes (with reason).
- Deterministic replay: feed a recorded observation sequence + time deltas and assert identical decisions.

## 8. Status board

Update this table in the PRs that change it.

| Area | Owner | Status | Notes |
|---|---|---|---|
| Phase 0 contracts + skeleton | All / C | Not started | Hard gate |
| Hackathon rules verified | All | Not started | See checklist |
| Synthetic telemetry | A | Not started | Unblocks B |
| CPU/RAM monitors | A | Not started | |
| State equations + pressure | B | Not started | |
| State machine + hysteresis | B | Not started | |
| ComputeBudget + tiers | B | Not started | |
| Window + organism visual | C | Not started | |
| Particle pool + clocks | C | Not started | |
| Coordinator + snapshots | C | Not started | |
| SQLite persistence | A | Not started | |
| Dormancy end-to-end | All | Not started | Phase 3 |
| Benchmark harness | A | Not started | Phase 4 |
| Observer panel + demo flow | C | Not started | Phase 4 |

## 9. Getting unstuck

- Contract questions → [docs/interfaces.md](docs/interfaces.md), then raise in the contract review.
- "Why this threshold?" → [docs/organism-model.md](docs/organism-model.md); if unjustified, mark it `untuned`.
- "How do we prove this claim?" → [docs/benchmark-methodology.md](docs/benchmark-methodology.md).
- Scope creep → [NORTHSTAR.md](NORTHSTAR.md) decision filter.
