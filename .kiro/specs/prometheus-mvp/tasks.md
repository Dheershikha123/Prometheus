# PROMETHEUS MVP – Tasks

> Generated from `requirements.md` and `design.md`.  
> Work through phases in order. Phase 0 is a hard gate — nothing else starts until it exits.  
> Each task lists its requirements, the files it touches, and its exit criteria.

---

## Phase 0 · Contracts and skeleton (all three devs, together)

### TASK-0.1 · Freeze shared contracts
- **Requirements:** all REQ-* (all modules consume shared types)
- **Files:** `prometheus-docs/interfaces.md`, `src/core/Types.h`, `src/core/OrganismSnapshot.h`, `src/sensory/HostObservation.h`, `src/brain/OrganismState.h`, `src/brain/Behaviour.h`, `src/metabolism/ComputeBudget.h`
- **Work:** ratify every type in `interfaces.md` §2; resolve the Behaviour naming decision (`docs/decisions.md`); set `kContractVersion = 1`; create header files with the ratified definitions.
- **Exit:** all three devs have approved the PR; headers compile with `/std:c++20 /W4 /WX`.

### TASK-0.2 · CMake skeleton and empty libraries
- **Requirements:** build foundation for all modules
- **Owner:** Dev C
- **Files:** `CMakeLists.txt`, `CMakePresets.json`, `src/*/CMakeLists.txt`
- **Work:** one static library target per module (`sensory`, `brain`, `metabolism`, `body`, `memory`, `core`, `ui`, `app`); stub `.cpp` files that compile; `app` target links them all; GoogleTest via `FetchContent`.
- **Exit:** `cmake --preset windows-msvc-debug && cmake --build --preset windows-msvc-debug` exits clean on all three machines; `ctest` runs with zero tests (pass by vacuous truth).

### TASK-0.3 · Stub implementations
- **Requirements:** unblocks parallel Phase 1 work
- **Owners:** Dev A (sensory stubs), Dev B (brain/metabolism stubs)
- **Files:** `src/sensory/SyntheticProvider.h/.cpp`, `src/brain/Brain.h/.cpp`, `src/metabolism/Metabolism.h/.cpp`
- **Work:** stubs return hardcoded valid values (e.g. `cpuUsage={0.1f, true}`). Brain returns `previous` unchanged. Metabolism returns a `Normal` tier budget.
- **Exit:** full project still builds and links with stubs in place.

### TASK-0.4 · Verify Kiroween rules
- **Files:** `prometheus-docs/hackathon-checklist.md`
- **Work:** fill in every TBD row from the official Kiroween 2026 rules page; update `prometheus-docs/roadmap.md` if deadline or required artefacts change.
- **Exit:** checklist has no TBD entries; team has agreed on submission format.

---

## Phase 1 · Foundations (parallel)

### TASK-1.1 · CpuMonitor (Dev A)
- **Requirements:** REQ-SENSE-01
- **Files:** `src/platform/windows/CpuMonitor.h/.cpp`, `tests/sensory/CpuMonitorTest.cpp`
- **Work:** PDH counter init; interval-based sampling; returns `Metric{value, valid}`; first call returns `valid=false`.
- **Tests:** valid range `[0,1]`; first-call invalid; API failure → invalid; never returns zero for missing.
- **Exit:** tests pass; no Windows headers leak into `src/sensory/` or `src/brain/`.

### TASK-1.2 · MemoryMonitor (Dev A)
- **Requirements:** REQ-SENSE-02
- **Files:** `src/platform/windows/MemoryMonitor.h/.cpp`, `tests/sensory/MemoryMonitorTest.cpp`
- **Work:** `GlobalMemoryStatusEx` wrapper; produces `ramUsageFraction` and `availableRamMB`; API failure → both invalid.
- **Tests:** valid range `[0,1]`; API-failure path returns `valid=false` for both metrics.
- **Exit:** tests pass.

