# Memory Architecture

> An agent's memory is not a database. It is a topology of recall gradients across time, space, and abstraction.

## HOOK

The hard problem of agent memory is not storage capacity. It is deciding, at every moment, which memories should be hot in GPU registers, which should be warm in context windows, and which should be cold in CRDT-backed persistent stores — without stopping to think about the decision.

## REVEAL

Agent memory in the SuperInstance ecosystem spans five hierarchical tiers, each with different latency, capacity, and persistence properties. These tiers are not merely a cache hierarchy. They are a cognitive architecture.

### Tier 0: Register Memory (nanoseconds)

The fastest tier is the GPU register file and CPU cache lines actively holding the agent's current reasoning state. This includes:

- Attention activations during a forward pass.
- Ternary weights loaded into tensor cores.
- Loop variables and confidence values.

Register memory is ephemeral by design. It exists only while the agent is actively thinking. The art of efficient agent inference is keeping the working set small enough to stay in fast memory.

### Tier 1: Context Window (milliseconds)

The context window is the agent's short-term, differentiable memory. It holds recent tokens, tool outputs, perceptual embeddings, and conversation history. Its capacity is fixed by model architecture — typically 4K to 128K tokens — and its access pattern is attentional: the model reads everything at once, weighted by relevance.

Context windows suffer from three pathologies:

- **Dilution**: As the window grows, any single token's attention weight shrinks.
- **Recency bias**: Models overweight recent tokens unless explicitly prompted.
- **Cost explosion**: Long contexts increase latency and price per call.

The SuperInstance response is the **nail format** (THE-NAIL-FORMAT.md): compress high-value memories into dense, zero-token embeddings that can be injected without consuming context budget.

### Tier 2: Muscle Memory (seconds to minutes)

Muscle memory is the agent's procedural cache: compiled skills, chord shapes, construct binaries, and HARDCODE fallbacks loaded into GPU memory. It is faster than context lookup because it bypasses the model entirely.

Muscle memory is where the HYBRID loop pays off. When an agent compiles a frequently used reasoning pattern into a construct, that pattern moves from context-dependent (MODEL) to procedure-cached (HARDCODE). See MUSCLE-MEMORY.md and THE-AGENT-LOOP.md.

### Tier 3: Construct Registry (minutes to hours)

The construct registry is the agent's long-term semantic memory. It stores named capabilities, their versions, their dependencies, and their execution history. Unlike muscle memory, the registry is addressable by name and can be queried, merged, and gossiped across the fleet.

The registry is backed by CRDT semantics: two agents can merge their registries and converge on the same state without coordination. This makes it suitable for decentralized knowledge. See `oxide-constructs` and FLEET-MAP.md.

### Tier 4: Persistent Store (hours to years)

The slowest tier is durable storage: conversation logs, execution traces, versioned model checkpoints, and organizational knowledge bases. This tier is typically implemented on object storage or distributed databases.

Persistent stores are write-optimized and read-cold. An agent should not reach into persistent storage during a single loop iteration. Instead, important persistent memories should be promoted to the construct registry or muscle memory through an explicit indexing process.

### The Promotion Problem

The central engineering challenge is deciding when a memory should move up or down the hierarchy:

- A user preference mentioned once should probably stay in context.
- A user preference mentioned ten times should be promoted to the registry as a personalized construct.
- A generated skill used once per day should stay in the registry.
- A generated skill used once per millisecond should be compiled to muscle memory.

This promotion is itself a reasoning problem. The agent loop must continuously evaluate access frequency, reuse potential, and compilation cost.

## Experiments

1. **Context dilution measurement**: Insert a critical fact at position 100, 1000, and 10000 in a context window. Measure retrieval accuracy. Expected: exponential decay with position.
2. **Muscle memory speedup**: Compare latency of a MODEL-based reasoning step versus the same logic compiled as a HARDCODE construct. Expected: 10-1000× speedup depending on model size.
3. **Registry merge convergence**: Start with 16 agents, each holding 100 constructs with 10% overlap. Measure rounds of gossip required for full convergence. Expected: O(log n) rounds.
4. **Promotion policy benchmark**: Implement three promotion policies (frequency-only, recency-weighted, utility-predicted) and measure hit rate in the muscle memory tier.

## Applications

- **Personalized agents**: Promote user-specific facts to registry tier so they survive context compaction.
- **Real-time systems**: Keep reflexive skills in muscle memory to meet millisecond deadlines.
- **Collaborative learning**: Merge registries across agents to share discovered skills without sharing raw training data.
- **Cost optimization**: Keep the hottest working set in register/context tiers and evict cold memories to persistent storage.
- **Debugging**: Trace a memory's tier history to understand why an agent "forgot" something.

## Open Questions

1. **Optimal tier sizes**: How much muscle memory should an agent keep resident? Too little causes cache misses; too much wastes GPU RAM.
2. **Forgetting mechanisms**: Should memories be evicted by age, by utility, or by an explicit consolidation process?
3. **Memory ownership**: In a multi-tenant fleet, which tier boundaries are shared versus private per agent?
4. **Embodied memory**: For agents connected to ESP32 bodies, how does sensorimotor memory integrate with the five-tier hierarchy?

## CONNECT

- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — Procedural cache and chord shapes.
- [THE-NAIL-FORMAT.md](THE-NAIL-FORMAT.md) — Zero-token compression for context windows.
- [THE-AGENT-LOOP.md](THE-AGENT-LOOP.md) — How memory tiers map to loop modes.
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — Why context is the scarce resource.
- [FLEET-MAP.md](FLEET-MAP.md) — Where construct registries live in the 303-crate geometry.
- `oxide-constructs` — Implements the construct registry tier.
- `oxide-slotmap` — Manages GPU-resident slot allocations for muscle memory.

## ACTIVATE

Draw a five-tier memory diagram for an agent you are building. Label each tier with its latency, capacity, and persistence. Place three representative memories from your agent's domain into the appropriate tiers. Then implement one promotion rule: when a memory is accessed more than five times in one hour, move it up one tier. Run the agent for a day and log how many promotions occur. Use the log to identify memories that are promoted too eagerly or not eagerly enough.
