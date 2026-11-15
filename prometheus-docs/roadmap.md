# Roadmap and 3-Developer Plan

The report's phases (Foundation → Senses → Brain → Survival → Memory → Validation) are sequential. To parallelize, we split by **module ownership** and freeze interfaces first.

Recommended build order: prove a Windows app can run a bounded simulation with a clean C++/QML boundary; then add telemetry, state, resource control and persistence. **Do not start with ML or evolution.**

## Roles

| | Dev A: Senses & Memory | Dev B: Brain & Metabolism | Dev C: Body, UI & Runtime |
|---|---|---|---|
| Owns | `sensory/`, `platform/windows/`, `memory/`, benchmarks | `brain/`, `metabolism/` | `app/`, `core/`, `body/`, `ui/`, CMake/Qt |
| Role | Provides the evidence | Holds the core idea | Critical path and integrator |

## Phase 0: Contracts and skeleton (all, together) · HARD GATE

- [ ] Freeze contracts in [interfaces.md](interfaces.md): `OrganismState`, `Behaviour`, `QualityTier`, `OrganismSnapshot`, `HardLimits`
- [ ] Resolve open decisions in [decisions.md](decisions.md) (Behaviour naming, pressure ownership, error reporting, SQLite approach, Qt version)
- [ ] Cross-cutting rules: time units, error reporting, snapshot thread-safety
- [ ] **Dev C:** CMake + Qt skeleton, one empty library per module
- [ ] **Dev A / B:** stub implementations (hardcoded values) so everything compiles
- [ ] GoogleTest, CI build, branch conventions
- [ ] **Verify Kiroween rules** ([hackathon-checklist.md](hackathon-checklist.md))

**Exit:** empty project builds on all three machines; interface headers merged.

## Phase 1: Foundations (parallel)

| Dev A | Dev B | Dev C |
|---|---|---|
| [ ] `CpuMonitor` (PDH or system-time deltas) | [ ] `OrganismState` update equations | [ ] Transparent frameless Qt window + basic visual |
| [ ] `MemoryMonitor` (`GlobalMemoryStatusEx`) | [ ] EMA filter + pressure estimator (renormalize missing) | [ ] Fixed-capacity `ParticleSystem` |
| [ ] `HostObservation` with validity + timestamps | [ ] Unit tests: bounds, determinism | [ ] Separate simulation and render clocks |
| [ ] **Synthetic telemetry** (deliver first; unblocks B) | | [ ] Narrow `ViewModel` (read-only `Q_PROPERTY`) |

**Exit:** real and synthetic observations validated; organism renders and simulates decoupled from UI; Brain unit tests pass.

## Phase 2: Core survival logic (parallel)

| Dev A | Dev B | Dev C |
|---|---|---|
| [ ] Monitoring worker + mutex-protected latest snapshot | [ ] Behaviour state machine with hysteresis + dwell | [ ] `SimulationCoordinator` + `RuntimeScheduler` |
| [ ] Stale-data detection + failure fallbacks | [ ] `ComputeBudget` + tiers 0–4 | [ ] Body obeys `ComputeBudget` |
| [ ] Versioned SQLite schema + `MemoryManager` | [ ] Degrade-fast / recover-slow logic | [ ] Immutable snapshot publishing to UI |
| | [ ] Hard safety limits | |

**Exit:** under synthetic pressure, behaviour changes and particle count and frame rate visibly drop.

## Phase 3: Integration (joint sprint)

- [ ] Dev C wires real monitor + Brain/Metabolism through the coordinator
- [ ] **Dormancy:** B defines entry/exit; C makes it genuinely cheap; A reduces monitoring frequency while dormant
- [ ] **Persistence hooks:** save on dormancy entry, transitions, periodic, clean shutdown; restore on startup (A + C)
- [ ] Integration tests: RAM spike → Conserve → budget drops → Body respects limits → state restores

**Exit:** healthy → dormant → recovered works end to end; identity survives restart.

## Phase 4: Validation and demo (parallel)

| Dev A | Dev B | Dev C |
|---|---|---|
| [ ] Benchmark harness (process CPU, WS, commit, dormant CPU, recovery latency) | [ ] Tune thresholds/tiers from measurements | [ ] Observer/developer panel |
| [ ] Controlled stress workload with safe stop | [ ] Fixed-complexity mode for baseline | [ ] Survival Test demo flow |
| [ ] Fault-injection tests | [ ] Boundary + regression tests | [ ] Polish + recorded backup demo |
| *Stretch:* NVML GPU telemetry | *Stretch:* adaptation statistics | *Stretch:* `ShaderEffect` visuals |

**Exit:** adaptive vs fixed benchmark results on a documented machine; every demo claim backed by a measurement.

## Sync points

- End of Phase 0: contract review (hard gate).
- End of each phase: 30-minute integration check; everyone merges and runs the full app.
- Mid-phase: shared-header changes need both other devs' approval.

## Priority order (from the report)

1. CMake C++20 + Qt 6/QML foundation; transparent window shows a basic organism
2. Bounded simulation + independent render loop
3. CPU/RAM observation adapter + synthetic mode
4. Deterministic organism state + compute-budget controller
5. Persistence + Survival Test benchmark

## Deferred (post-MVP)

GPU telemetry · genetic evolution · reinforcement learning · LLM personality (excluded from core) · complex 3D rendering · multi-organism ecosystem.
