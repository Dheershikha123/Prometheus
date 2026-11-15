# Glossary

| Term | Meaning in PROMETHEUS |
|---|---|
| Host observation | Normalized, timestamped report of computer conditions, with per-metric validity |
| Organism state | Internal variables representing the organism's condition (energy, health, stress, curiosity, adaptation, age, ...) |
| Behaviour | Selected action mode: explore, idle, conserve, recover, dormant |
| Compute budget | Explicit limits on simulation and rendering work (particles, Hz, FPS, detail, effects) |
| Quality tier | Discrete budget level, 0 (Dormant) to 4 (Thriving) |
| Hard limits | Non-negotiable caps enforced after any proposal; no heuristic or model may exceed them |
| Pressure | Normalized estimate of host constraint, combined from valid signals and smoothed |
| EMA | Exponential moving average; smooths noisy signals |
| Hysteresis | Different enter/exit thresholds for a state to prevent oscillation |
| Dwell time | Minimum time in a state before another transition is allowed |
| Dormancy | Low-activity lifecycle state that stops normal simulation work while preserving identity; not just a hidden window |
| Adaptation | Within-lifetime adjustment based on prior conditions and outcomes |
| Evolution | Across-generation inheritance and selection of traits |
| Snapshot | Immutable `OrganismSnapshot` published to the UI |
| Synthetic telemetry | Scripted host observations for deterministic tests and demos; always labelled |
| Survival Test | Controlled comparison/demonstration of adaptive vs fixed-complexity behaviour under pressure |
| Observer panel | Developer diagnostics UI separating measured host values from simulated variables |
| Decorative biology | An internal variable with no causal effect; to be removed or given one |
| Private working set | Resident memory attributable to the process (not shared pages) |
| Commit size | Committed virtual memory (`Private Bytes`) |
| NVML | NVIDIA Management Library; optional GPU telemetry |
