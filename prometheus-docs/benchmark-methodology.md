# Benchmark Methodology

Purpose: make every performance claim **reproducible and defensible**. The thesis, that survival behaviour changes the real computational footprint, is only proven by process-level measurements.

> **No numbers exist yet.** Every results table below is a blank template. Do not fill it with estimates. The ~50 MB idle RAM and ~1% average CPU figures are aspirational targets until measured.

## 1. What we compare

| Variant | Description |
|---|---|
| **Adaptive** | Full controller: pressure → behaviour → budget → tiers → dormancy |
| **Fixed-complexity** | Same app, same visuals, controller pinned to a fixed tier (e.g., Thriving or Normal), no degradation, no dormancy |

Everything else identical: build, machine, workload script, duration, telemetry source.

## 2. Metrics

| Metric | Why | How |
|---|---|---|
| Process CPU (avg, peak) | Core claim | `\Process(prometheus)\% Processor Time` (divide by logical core count to compare with Task Manager) |
| Private working set | Resident memory | `\Process(prometheus)\Working Set - Private` |
| Commit size | Committed virtual memory | `\Process(prometheus)\Private Bytes` |
| GPU memory / utilization | Graphics overhead (optional) | Task Manager / NVML on supported hardware; verify integrated-GPU-only operation |
| Simulation update time | Cost and variability | Internal instrumentation (mean, p95, max) |
| Render FPS per tier | Responsiveness | Internal counter + external check |
| Dormant CPU activity | Proves dormancy is real | Process CPU during dormancy; count of normal sim ticks (must be 0) |
| Recovery latency | Time from safe conditions to restored activity | Timestamped transition log |

**Do not infer application savings from total system CPU.** Measure the process itself.

## 3. Tools

- Windows Performance Monitor (`perfmon`): counters above, logged to file.
- Process Explorer: independent cross-check.
- Windows Performance Recorder / Analyzer: deeper investigation.
- Internal metrics exported to a CSV per run (tick counts, tier changes, budget values, with timestamps).

## 4. Environment record (fill for every run)

| Field | Value |
|---|---|
| Date | TBD |
| Windows version/build | TBD |
| CPU model, cores/threads | TBD |
| RAM | TBD |
| GPU (integrated/discrete), driver | TBD |
| Power mode / battery state | TBD |
| Build (commit hash, preset, config) | TBD |
| Qt version | TBD |
| Display (resolution, scaling, monitors) | TBD |
| Background load | TBD |

## 5. Protocol

1. Clean boot or documented background state; close unrelated apps; plugged in, fixed power plan.
2. Build the exact commit in Release; record the hash.
3. Launch; **warm-up** (e.g., 60 s, untuned) discarded from statistics.
4. Run the **scripted workload** (see §6) for a fixed duration.
5. Log counters at a fixed interval (e.g., 1 s) plus internal CSV.
6. Repeat **at least 3 runs** per variant, alternating order to avoid drift.
7. Report **median and range**, not a single best run. Note outliers and why.
8. Archive raw logs under `docs/benchmarks/<date>-<machine>/` (not summaries only).

## 6. Workload script

| Phase | Duration | Condition |
|---|---|---|
| Baseline idle | TBD | No stress |
| Ramp | TBD | Rising CPU/memory pressure from the stress tool |
| Sustained pressure | TBD | Hold; observe degradation and dormancy |
| Release | TBD | Stop stress; observe recovery |
| Post-recovery idle | TBD | Verify tier restore, no oscillation |

**Stress tool requirements:** separate executable; bounded intensity; hard duration cap; safe stop (Ctrl+C and kill file/flag); never risks destabilizing the machine. Provide a **synthetic telemetry** equivalent for deterministic runs; it is labelled synthetic in every output.

## 7. Results templates (blank)

**Process footprint**

| Variant | Avg CPU % | Peak CPU % | Private WS (MB) | Private Bytes (MB) |
|---|---|---|---|---|
| Fixed | TBD | TBD | TBD | TBD |
| Adaptive: healthy | TBD | TBD | TBD | TBD |
| Adaptive: under pressure | TBD | TBD | TBD | TBD |
| Adaptive: dormant | TBD | TBD | TBD | TBD |

**Behaviour and responsiveness**

| Metric | Value |
|---|---|
| Time from pressure onset to first degradation | TBD |
| Tier at sustained pressure | TBD |
| Dormant normal-sim ticks | TBD (expected 0) |
| Recovery latency | TBD |
| State transitions during stable load (oscillation check) | TBD |

## 8. Reporting rules

- Every figure states machine, duration, workload and method.
- Distinguish **live** from **synthetic** and **prerecorded**; never blur them.
- Report limitations (single machine, short duration, etc.).
- Targets are targets; only measured values are results.
