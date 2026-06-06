# Failure Modes

> A system that cannot describe how it fails is a system that will fail without warning.

## HOOK

Ternary systems are not immune to failure. They fail differently: not with a clean crash but with a drift in the balance of `-1, 0, +1` signals that can propagate silently until the system produces confident wrong answers.

## REVEAL

The SuperInstance ecosystem is designed to be fault-tolerant, but fault tolerance requires an honest inventory of failure modes. This document catalogs the ways agents, fleets, and constructs can fail, organized by where the failure originates.

### 1. Capability Failure

A capability failure occurs when an agent advertises a capability it cannot actually provide.

**Examples:**
- An agent claims `KernelExecutor { min_sm: 80 }` but its GPU is SM 70.
- A construct declares compatibility with compute capability 80 but uses PTX instructions that require SM 90.
- A node loses network connectivity but does not update its status to Offline.

**Symptoms:** Work is assigned to the agent and fails at runtime. `oxide-fleet` stats show successful assignments but `oxide-circuit-breaker` trips open.

**Mitigations:** Capability self-tests on registration, periodic health checks, and breaker-based isolation.

### 2. Conservation Violation

A conservation violation occurs when a ternary operation does not preserve the invariants documented in CONSERVATION-LAWS.md.

**Examples:**
- A ternary kernel overflows its `{-1, 0, +1}` range during accumulation.
- A weight update pushes a ternary weight to a value outside the allowed symbol set.
- A cross-agent merge produces a state that no single agent could have produced.

**Symptoms:** Silent numerical drift, incorrect classifications, divergent CRDT states.

**Mitigations:** Runtime conservation checks, `oxide-sandbox` validation, and bounded arithmetic.

### 3. Cascade Failure

A cascade failure occurs when one failure triggers others faster than the breakers can isolate.

**Examples:**
- A popular kernel trips a breaker; all traffic falls back to a CPU implementation; the CPU nodes saturate and fail.
- A canary rollback triggers a fleet-wide construct reload that overwhelms the registry.
- A memory leak in one construct exhausts GPU memory and causes neighboring constructs to fail.

**Symptoms:** Correlated failures across multiple agents, load spikes on fallback paths.

**Mitigations:** Gradual fallback throttling, rate-limited rollbacks, and resource quotas via `oxide-slotmap`.

### 4. Mode Confusion

A mode confusion occurs when an agent operates in the wrong reasoning mode for the situation.

**Examples:**
- A HARDCODE reflex fires on a novel input that requires MODEL reasoning.
- A MODEL deliberates extensively on a routine input that should use HARDCODE.
- A HYBRID construct is deployed before sandbox validation completes.

**Symptoms:** Slow responses, wrong actions, or unsafe execution of unverified code.

**Mitigations:** Confidence gating, mandatory validation states, and explicit mode telemetry.

### 5. Learning Corruption

A learning corruption occurs when online updates degrade rather than improve behavior.

**Examples:**
- Catastrophic forgetting after a new task is learned.
- Adversarial examples poison the replay buffer.
- A generated construct passes tests but fails under real distribution shift.

**Symptoms:** Accuracy drops on previously mastered tasks, unexpected behavior on familiar inputs.

**Mitigations:** Replay buffers, elastic weight consolidation, canary validation of learned constructs.

### 6. Coordination Failure

A coordination failure occurs when multiple agents fail to reach agreement.

**Examples:**
- CRDT registries converge to different states due to conflicting updates.
- Two agents assign the same work to the same GPU, causing a conflict.
- A network partition creates two fleet halves with divergent breaker states.

**Symptoms:** Duplicate work, stale capabilities, inconsistent health verdicts.

**Mitigations:** Version vectors, CRDT merge semantics, and partition-aware circuit breakers.

### 7. Human-Interface Failure

A human-interface failure occurs when the agent and its operator have mismatched expectations.

**Examples:**
- The agent takes an action the user did not authorize.
- The agent explains a decision in language the user does not understand.
- The agent hides uncertainty behind overconfident output.

**Symptoms:** User distrust, misuse, disengagement.

**Mitigations:** Explicit confidence reporting, approval gating for high-stakes actions, and ternary status displays.

## Experiments

1. **Fault injection**: Systematically inject each failure mode into a test fleet and measure detection time, containment radius, and recovery time.
2. **Cascade propagation**: Simulate a breaker trip under different load conditions and measure whether fallback paths survive.
3. **Mode confusion audit**: Review agent logs and classify decisions that used the wrong reasoning layer. Measure frequency and cost.
4. **CRDT divergence**: Partition a fleet, issue conflicting updates on both sides, and measure merge behavior after healing.

## Applications

- **Incident response playbooks**: Map observed symptoms to failure modes for faster diagnosis.
- **Red team exercises**: Deliberately trigger failure modes to validate mitigations.
- **System design reviews**: Use the catalog as a checklist when adding new agents or constructs.
- **Safety certification**: Demonstrate that known failure modes are enumerated and addressed.
- **User documentation**: Help operators understand what can go wrong and what to watch for.

## Open Questions

1. **Unknown unknowns**: How do we discover failure modes that are not in this catalog?
2. **Failure mode interactions**: Do combinations of minor failure modes produce emergent major failures?
3. **Quantitative risk**: Can we assign probabilities and severities to each failure mode?
4. **Automated recovery**: Which failure modes can be handled without human intervention, and which require escalation?

## CONNECT

- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — The design principles that mitigate these failure modes.
- [DEBUGGING-AND-TRACING.md](DEBUGGING-AND-TRACING.md) — How to investigate failures when they occur.
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — Practical steps for recovering from failures.
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — The invariants that conservation violations break.
- [ONLINE-LEARNING.md](ONLINE-LEARNING.md) — Where learning corruption originates.
- `oxide-circuit-breaker` — Isolates cascade failures.
- `oxide-canary` — Catches failures before full deployment.

## ACTIVATE

For each of the seven failure modes above, identify one concrete scenario that could occur in a system you operate. Write a one-paragraph incident narrative for each: what happened, what symptom was observed, and what mitigation activated (or failed to activate). Rate each scenario by likelihood and severity. Pick the highest-risk scenario and design a simple test that would detect it before it affects users. Run that test and document the result.
