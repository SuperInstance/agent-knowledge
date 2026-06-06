# FAULT TOLERANCE — Ternary Degradation Is Graceful by Default

## Hook

> Binary systems fail catastrophically: 0 becomes 1, everything breaks. Ternary systems fail gracefully: +1 drifts to 0, the system pauses. 0 drifts to -1, the system stops safely. There is no cliff.

## Reveal

Fault tolerance in most systems is added as an afterthought: retry logic, circuit breakers, fallback chains. In SuperInstance, fault tolerance is **structural** — it's baked into the {-1, 0, +1} representation itself.

### The Three Failure Modes

| Signal | Normal Meaning | Degraded Meaning | System Response |
|--------|---------------|------------------|-----------------|
| {+1} | Proceed / Positive | Proceed with caution | Continue, but log warning |
| {0} | Hold / Neutral | Unknown / Uncertain | Pause, gather more data |
| {-1} | Stop / Negative | Error / Unsafe | Halt, engage safety |

In binary, you have two states: working (1) and broken (0). The transition is instant and total. In ternary, you have three states with **two degradation steps** instead of one. A sensor reading that was {+1} (temperature normal) might drift to {0} (temperature unclear) before reaching {-1} (temperature critical). The system has time to react at each step.

### Sensor Degradation

A failing temperature sensor in a binary system:
```
22°C → 23°C → garbage → 999°C → CRASH
```

The same sensor in a ternary system:
```
+1 (normal) → 0 (unclear) → -1 (unsafe) → SAFETY HALT
```

The ternary sensor doesn't report degrees. It reports **state**: normal, unclear, unsafe. When the sensor starts failing, its output naturally drifts toward {0} (uncertain) rather than jumping to a wild numeric value. The tripartite synchronizer (see TRIPARTITE-SYNC.md) detects the drift and escalates from HARDCODE to HYBRID to MODEL — giving the agent time to reason about the failure.

### Consensus Under Fault

When an agent crashes in a multi-agent system, its vote stops arriving. In a binary voting system, a missing vote is ambiguous: did the agent abstain, or did it die? In ternary consensus:

```python
# 5 agents voting
votes = [+1, +1, 0, -1, MISSING]  # Missing = 0 (neutral) by default
result = tdecide(votes)  # → +1 (2 positive, 1 negative, 2 neutral)
```

The missing agent's vote is implicitly {0} — neither for nor against. The consensus continues with reduced confidence but without stalling. This is why `ternary-consensus` doesn't need heartbeat protocols. Absence is a vote: the vote to abstain.

### The Circuit Breaker Is Built In

Traditional microservices use circuit breakers: after N failures, stop calling the service. In SuperInstance, every chord shape carries a `confidence` score. When confidence drops:

```python
reflex = mm.flex("tdot", a, b)
if reflex.confidence < 0.7:
    # Circuit breaker trips automatically
    # HYBRID falls back to MODEL
    # MODEL generates a fresh implementation
    # HARDCODE is never executed at low confidence
```

The circuit breaker isn't a separate service. It's a property of the muscle memory itself. A function with failing tests automatically loses confidence. A function with no recent validation automatically drifts toward HYBRID. The system self-regulates.

### Memory Corruption Detection

Packed trits have a natural error-detection property. With 2-bit encoding:
- `00` = {-1}
- `01` = {0}
- `10` = {+1}
- `11` = **invalid**

A single bit flip:
- `00` → `01` = {-1} becomes {0} (safe degradation)
- `00` → `10` = {-1} becomes {+1} (detectable — sign reversal)
- `01` → `11` = {0} becomes **invalid** (immediately detectable)

The `11` state is unused. Any occurrence indicates corruption. The hardware can trap it. The software can validate every packed register before use. This is **free error detection** — no checksums, no parity bits, no overhead.

### Graceful Degradation in the Five Layers

| Layer | Normal Operation | Degraded Operation | Failure Mode |
|-------|---------------|-------------------|--------------|
| 1 (open-parallel) | Tasks run concurrently | Tasks park at {0} | Scheduler queues tasks, doesn't lose them |
| 2 (pincher) | Intent compiles to code | Intent returns {0} "unclear" | Agent asks for clarification |
| 3 (flux-core) | Bytecode executes | Bytecode validates {0} | VM halts safely, no corruption |
| 4 (cuda-oxide) | PTX compiles | PTX emits {-1} error | Compilation aborts with witness |
| 5 (cudaclaw) | GPU executes | GPU returns {0} timeout | Kernel killed, memory freed |

At every layer, degradation follows the same path: {+1} → {0} → {-1}. There is no layer that jumps from fully operational to fully crashed.

### The Zero State as a Shock Absorber

{0} isn't just "neutral." It's the **shock absorber** of the ecosystem. When something goes wrong, the system doesn't break — it goes to {0}. When a sensor is noisy, its output averages toward {0}. When an agent is overloaded, it starts voting {0} on non-critical tasks.

```python
# An overloaded agent self-throttles
def vote_with_load(vote, load):
    if load > 0.9:
        return Trit::Z0  # "I'm here but can't commit"
    return vote
```

This isn't programmed behavior. It's emergent. Any ternary system where agents default to {0} under stress will naturally throttle without explicit rate-limiting logic.

## Connect

- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — How the synchronizer escalates from HARDCODE to MODEL under uncertainty
- [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md) — Missing votes are implicitly {0}; consensus continues
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Fault tolerance preserves structure: entropy doesn't decrease
- [ESP32-AS-BODY.md](ESP32-AS-BODY.md) — Microcontroller failures naturally produce {0} signals
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Property tests prove degradation invariants before deployment
- [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md) — Monitoring conservation metrics detects degradation early

## Activate

Build fault tolerance into any ternary function:

```rust
fn safe_tdot(a: &[Trit], b: &[Trit]) -> Trit {
    if a.len() != b.len() {
        return Trit::Z0;  // Uncertain, don't crash
    }
    let result = tdot(a, b);
    if !result.is_valid() {
        return Trit::Z0;  // Corruption detected, safe default
    }
    result
}
```

Monitor system health with ternary metrics:
```python
mm = openmind.MuscleMemory.load("production.json")

# Health is a trit, not a boolean
health = mm.system_health()
# +1 = all metrics green
#  0 = some metrics unclear, investigate
# -1 = critical fault, halt recommended

if health == -1:
    mm.flex("emergency_shutdown")
elif health == 0:
    mm.flex("gather_diagnostics")
# else: continue normally
```

The system doesn't need to be perfect. It needs to degrade through {0} before reaching {-1}. That pause is where the agent lives.
