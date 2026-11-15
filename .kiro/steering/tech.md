# PROMETHEUS – Technical Steering

## Stack

| Layer | Technology |
|---|---|
| Language | C++20 (MSVC, `/std:c++20`) |
| Build | CMake 3.x with `CMakePresets.json`; Ninja generator |
| UI | Qt 6 LTS — Qt Quick / QML; thin ViewModel only |
| Persistence | SQLite (source or package manager; decided in Phase 0) |
| Testing | GoogleTest (via CMake FetchContent or vcpkg) |
| Platform APIs | Windows PDH (CPU), `GlobalMemoryStatusEx` (RAM), optional NVML (GPU) |
| Optional research | Python 3 in `experiments/` only — never in the core build |

## Module dependency rule

```
app/ui  →  core  →  {sensory, brain, metabolism, body, memory}
```

`brain/` and `metabolism/` depend **only** on the C++ standard library and shared types (`src/core/`). No Qt headers, no Windows headers, no PDH in `brain/` or `metabolism/`. Platform code belongs exclusively in `src/platform/windows/` behind a portable `SensoryProvider` interface.

## Code conventions

- **Types:** `PascalCase`. **Functions/members:** `camelCase`. **Constants:** `kPascalCase`. **Files:** match the primary type name.
- **Time:** `std::chrono::steady_clock` for all timestamps and intervals. `float` seconds for `deltaTime`. Never wall-clock time for control logic.
- **Errors:** no exceptions across module boundaries in the hot loop. Return explicit result types or validity flags (`Metric.valid`). Missing telemetry is **never** substituted with zero.
- **Config over magic numbers:** every threshold, weight, tier limit, and hard cap lives in a named config struct with documented defaults. Inline magic numbers block review.
- **Logging:** rate-limited. Sensor failures must not flood the log. Transitions and budget changes are logged with a reason code.
- **Comments:** explain *why*, especially for thresholds and hysteresis. Mark every tuned constant `// initial, to be tuned`.
- **Warnings as errors** (`/W4 /WX` on MSVC) on all new code once the Phase 0 skeleton lands.

## Shared contracts (frozen after Phase 0)

The following types are shared across modules and require both other devs' approval to change:  
`HostObservation`, `OrganismState`, `Behaviour`, `ComputeBudget`, `QualityTier`, `HardLimits`, `OrganismSnapshot`, `kContractVersion`.

See `docs/interfaces.md` for definitions and the change-control protocol.

## Threading model

| Thread | Owns |
|---|---|
| GUI thread | Qt objects, QML scene, ViewModel — never blocks on I/O, DB, or simulation |
| Sensor worker | Sampling loop; mutex-protected latest `HostObservation` |
| Simulation thread | Brain, Metabolism, Body, Coordinator tick; publishes immutable `OrganismSnapshot` |
| Persistence worker | SQLite work via a bounded request queue |

Snapshots cross threads via `std::shared_ptr<const OrganismSnapshot>` and queued signals or atomic swap. Live simulation objects never cross thread boundaries.

## Hard rules (never violate)

1. `ComputeBudget` never exceeds `HardLimits` — no heuristic, model, or config override can bypass this.
2. Dormancy stops normal simulation updates and expensive rendering. Hiding the window is not dormancy.
3. Rendering never triggers a Brain or simulation update.
4. Brain and Metabolism are pure: same inputs + same `deltaTime` = same output. No hidden state, no random calls outside an injected seeded RNG.
5. No unbounded containers in hot paths: fixed-capacity particle pool, capped queues, bounded histories.
6. No new external dependencies (beyond the declared stack) without a decision logged in `docs/decisions.md`.
7. Never fabricate benchmark numbers, RAM/CPU figures, or test outcomes. Synthetic telemetry is always labelled as synthetic.

## Build commands (once Phase 0 skeleton lands)

```powershell
cmake --preset windows-msvc-debug
cmake --build --preset windows-msvc-debug
ctest --preset windows-msvc-debug
.\build\windows-msvc-debug\src\app\prometheus.exe --synthetic
```