### TASK-1.3 · SyntheticProvider (Dev A) — delivers first, unblocks Dev B
- **Requirements:** REQ-SENSE-03
- **Files:** `src/sensory/SyntheticProvider.h/.cpp`, `tests/sensory/SyntheticProviderTest.cpp`
- **Work:** implements `SensoryProvider`; replays a scripted curve (ramp, spike, plateau, stale, drop-out); labels all observations `TelemetrySource::Synthetic`.
- **Tests:** output matches the scripted curve; `source == Synthetic` on every observation; stale profile produces `valid=false` metrics.
- **Exit:** tests pass; Dev B can use this to test Brain without real hardware.

### TASK-1.4 · Brain state equations (Dev B)
- **Requirements:** REQ-BRAIN-01, REQ-BRAIN-02
- **Files:** `src/brain/Brain.h/.cpp`, `tests/brain/BrainTest.cpp`
- **Work:** energy/stress update equations from `organism-model.md` §3; asymmetric EMA for `smoothedPressure`; renormalise over valid signals.
- **Tests:** determinism (same inputs → same output × 100); energy stays in `[0,100]`; stress stays in `[0,100]`; EMA rises faster than it falls; missing signal (valid=false) never treated as zero pressure.
- **Exit:** tests pass with no Qt or Windows headers included.

### TASK-1.5 · Behaviour state machine (Dev B)
- **Requirements:** REQ-BRAIN-03, REQ-BRAIN-04
- **Files:** `src/brain/Brain.h/.cpp` (extend), `tests/brain/BehaviourTest.cpp`
- **Work:** priority-chain policy; hysteresis thresholds from config; dwell-time guard; debounce guard.
- **Tests:** critical-low energy → Dormant regardless of pressure; enter-Conserve threshold triggers Conserve; not exited before dwell; debounce prevents single-tick flip; all five behaviours reachable.
- **Exit:** tests pass; no oscillation under a noisy synthetic ramp.

### TASK-1.6 · Transparent Qt window + basic visual (Dev C)
- **Requirements:** REQ-UI-01 (foundation)
- **Files:** `src/app/main.cpp`, `src/ui/OrganismViewModel.h/.cpp`, `resources/qml/Main.qml`
- **Work:** frameless, transparent window; a placeholder visual (e.g. a coloured circle whose radius maps to `activeParticles`); `OrganismViewModel` with read-only `Q_PROPERTY`s fed by a hardcoded snapshot.
- **Exit:** window appears on a Windows desktop; no crash; placeholder visual visible; no simulation logic in QML or ViewModel.

### TASK-1.7 · Fixed-capacity ParticleSystem (Dev C)
- **Requirements:** REQ-BODY-01
- **Files:** `src/body/ParticleSystem.h/.cpp`, `tests/body/ParticleSystemTest.cpp`
- **Work:** `std::array<Particle, kMaxParticles>`; `activate()` / `deactivate()` using swap-with-last; `update(budget, dt)` processes only active particles.
- **Tests:** capacity never exceeded; active count matches budget limit after a `setLimit()` call; no per-call allocation (verify with a custom allocator or static assertion).
- **Exit:** tests pass.

### TASK-1.8 · Independent simulation and render clocks (Dev C)
- **Requirements:** REQ-BODY-02
- **Files:** `src/core/SimulationThread.h/.cpp`, `src/core/RuntimeScheduler.h/.cpp`
- **Work:** simulation thread runs Brain + Metabolism + Body on its own `QTimer` (or `std::thread` + `steady_clock`); GUI `QQuickWindow::frameSwapped` does not drive any simulation step.
- **Exit:** rendering at a forced low FPS does not slow the simulation tick rate (verified by logging timestamps).

---

## Phase 2 · Core survival logic (parallel)

### TASK-2.1 · Sensor worker + staleness detection (Dev A)
- **Requirements:** REQ-SENSE-04
- **Files:** `src/sensory/SensorWorker.h/.cpp`, `tests/sensory/SensorWorkerTest.cpp`
- **Work:** background thread sampling at `config.sensorIntervalMs`; mutex-protected latest `HostObservation`; staleness check in the simulation thread before each `Brain::update` call.
- **Tests:** stale observation (age > threshold) produces `valid=false` for affected metrics; conservative fallback applied (smoothedPressure does not drop when data is stale).
- **Exit:** tests pass; sensor thread never crashes on API failure.

