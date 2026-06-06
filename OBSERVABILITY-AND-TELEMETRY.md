# Observability and Telemetry

> In a ternary fleet, you do not monitor boolean health; you observe a field of {-1, 0, +1} signals propagating through silicon and agents.

## HOOK

Traditional observability asks "is the system up or down?" Ternary observability asks "where is the system in its phase space, and in which direction is it moving?"

## REVEAL

A GPU fleet running agentic workloads produces three qualitatively different kinds of telemetry:

1. **Infrastructure telemetry** — temperature, memory, utilization, PCIe bandwidth.
2. **Kernel telemetry** — latency, throughput, error rate, register pressure, warp divergence.
3. **Agent telemetry** — intent signals, capability advertisements, breaker states, canary verdicts, rhythm analysis.

Most monitoring systems collapse these into scalar dashboards: CPU%, memory%, request rate. That collapse destroys information. A GPU at 95% utilization may be healthy (saturated throughput) or unhealthy (spinning on a barrier). The scalar is not enough. You need the ternary signal: `{-1, 0, +1}` mapped to `degrading / steady / improving`.

### The Ternary Health Vector

For every observed entity — agent, kernel, GPU, construct — define a health vector:

```
health = (error_signal, latency_signal, throughput_signal)
```

where each component is:

- `-1` — worse than baseline
- `0` — within baseline bounds
- `+1` — better than baseline

The vector is not reduced to a single color. A kernel with `(-1, 0, +1)` is not "yellow." It is "errors rising, latency steady, throughput improving" — a precise diagnosis that tells you whether the problem is in the algorithm, the input distribution, or the hardware scheduling.

### Signal Aggregation

Agents produce high-cardinality signals: one breaker state per kernel per node, one canary verdict per release, one rhythm metric per agent. Naive aggregation loses the topology. The correct aggregation is a **labeled event graph** where nodes are entities and edges are causal relationships.

For example:

- A `Rollback` verdict in `oxide-canary` should create an edge to `oxide-circuit-breaker` tripping Open.
- An Open breaker should create an edge to `oxide-fleet` excluding that kernel from assignment.
- A fleet assignment failure should create an edge to the agent's capability advertisement.

This graph is the observability data model. Dashboards are views onto the graph; alerts are graph traversals.

### Sampling Strategy

GPU kernel telemetry is too voluminous to capture at 100%. The recommended strategy is **ternary sampling**:

- Always capture `-1` events (failures, rollbacks, breaker trips).
- Sample `0` events at 1% (steady state).
- Capture `+1` events at 10% (improvements are rarer and worth understanding).

This produces a biased but information-rich sample that respects the asymmetry of failure and success.

## Experiments

The oxide crates encode ternary observability primitives that can be composed into experiments:

1. **Breaker propagation latency**: Inject a synthetic kernel failure on one node and measure how many milliseconds until every fleet node marks the kernel Open.
2. **Canary detection sensitivity**: Run identical baseline and canary kernels under production load and measure the false-rollback rate; then introduce a 5% latency regression and measure time-to-detect.
3. **Rhythm prediction accuracy**: Use `oxide-fleet`'s `analyze_rhythm()` history to predict next-hour load imbalance, then compare against actual assignment distribution.

A larger experiment: instrument 16 fleet nodes with ternary health vectors for one week. Train a simple classifier to predict breaker trips from the preceding five minutes of health vectors. Expected outcome: >85% precision with >80% recall, demonstrating that ternary signals carry predictive power beyond scalar utilization metrics.

## Applications

- **Root-cause analysis**: Follow the labeled event graph from a rollback to the originating kernel change, build, and construct version.
- **Capacity planning**: Use rhythm analysis and health vectors together to scale the fleet before load imbalance becomes saturation.
- **Incident response**: Trigger breaker trips automatically when health vectors show correlated `-1` signals across multiple kernels.
- **Optimization feedback**: Capture `+1` signals to identify which compiler flags, construct versions, or scheduling policies actually improve performance.
- **Agent self-monitoring**: Agents observe their own health vectors and request help from the fleet when their signals degrade.

## Open Questions

1. **Health vector cardinality**: Three signals per entity is manageable, but what happens when entities number in the millions? Can we compress without semantic loss?
2. **Causal inference**: Correlation in the event graph is not causation. What experimental framework proves that a canary change caused a latency shift?
3. **Privacy boundaries**: Agent telemetry may include user-facing latency patterns. How do we aggregate across tenants without leaking behavior?
4. **Real-time graph queries**: Can we answer "show me all kernels whose error_signal is -1 and whose parent construct changed in the last hour" in sub-second latency at fleet scale?

## CONNECT

- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — How ternary health signals drive degradation decisions.
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Why every observability claim must be testable.
- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — The methodology for turning telemetry into reproducible measurements.
- [DEBUGGING-AND-TRACING.md](DEBUGGING-AND-TRACING.md) — Following individual requests through the same event graph.
- [COST-ECONOMICS.md](COST-ECONOMICS.md) — The dollar cost of telemetry volume and sampling trade-offs.
- `oxide-circuit-breaker` — Produces `-1` health signals when kernels fail.
- `oxide-canary` — Generates verdicts that are themselves ternary telemetry.
- `oxide-fleet` — Rhythm analysis is a higher-order telemetry stream.

## ACTIVATE

For every service you operate, replace its single health metric with a ternary health vector of `(error_signal, latency_signal, throughput_signal)`. Define baseline bounds using the last seven days of data. Emit the vector every ten seconds. Build one alert that fires only when at least two components are `-1`, and one dashboard that displays the vector as a tri-color sparkline rather than a scalar line. After one week, review whether the vector gave you earlier or more precise incident signals than your previous boolean up/down check.
