# PROMETHEUS MVP – Design

> Status: **draft**. Corresponds to the requirements in `requirements.md`.  
> Full subsystem detail lives in `prometheus-docs/architecture.md` and `prometheus-docs/interfaces.md`.  
> This document describes *how* the requirements will be satisfied, not what the requirements are.

---

## 1. The central design decision

The organism's computational footprint is **explicitly budgeted**, not emergent. `Metabolism` converts `Behaviour` + host pressure into a hard-capped `ComputeBudget`. `Body` obeys that budget. The UI reads an immutable snapshot. No path from rendering back to simulation exists.

This one decision satisfies REQ-BUDGET-01, REQ-BUDGET-02, REQ-BODY-02, and REQ-UI-01 simultaneously and is the reason the architecture has the shape it does.

---

## 2. Module interaction flow

```
Sensor worker  ──► HostObservation (mutex-protected latest)
                          │
                          ▼
              Simulation thread (≈1 Hz decision loop)
                    │
                    ├─► Brain::update(obs, prev, dt)
                    │        └─► OrganismState  (energy, stress, curiosity,
                    │                            smoothedPressure, Behaviour)
                    │
                    ├─► Metabolism::calculate(obs, state)
                    │        └─► ComputeBudget  (clamped to HardLimits)
                    │
                    ├─► Body::update(budget, dt)   [inner loop ≈10–30 Hz]
                    │        └─► particle pool, bounded by budget.particleLimit
                    │
                    ├─► publish OrganismSnapshot  (shared_ptr<const>)
                    │
                    └─► MemoryManager::maybeSaveCheckpoint(state, trigger)

GUI thread  ◄── queued signal ── OrganismSnapshot
    │
    └─► QML ViewModel (read-only Q_PROPERTYs, no mutable sim refs)
```

---

## 3. Sensory subsystem (REQ-SENSE-01 – REQ-SENSE-04)

**CpuMonitor** wraps PDH. It primes the counter on construction, then returns `Metric{fraction, valid}` from an interval between two samples. The first call always returns `valid=false` (no interval yet).

**MemoryMonitor** wraps `GlobalMemoryStatusEx`, converting raw bytes to a `[0,1]` fraction and an absolute `availableRamMB`.

**SyntheticProvider** implements the same `SensoryProvider` interface and replays a scripted curve from a config file or hard-coded profile. Every `HostObservation` it emits sets `source = TelemetrySource::Synthetic`.

**Staleness detection:** the sensor worker stamps each observation with `SteadyTime sampledAt`. The simulation thread checks age before use; observations older than `config.stalenessThresholdMs` are treated as `valid=false` for all metrics.

No sensor type is ever substituted with zero. `CpuMonitor` and `MemoryMonitor` return `valid=false` on API failure.

---

## 4. Brain (REQ-BRAIN-01 – REQ-BRAIN-04)

`Brain::update` is a free function (or stateless class): all inputs arrive as parameters; the previous `OrganismState` is passed in and a new one is returned. No static state, no globals, no I/O.

**Pressure EMA:**
```
rise_α  = config.pressureRiseAlpha   // larger → reacts faster
fall_α  = config.pressureFallAlpha   // smaller → recovers slower
delta   = rawPressure - prev.smoothedPressure
α       = (delta > 0) ? rise_α : fall_α
smoothedPressure = prev.smoothedPressure + α * delta
```
Raw pressure is computed only over `valid` signals; weights renormalise over the valid set.

**Behaviour selection** is a priority chain (see REQ-BRAIN-03). Transition guard: a candidate behaviour is only adopted if the trigger has held for `config.debounceTicks` consecutive intervals AND `secondsInBehaviour >= config.minDwellSeconds` for the current behaviour. These two guards satisfy REQ-BRAIN-04.

**Energy / stress equations** follow `prometheus-docs/organism-model.md` §3. `Brain` owns the update; `Metabolism` reads `smoothedPressure` and `Behaviour` from the returned state.

---

## 5. Metabolism (REQ-BUDGET-01 – REQ-BUDGET-03)

`Metabolism::calculate` maps `(Behaviour, smoothedPressure)` to a `QualityTier` via a config table, then looks up the `ComputeBudget` for that tier from a second config table. The final step clamps every field against `HardLimits` — this is always last, unconditional.

**Tier table (illustrative, untuned):**

| Behaviour | Pressure range | Tier |
|---|---|---|
| Dormant | any | Dormant (0) |
| Conserve | > 0.70 | Survival (1) |
| Conserve | 0.50–0.70 | Reduced (2) |
| Recover | any | Reduced (2) |
| Idle | < 0.40 | Normal (3) |
| Explore | < 0.30 | Thriving (4) |