### TASK-2.2 · ComputeBudget + quality tiers (Dev B)
- **Requirements:** REQ-BUDGET-01, REQ-BUDGET-02, REQ-BUDGET-03
- **Files:** `src/metabolism/Metabolism.h/.cpp`, `src/metabolism/BudgetConfig.h`, `tests/metabolism/MetabolismTest.cpp`
- **Work:** tier table and budget table in `BudgetConfig`; asymmetric tier pacing; hard-limit clamp as the last operation.
- **Tests:** every output field ≤ corresponding `HardLimits` field (property test over randomised inputs); downgrade immediate; upgrade by one tier per dwell; `Dormant` behaviour → tier 0 always.
- **Exit:** tests pass; no magic numbers in `Metabolism.cpp`.

### TASK-2.3 · Body obeys ComputeBudget (Dev C)
- **Requirements:** REQ-BODY-01 (enforcement path)
- **Files:** `src/body/ParticleSystem.cpp` (extend TASK-1.7), `src/core/SimulationThread.cpp`
- **Work:** simulation thread passes `budget` to `Body::update`; `ParticleSystem` deactivates excess particles in the same tick; `measuredSimHz` tracked and included in the snapshot.
- **Tests:** particle count never exceeds `budget.particleLimit` in any tick; `measuredSimHz` within 10% of `budget.simulationFrequencyHz` under steady load.
- **Exit:** tests pass; no allocation in the update hot path.

### TASK-2.4 · Versioned SQLite schema + MemoryManager (Dev A)
- **Requirements:** REQ-MEMORY-01, REQ-MEMORY-02
- **Files:** `src/memory/MemoryManager.h/.cpp`, `src/memory/Schema.sql`, `tests/memory/MemoryManagerTest.cpp`
- **Work:** schema from `architecture.md` §9; `schema_version` table; migration path; `saveCheckpoint` and `loadCheckpoint`; persistence worker thread with bounded queue; atomic replacement via transaction.
- **Tests:** save + load round-trip preserves all `OrganismState` fields within float tolerance; corrupt DB falls through to `initNewOrganism`; `age` is monotonic across restarts; DB work never called from the GUI thread (enforced by thread-id assert in debug builds).
- **Exit:** tests pass; no DB call path reachable from the GUI thread.

### TASK-2.5 · Immutable snapshot publishing (Dev C)
- **Requirements:** REQ-UI-01
- **Files:** `src/core/RuntimeCoordinator.h/.cpp`, `src/ui/OrganismViewModel.cpp`
- **Work:** simulation thread creates `std::make_shared<const OrganismSnapshot>` after each tick; emits a queued signal; ViewModel receives it and updates `Q_PROPERTY`s.
- **Tests:** ViewModel never holds a non-const pointer to any simulation object (static_assert or code review gate); snapshot sequence number is monotonically increasing.
- **Exit:** integration test: snapshot received by ViewModel within two tick periods of publication.

---

## Phase 3 · Integration (joint sprint)

### TASK-3.1 · Wire real sensors through Coordinator (Dev C + Dev A)
- **Files:** `src/core/RuntimeCoordinator.cpp`, `src/sensory/SensorWorker.cpp`
- **Work:** replace `SyntheticProvider` stub in the coordinator with real `CpuMonitor` + `MemoryMonitor`; `--synthetic` flag substitutes `SyntheticProvider` at startup.
- **Exit:** running with `--synthetic` and without produce consistent snapshot shapes; real host CPU/RAM values visible in the observer panel (Phase 4) or logged.

