# Distributed Compute Graph

> A single GPU executes a graph. A fleet executes a graph that spans space, trust boundaries, and failure domains.

## HOOK

The distributed compute graph is not a larger version of a local compute graph. It is a different mathematical object: a graph whose edges are network links, whose nodes are agents with heterogeneous capabilities, and whose execution semantics must tolerate partial failure without producing partial truth.

## REVEAL

A local compute graph is a directed acyclic graph (DAG) where nodes are operations and edges are tensors. Frameworks like PyTorch, TensorFlow, and JAX optimize these graphs for a single machine: fusion, scheduling, memory planning.

A distributed compute graph in the SuperInstance ecosystem has additional dimensions:

- **Capability heterogeneity**: Not every node can execute every operation. A Flux compiler may live on one agent; a kernel executor with tensor cores may live on another.
- **Network topology**: Edges have latency, bandwidth, and reliability properties that vary by physical location.
- **Dynamic membership**: Agents join and leave the fleet. The graph must be re-routable at runtime.
- **Partial failure**: A node can fail without failing the whole computation, but only if the graph contains redundant paths or explicit checkpointing.

### The Three Graph Layers

SuperInstance decomposes distributed execution into three layered graphs:

1. **Intent graph**: The user's high-level goal, expressed as a declarative computation. "Compute the attention forward pass over this batch using the ternary attention construct."
2. **Capability graph**: A mapping from intent to required capabilities and from capabilities to agents that provide them. This is the output of `oxide-fleet` discovery.
3. **Execution graph**: The concrete assignment of operations to agents with data-flow edges between them. This is what actually runs.

The intent graph is static. The capability graph changes when agents join or leave. The execution graph is a runtime instance that must be reconfigurable when the capability graph changes.

### Execution Semantics

A distributed compute graph has three execution primitives:

- **Local compute**: Run an operation on a single agent.
- **Send**: Transfer a tensor from one agent to another.
- **Barrier**: Synchronize a set of agents before continuing.

These primitives compose into patterns:

- **Pipeline parallelism**: Different layers of a model run on different agents; activations flow forward.
- **Tensor parallelism**: A single layer is sharded across agents; partial results are reduced.
- **Expert parallelism**: Different inputs route to different expert agents based on a gating function.
- **Redundant execution**: Critical operations run on multiple agents; outputs are voted or CRDT-merged.

### Failure and Conservation

The most subtle problem in distributed graphs is conservation. If a tensor is produced by one agent and consumed by another, the consumer must receive exactly what the producer sent — not approximately, not eventually, but exactly. This is why the ecosystem emphasizes:

- **Deterministic kernels**: Same inputs always produce same outputs, enabling recomputation after failure.
- **CRDT state**: Agent state can be merged without coordination, enabling recovery from partition.
- **Circuit breakers**: Failed kernels are isolated before their errors propagate.
- **Canary deployments**: New graph nodes are validated before they carry full traffic.

## Experiments

1. **Capability discovery latency**: Submit an intent graph requiring three capabilities to a fleet of 32 agents and measure time to produce a valid execution graph. Expected: O(log n) with gossip, O(1) with a centralized coordinator.
2. **Recovery from node failure**: Run a 10-node pipeline, kill one node mid-execution, and measure time to reassign and resume. Expected: <2× baseline latency with checkpointing every layer.
3. **Bandwidth vs. compute trade-off**: Shard a large matmul across 2, 4, and 8 agents. Measure total execution time. Expected: speedup peaks when communication overhead equals compute savings.
4. **Ternary all-reduce**: Implement an all-reduce for ternary tensors using `TADD` and measure bandwidth efficiency versus FP32 all-reduce. Expected: 10-16× bandwidth reduction.

## Applications

- **Large model inference**: Distribute transformer layers across a fleet of GPUs to serve models larger than any single GPU's memory.
- **Federated training**: Compute gradients on edge agents, aggregate centrally, and broadcast updates without moving raw data.
- **Multi-agent scientific computing**: Compose physics simulators, statistical models, and visualization agents into a single distributed experiment.
- **Resilient streaming**: Build pipelines where failed agents are automatically replaced without dropping events.
- **Cross-datacenter ML**: Train models with data and compute spread across regions, using the graph to minimize data movement.

## Open Questions

1. **Optimal sharding**: Given an intent graph and a capability graph, what algorithm produces the execution graph with minimal latency and energy?
2. **Dynamic reconfiguration**: How quickly can the execution graph adapt to capability graph changes without restarting the computation?
3. **Byzantine agents**: What fraction of malicious agents can a distributed graph tolerate while still producing correct results?
4. **Economic scheduling**: Should execution graphs optimize for wall-clock time, dollar cost, carbon emissions, or some weighted combination?

## CONNECT

- [FLEET-MAP.md](FLEET-MAP.md) — The 303-crate geometry that the distributed graph navigates.
- [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md) — The ternary signal protocol underlying graph edges.
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — How partial failure is contained.
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Why exact conservation matters across network boundaries.
- [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md) — How distributed graphs are deployed and observed.
- `oxide-fleet` — Produces capability graphs and execution assignments.
- `oxide-circuit-breaker` — Isolates failed nodes in the execution graph.
- `oxide-canary` — Validates new nodes before they join the graph.

## ACTIVATE

Take a compute task you currently run on a single GPU and decompose it into three operations that can run on different agents. Define the input/output tensors for each operation and the send edges between them. Implement the execution graph using `oxide-fleet` for capability discovery and `oxide-circuit-breaker` for failure isolation. Run the graph on two agents connected by a real network link. Measure the end-to-end latency and compare it against the single-GPU baseline. Identify which edge in your graph is the bottleneck and optimize it.
