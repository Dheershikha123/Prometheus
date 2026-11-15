# Organism Model

> Status: **design**. All thresholds, weights and rates are **initial, untuned placeholders**; tune them from tests and measurements. Items marked *(proposed)* extend the original report.

## 1. The rule that keeps this honest

Every internal variable must have a **causal effect** and a **test**. If curiosity changes no behaviour, it is decoration. If adaptation does not improve a measured outcome, it is not yet demonstrated learning.

## 2. Variables

| Variable | Range | Causal role | MVP? | Test that proves it matters |
|---|---|---|---|---|
| Energy | 0–100 | Low energy → Recover; critically low → Dormant; scales with consumption model | **Yes** | Stays in bounds; critical-low forces Dormant |
| Stress | 0–100 | Accumulates from sustained pressure; drives Conserve | **Yes** | Rises under sustained pressure, decays when safe |
| Curiosity | 0–100 | Gates Explore vs Idle; influences movement | Yes (minimal) | Explore only when curiosity high **and** budget permits |
| Health | 0–100 | Declines under prolonged severe stress, recovers when safe. *(Proposed: caps the maximum quality tier.)* | Post-MVP unless a causal effect is defined | Prolonged stress lowers tier cap |
| Adaptation | 0–100 | Lifetime-learned response strength. *(Proposed: small bounded shift of thresholds.)* | Post-MVP | Improves a measured outcome vs baseline |
| Age | monotonic s | History and optional life-cycle rules | Yes | Monotonic across save/restore |
| Smoothed pressure *(proposed)* | 0–1 | EMA-filtered host pressure estimate | **Yes** | Responds fast up, slower down |

Energy is a **simulated internal reserve**. Keep it conceptually separate from CPU utilization or available RAM: host conditions constrain what is affordable; energy is the organism's own reserve. It is not joules or electricity used.

## 3. Equations (illustrative)

**Energy**

```
E(t+1) = clamp(E(t) + G(t) − C(t), 0, 100)
C = C_base + C_movement + C_complexity + C_stress
G = recovery gain under safe conditions
```

`C_complexity` should scale with the active quality tier so that running richer simulation visibly costs the organism more.

**Filtering (EMA)**

```
EMA(t) = α·x(t) + (1 − α)·EMA(t−1)
```

Use a larger α for rising pressure (react quickly) and a smaller α for falling pressure (recover gradually). Smaller α = smoother but slower.

**Environmental pressure** (weights are configurable; not universal truth)

```
P = Σ w_i · p_i  over VALID signals only, renormalizing w over the valid set
signals: CPU, RAM (and available-memory trend), optional GPU
```

- CPU alone is not distress: a busy CPU may be doing useful work. Weight available memory, memory trend and the organism's own resource use as context.
- If GPU telemetry is missing, drop its term and renormalize. Never treat missing as zero pressure.

**Stress accumulation** *(proposed form)*

```
S(t+1) = clamp(S(t) + k_up·max(0, P − P_safe)·dt − k_down·max(0, P_safe − P)·dt, 0, 100)
```

## 4. Behaviours and policy

Baseline deterministic policy (thresholds configurable, boundary-tested):

```
if energy is critically low:        DORMANT
else if sustained pressure severe:  CONSERVE
else if energy is low:              RECOVER
else if curiosity high and budget permits: EXPLORE
else:                               IDLE
```

| Behaviour | Typical effect | Exit idea |
|---|---|---|
| Explore (Active) | Movement, normal detail | → Conserve on sustained pressure; → Idle when curiosity falls |
| Idle | Low activity, observation | → Explore when curiosity rises and resources permit |
| Conserve (Stressed) | Reduce movement and detail | → Recover once pressure stays below the *lower* threshold |
| Recover (Recovery) | Rebuild energy, cautious restoration | → Idle/Explore when recovery conditions met |
| Dormant | Stop normal sim and expensive rendering; keep essential state | Wake on host recovery or user resume |

The safety controller (Metabolism + runtime) has final authority over resource limits regardless of what policy proposes.

## 5. Transition safeguards

- **Hysteresis:** different enter/exit thresholds (e.g., enter Conserve when smoothed pressure > upper; exit only below lower).
- **Minimum dwell time** in each behaviour before another transition is allowed.
- **Debounce:** a single noisy sample must never trigger a change.
- **Asymmetric pacing:** degrade immediately under severe pressure; restore one tier at a time.

Illustrative placeholder values, **all untuned**:

| Parameter | Placeholder |
|---|---|
| Enter Conserve | smoothed pressure > 0.70 |
| Exit Conserve | smoothed pressure < 0.50 |
| Minimum dwell | 5 s |
| Critical energy | < 10 |
| Low energy | < 30 |
| Wake from Dormant | pressure < 0.40 for N s, and energy above wake threshold |

## 6. Adaptation vs evolution

| | Adaptation | Evolution |
|---|---|---|
| Scope | Within one organism's lifetime | Across generations |
| Mechanism | Shift preferences/parameters from repeated conditions | Inheritance + selection |
| Status | Post-MVP; start as bounded contextual statistics | Deferred until a fitness definition and stable survival model exist |

Do not conflate them. A first adaptation mechanism: keep bounded statistics of which strategies worked under which pressure contexts; compare against the deterministic baseline before claiming any benefit. Start offline.

## 7. Life-cycle (future)

Generation, traits and morphology inheritance are persisted in the schema but unused in the MVP. Introduce reproduction only with a transparent fitness measure (for example, survival duration under a defined resource profile and fixed compute budget).
