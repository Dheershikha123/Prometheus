# NORTHSTAR

The document that settles arguments. When a feature, a refactor or a visual idea is in doubt, come back here.

## The thesis

> **An artificial organism that learns to survive within the computational limitations of its host.**

## The one proof

Under rising host pressure, PROMETHEUS:

1. **reduces its own computational work** (measurably, at process level),
2. **preserves its essential identity**, and
3. **resumes normal activity when conditions recover**.

If a piece of work does not strengthen this proof, it is not MVP.

## The engineering story we are telling

Not: *"we used AI to create a digital creature."*

But: *"we built an artificial organism whose survival behaviour is coupled to a measurable, bounded computational budget."*

## Decision filter

Before starting or accepting any work, ask in order:

1. Does it make the **pressure → behaviour → budget → measured workload** chain stronger, clearer or more testable?
2. Does every new internal variable have a **causal effect** and a **test**?
3. Is it **bounded** (memory, work, queues, history)?
4. Does it keep the system **deterministic and reproducible** by default?
5. Can we **measure** the claim it supports?

If the answer to (1) is no, defer it. If (2) to (5) fail, fix before merging.

## MVP definition of done

A transparent Windows desktop organism that:

- monitors real CPU and RAM with validity and timestamps (plus a synthetic telemetry mode),
- maintains energy and stress,
- changes behaviour and simulation complexity under pressure through a deterministic state machine,
- enforces a bounded `ComputeBudget` with quality tiers,
- enters **genuinely cheap** dormancy and recovers,
- persists identity and checkpoints in SQLite and survives restart,
- and ships with a **repeatable benchmark** comparing fixed vs adaptive complexity.

## Success criteria (all must be evidenced, not asserted)

| # | Criterion | Evidence |
|---|---|---|
| 1 | Pressure changes behaviour | Integration test + live demo trace |
| 2 | Behaviour changes the budget | Budget log per tier transition |
| 3 | Budget changes real process cost | Process CPU / private working set, adaptive vs fixed |
| 4 | Dormancy is real, not cosmetic | Dormant CPU activity measured; normal updates provably stopped |
| 5 | Identity survives dormancy and restart | Restore test; same id, age, essential state |
| 6 | No oscillation | Hysteresis and dwell-time tests; transition log |
| 7 | Hard limits hold | Property tests: budget never exceeds caps |

## Non-goals (for now)

- Genetic evolution, reproduction, morphology inheritance (needs a fitness definition first)
- Reinforcement learning or any ML in the critical loop
- LLM personality or chat
- Complex 3D rendering
- Multi-organism ecosystems
- GPU telemetry beyond optional NVML after the CPU/RAM MVP

## What we will not claim

- That the modules exist before they do.
- That ~50 MB idle RAM or ~1% CPU were achieved before they are measured on a documented machine.
- That synthetic telemetry or prerecorded results are live measurements.
- That any feature or judging alignment is an official Kiroween requirement before it is verified.
- That the organism is conscious, alive, or "learning" beyond what a measured outcome shows.

## Principles

| Principle | Meaning |
|---|---|
| Observe / decide / execute separation | Sensors report, Brain decides, Metabolism budgets, Body obeys |
| Bounded by design | Fixed-capacity pools, capped histories, explicit budgets |
| Graceful degradation | Visuals and detail fall before state or stability |
| Degrade fast, recover slowly | Prevents oscillation |
| Deterministic baseline | Same inputs, same decisions |
| Measured claims | Machine, workload, duration and method recorded |
| No decorative biology | A variable that changes no behaviour is deleted |
