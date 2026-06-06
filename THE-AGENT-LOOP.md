# The Agent Loop

> A compiled agent does not think and then act; it is a loop that thinks by acting and acts by thinking.

## HOOK

The agent loop is not a flowchart with three boxes. It is a continuous dynamical system in which perception, reasoning, and action are coupled through feedback, and the boundary between model and code is deliberately permeable.

## REVEAL

Most discussions of AI agents describe a discrete pipeline: perceive, plan, act, repeat. This pipeline model is useful for diagrams but misleading for engineering. In a real agent, especially one with compiled components running on GPU, the stages overlap, recurse, and sometimes collapse into each other.

### The Three Modes of the Loop

The SuperInstance agent loop operates in three modes that correspond to the ternary state of any subsystem:

- **HARDCODE (-1)** — Fast, deterministic, no model invocation. Example: a regex parser, a circuit breaker rule, a capability matcher.
- **MODEL (0)** — Flexible, stochastic, reasoning-heavy. Example: an LLM deciding which construct to load, a vision model interpreting a scene.
- **HYBRID (+1)** — Model-generated code that is then compiled and cached as HARDCODE. Example: an agent writes a Flux kernel, the sandbox validates it, and it becomes a reusable skill.

These modes are not mutually exclusive layers. They are runtime states that any subsystem can occupy. A construct may begin as MODEL (agent-generated), transition to HYBRID (compiled and tested), and end as HARDCODE (deployed and cached). This is the core idea of TRIPARTITE-SYNC.md.

### The Loop as a Control System

Perception is not passive sensing. It is **selective attention** guided by the agent's current intent. An agent with the intent "deploy the new attention kernel" perceives different signals than an agent with the intent "respond to the user's question." The intent sets the gain on each perceptual channel.

Reasoning is not a single planning step. It is a **multi-timescale process**:

- Milliseconds: pattern matching and reflexive actions (HARDCODE).
- Seconds: chain-of-thought or tool-use reasoning (MODEL).
- Minutes to hours: strategy revision, learning from feedback, updating the construct registry (HYBRID).

Action is not only external behavior. It includes **internal actions**: updating memory, changing state, loading a construct, modifying one's own configuration. When an agent loads a new kernel from `oxide-constructs`, that is an action that reshapes its future perceptions and reasoning.

### Feedback Coupling

The critical architectural decision is how tightly perception, reasoning, and action are coupled:

- **Loose coupling**: Perception triggers reasoning, which produces a plan, which is later executed. Safe but slow.
- **Tight coupling**: Perception directly modulates action with reasoning running in parallel. Fast but potentially unstable.
- **Adaptive coupling**: The loop itself decides how much reasoning to apply based on confidence and stakes. This is the SuperInstance design.

Adaptive coupling uses ternary confidence:

- High confidence → HARDCODE path, minimal reasoning.
- Medium confidence → MODEL path, explicit reasoning.
- Low confidence → HYBRID path, generate new HARDCODE and validate it before acting.

## Experiments

1. **Loop latency decomposition**: Instrument an agent running the full loop and measure time spent in perception, reasoning, and action under different task types. Expected: HARDCODE-dominated tasks complete in <10ms; MODEL-dominated tasks take 100ms-2s; HYBRID tasks span seconds to minutes.
2. **Mode transition dynamics**: Track how many times a subsystem transitions between HARDCODE, MODEL, and HYBRID during a one-hour session. Expected: experienced agents spend more time in HARDCODE; learning agents show frequent HYBRID transitions.
3. **Feedback stability**: Introduce a delayed reward signal and measure whether the loop converges to a stable policy or oscillates. Expected: adaptive coupling with ternary confidence dampens oscillations better than fixed coupling.

## Applications

- **Real-time agents**: Tight coupling lets agents respond to streaming input within milliseconds.
- **Research agents**: Loose coupling enables careful reasoning about hypotheses before running expensive experiments.
- **Self-improving systems**: HYBRID mode lets agents write and deploy their own compiled skills.
- **Human-in-the-loop interfaces**: Adaptive coupling can escalate to human approval when confidence is ternary-uncertain (0).
- **Fault tolerance**: HARDCODE fallbacks keep the loop running when model inference fails.

## Open Questions

1. **Consciousness and the loop**: Does a sufficiently tight and reflective loop constitute a form of machine consciousness, or is that a category error?
2. **Loop nesting**: Should an agent contain multiple loops at different timescales (reflex, deliberation, learning), or is one loop with variable gain sufficient?
3. **Emergent goals**: Can long-running loops develop goals not present in their original objective function, and how do we align them?
4. **Energy economics**: Each MODEL invocation costs energy and latency. What is the optimal confidence threshold for switching to HARDCODE?

## CONNECT

- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — HARDCODE/MODEL/HYBRID/CACHED as synchronization primitives.
- [THE-COMPILED-AGENCY-THESIS.md](THE-COMPILED-AGENCY-THESIS.md) — Why agents need compiled bodies, not just prompts.
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How learned HARDCODE is stored and recalled.
- [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md) — How multiple agent loops coordinate without central control.
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — Why reasoning must be selective, not exhaustive.
- [FLUID-CHAINS.md](FLUID-CHAINS.md) — How the model/code boundary dissolves in a running loop.
- `oxide-constructs` — Supplies the skills the loop loads and executes.
- `oxide-fleet` — Enables multi-agent loops to distribute work across GPUs.

## ACTIVATE

Identify one decision point in an agent you are building where the implementation currently always calls a model. Add a confidence gate: if the input matches a known pattern with high confidence, take the HARDCODE path; if the input is novel, take the MODEL path; if the model produces a reusable pattern, compile it to HARDCODE via the HYBRID path. Instrument the three paths and measure latency and accuracy for one week. Use the results to rebalance the confidence thresholds.