### TASK-3.2 · Dormancy end-to-end (Dev A + Dev B + Dev C)
- **Requirements:** REQ-DORMANCY-01
- **Files:** `src/core/RuntimeCoordinator.cpp`, `src/core/SimulationThread.cpp`, `src/sensory/SensorWorker.cpp`, `src/memory/MemoryManager.cpp`
- **Work:** entry path (checkpoint → park simulation → slow sensors → event-driven render); wake path (restore checkpoint → resume simulation); `dormant=true` in snapshot so UI can reflect the state.
- **Tests:** integration test: synthetic ramp to critical energy → `dormant=true` in snapshot; synthetic recovery → `dormant=false`; identity (`id`, `age`) identical before and after; normal simulation updates provably stopped during dormancy (tick counter frozen).
- **Exit:** healthy → dormant → recovered works end-to-end; identity survives restart.

### TASK-3.3 · Persistence hooks (Dev A + Dev C)
- **Requirements:** REQ-MEMORY-02
- **Files:** `src/core/RuntimeCoordinator.cpp`, `src/memory/MemoryManager.cpp`
- **Work:** save on dormancy entry/exit, behaviour transitions, periodic interval, clean shutdown.
- **Tests:** each trigger produces a checkpoint row with the correct `event_type`; shutdown test: kill the process cleanly and verify checkpoint row exists.
- **Exit:** tests pass; no checkpoint write occurs on the GUI thread.

### TASK-3.4 · Synthetic telemetry label in UI (Dev C)
- **Requirements:** REQ-UI-02
- **Files:** `src/ui/OrganismViewModel.h/.cpp`, `resources/qml/Main.qml`
- **Work:** `Q_PROPERTY bool syntheticMode` derived from `snapshot->telemetrySource`; QML `Text { visible: syntheticMode; text: "SYNTHETIC" }` overlay.
- **Exit:** overlay visible when running `--synthetic`; invisible in real-telemetry mode.

---

## Phase 4 · Validation and demo (parallel)

### TASK-4.1 · Benchmark harness (Dev A)
- **Requirements:** REQ-BENCH-01
- **Files:** `scripts/benchmark.ps1` (or `.py`), `scripts/BenchmarkConfig.json`
- **Work:** scripted stress workload (safe CPU spinner); process CPU (normalised by logical core count), private working set, committed memory sampled at 1 Hz; separate adaptive vs fixed-complexity runs; machine config recorded in output JSON.
- **Exit:** two output files produced on a documented machine; adaptive run shows lower average CPU than fixed-complexity run under the same synthetic pressure profile.

### TASK-4.2 · Fixed-complexity mode (Dev B)
- **Requirements:** REQ-BENCH-01 (baseline comparison)
- **Files:** `src/metabolism/Metabolism.h/.cpp`, `src/app/main.cpp`
- **Work:** `--fixed-tier N` flag bypasses `Metabolism::calculate` and returns a constant `ComputeBudget` for tier N.
- **Exit:** flag works; benchmark can run both modes in the same session.

### TASK-4.3 · Observer / developer panel (Dev C)
- **Files:** `resources/qml/ObserverPanel.qml`, `src/ui/OrganismViewModel.h/.cpp`
- **Work:** toggle panel (keyboard shortcut); shows raw `HostObservation` values, `OrganismState` variables, current tier, `ComputeBudget` fields, `activeParticles`, `measuredSimHz`, `measuredRenderFps`, last 20 behaviour transitions with reasons.
- **Exit:** all values update live; `SYNTHETIC` badge visible when applicable; panel adds no simulation logic.

### TASK-4.4 · Survival Test demo flow (Dev C)
- **Files:** `prometheus-docs/demo-script.md` (update), `scripts/demo-stress.ps1`
- **Work:** scripted stress tool that ramps CPU load safely and can be stopped cleanly; demo script narration aligned with confirmed submission criteria.
- **Exit:** demo script runs end-to-end without crashes; every claim in the narration backed by a visible measurement or logged value.

### TASK-4.5 · Fault-injection and regression tests (Dev A + Dev B)
- **Files:** `tests/sensory/FaultInjectionTest.cpp`, `tests/brain/RegressionTest.cpp`, `tests/metabolism/PropertyTest.cpp`
- **Work:** invalid/delayed/unavailable sensor readings; Brain determinism regression suite; Metabolism property test (budget ≤ limits over 10 000 randomised inputs).
- **Exit:** all tests pass; no test fabricates results or asserts on unmeasured values.
