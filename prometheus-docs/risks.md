# Risk Register

| # | Risk | Why it matters | Mitigation | Owner |
|---|---|---|---|---|
| 1 | Biological variables become decorative | Loses internal coherence | Require causal effect + test per variable | B |
| 2 | CPU utilization treated as stress by itself | Busy ≠ unhealthy | Combine CPU, RAM, trends, available memory | B |
| 3 | Monitoring overhead | Monitor becomes the problem | Centralized sampling; measured intervals | A |
| 4 | Rendering drives simulation | UI refresh creates needless work | Separate clocks; immutable snapshots | C |
| 5 | ML added for appearance | Dependencies, no gain | Require measurable gain over deterministic baseline | B |
| 6 | Evolution without fitness | Trait changes meaningless | Defer until fitness + inheritance defined | All |
| 7 | Frequent database writes | I/O and latency | Event-based + periodic checkpoints; off render path | A |
| 8 | **Dormancy is cosmetic** | Central claim fails | Stop normal updates; measure dormant process activity | C / A |
| 9 | Recovery oscillation | Instability | Hysteresis, dwell times, gradual recovery | B |
| 10 | Contract drift between devs | Parallel work stalls | Frozen, versioned contracts; two-approval change rule | All |
| 11 | Dev B blocked on telemetry | Test loop depends on it | Synthetic telemetry is Dev A's first deliverable | A |
| 12 | Dev C overloaded | Critical path | Reassign observer panel / coordinator tests to first free dev | All |
| 13 | Late integration | Surprises at the end | `main` runnable with stubs from day one; phase-end merges | C |
| 14 | Qt baseline RAM exceeds ~50 MB target | Aspirational target may be unreachable | Profile Qt/runtime baseline early; report measured value | C / A |
| 15 | Click-through / transparent window quirks | Demo fails on some displays | Test multi-monitor/DPI early; fallback mode | C |
| 16 | Stress tool destabilizes demo machine | Demo failure | Bounded intensity, duration cap, safe stop, synthetic mode | A |
| 17 | Unverified hackathon rules | Wrong deliverables | Verify in Phase 0 ([hackathon-checklist.md](hackathon-checklist.md)) | All |
| 18 | Overclaiming results | Credibility loss | Measured claims only; label synthetic/prerecorded | All |
| 19 | NVML/GPU assumptions | Not all hardware has it | Optional; core works without GPU | A |
| 20 | Corrupt checkpoint on crash/shutdown mid-transition | Identity lost | Transactions, atomic replace, recovery path, fault tests | A |