**Asymmetric pacing (REQ-BUDGET-03):** `Metabolism` tracks `currentTier`. Downgrade is immediate. Upgrade only increments `currentTier` by one per call, subject to a dwell timer `config.tierRestoreIntervalSeconds`.

---

## 6. Body (REQ-BODY-01 – REQ-BODY-02)

**Particle pool:** a `std::array<Particle, kMaxParticles>` allocated once at startup. An `activeCount` integer tracks how many are live. `update()` only processes `[0, activeCount)`. Activating a particle sets its index; deactivating swaps it with the last active entry and decrements `activeCount`.

The simulation thread calls `Body::update(budget, dt)` on its own `QTimer` or loop at ≈10–30 Hz while `QualityTier > Dormant`. The GUI thread never calls into `Body`. A rendered frame reads only the published `OrganismSnapshot`.

---

## 7. Dormancy (REQ-DORMANCY-01)

Dormancy is a lifecycle state managed by the **Runtime Coordinator** in `src/core/`. On entry:

1. `MemoryManager::saveCheckpoint(state, Trigger::DormancyEntry)` — blocking, on the persistence worker.
2. Simulation thread parks on a `std::condition_variable` with a low-frequency wake timeout (e.g. 10 s).
3. Sensor worker reduces its sampling interval to `config.dormantSampleIntervalMs`.
4. Rendering switches to event-driven (no timer-driven frame production).

Wake condition: `smoothedPressure < config.wakeThreshold` sustained for `config.wakeDwellSeconds`, evaluated in the low-frequency heartbeat.

On wake, `MemoryManager::loadCheckpoint()` is called and the restored state is injected into the first `Brain::update` call of the new active period.

---

## 8. Memory / persistence (REQ-MEMORY-01 – REQ-MEMORY-02)

`MemoryManager` owns the SQLite connection. All DB work happens on a dedicated **persistence worker thread** that receives requests via a bounded `std::queue` (capacity: `config.persistenceQueueDepth`). The simulation thread enqueues a `CheckpointRequest`; it never waits for the write to complete.

Schema version is stored in `schema_version`. Migrations are handled at startup before the simulation starts. Corrupt or incomplete checkpoints fall through to `initNewOrganism()`.

Atomic checkpoint replacement: write to a `checkpoints_tmp` row, then rename/update within a transaction.

---

## 9. UI / ViewModel (REQ-UI-01 – REQ-UI-02)

`OrganismViewModel : QObject` holds a `std::shared_ptr<const OrganismSnapshot>` updated via a queued signal from the simulation thread. All `Q_PROPERTY` values are read from that snapshot; there are no setters that modify simulation state.

Synthetic telemetry indicator: a `Q_PROPERTY bool syntheticMode` derived from `snapshot->telemetrySource`. QML binds a visible `Text { text: "SYNTHETIC" }` element to this property.

---

## 10. Threading summary

| Thread | Created by | Communicates via |
|---|---|---|
| GUI (Qt main) | Qt framework | Queued signals from simulation thread |
| Sensor worker | `SensorWorker::start()` in Core | Mutex-protected `HostObservation` latest-value slot |
| Simulation | `SimulationThread::start()` in Core | Publishes `shared_ptr<const OrganismSnapshot>` via queued signal; enqueues `CheckpointRequest` |
| Persistence worker | `PersistenceWorker::start()` in Core | Bounded queue from simulation thread |

No `QObject` crosses thread boundaries. `OrganismSnapshot` is immutable after publication.

---

## 11. Observer / developer panel (Phase 4)

A second QML view (toggled by a keyboard shortcut) shows:
- Measured host values (`HostObservation`) separately from simulated organism variables (`OrganismState`).
- Current `QualityTier`, `ComputeBudget` fields, `activeParticles`, `measuredSimHz`, `measuredRenderFps`.
- Recent behaviour transitions with timestamps and trigger reasons.
- A `SYNTHETIC` badge when running synthetic telemetry.

This panel reads the same `OrganismSnapshot` as the main view. It adds no simulation logic.

---

## 12. What this design deliberately excludes

- No ML, reinforcement learning, or LLM calls in any module.
- No network I/O.
- No GPU telemetry in MVP (NVML is optional and isolated behind a compile flag).
- No genetic evolution, reproduction, or multi-organism coordination.
- No third-party dependencies beyond the declared stack (CMake, Qt 6, SQLite, GoogleTest, Windows SDK).
