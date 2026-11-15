# Decision Log

Lightweight ADR log. Add an entry whenever a choice affects a contract, dependency, thread, persistent format or public claim.

## Open decisions (resolve in Phase 0)

| # | Decision | Options | Proposed default | Owner | Status |
|---|---|---|---|---|---|
| D1 | `Behaviour` naming | Active/Idle/Stressed/Recovery/Dormant vs Explore/Idle/Conserve/Recover/Dormant | Explore/Idle/Conserve/Recover/Dormant, documented mapping to state-table names | B | Open |
| D2 | Where smoothed pressure lives | In `OrganismState` vs recomputed in Metabolism | In `OrganismState` (Brain owns it) | B | Open |
| D3 | Error reporting | Exceptions vs return values/validity flags vs result type | Validity flags for sensors; explicit result for I/O; no exceptions in hot loop | All | Open |
| D4 | Snapshot publication | `shared_ptr<const T>` + queued signal vs atomic swap | `shared_ptr<const T>` | C | Open |
| D5 | CPU telemetry source | PDH vs `GetSystemTimes` deltas | Pick by measured overhead and simplicity | A | Open |
| D6 | SQLite integration | Amalgamation vs vcpkg vs Qt SQL | Direct SQLite (amalgamation or package) | A | Open |
| D7 | Qt version pin | Qt 6 LTS release | Pin one LTS in Phase 0 | C | Open |
| D8 | Test framework / package manager | GoogleTest via FetchContent vs vcpkg | FetchContent or vcpkg, whichever is simpler on all 3 machines | C | Open |
| D9 | Health's causal role | Caps max tier vs defer entirely | Defer or cap max tier | B | Open |
| D10 | Simulation threading | Worker thread vs QTimer on a worker | Dedicated simulation thread | C | Open |
| D11 | Config format for thresholds/tiers | JSON vs TOML vs QML-independent C++ struct + file | Single config struct loaded from a file, with compiled defaults | B / C | Open |
| D12 | Logging approach | Qt logging vs small custom rate-limited logger | Small rate-limited logger | C | Open |

## Decided

| # | Date | Decision | Rationale |
|---|---|---|---|
| 1 | 2026-11-15 | C++20 + Qt 6/QML + SQLite; Python only for offline experiments | From the project report; avoids browser runtime overhead |
| 2 | 2026-11-15 | Deterministic controller first; no ML in critical loop | Baseline must be validated first |
| 3 | 2026-11-15 | Observation / decision / execution separation | Core architecture principle |
| 4 | 2026-11-15 | Split work across three devs by module ownership | Enables parallel progress ([roadmap.md](roadmap.md)) |

## Entry template

```
### DN: <title>
Date:
Context:
Options considered:
Decision:
Consequences / follow-ups:
```
