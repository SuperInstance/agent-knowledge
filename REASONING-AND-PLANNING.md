# Reasoning and Planning

> Reasoning is not the ability to answer hard questions. It is the ability to know which questions are worth asking.

## HOOK

In a compiled agency, reasoning is not a monolithic capability that lives inside a large language model. It is a layered stack in which fast heuristics, structured search, and generative models trade off accuracy, latency, and cost according to the stakes of the decision.

## REVEAL

The classical AI planning literature distinguishes between:

- **Deliberative planning**: Search through a model of the world to find a sequence of actions that achieves a goal.
- **Reactive planning**: Select actions based on current percepts without explicit search.
- **Hierarchical planning**: Decompose goals into subgoals, plan each subgoal, and compose the results.

In the SuperInstance ecosystem, these three styles are not alternatives. They are runtime modes that the agent selects based on a ternary confidence assessment:

- `+1` high confidence → Reactive (HARDCODE) path.
- `0` uncertain → Deliberative (MODEL) path.
- `-1` novel or high stakes → Hierarchical (HYBRID) path that generates new HARDCODE.

### The Reasoning Stack

The reasoning stack has four layers, ordered by latency:

1. **Reflexive retrieval**: Pattern match against muscle memory. Latency: microseconds. No model call.
2. **Local inference**: Run a small model or a chain-of-thought prompt on the current context. Latency: milliseconds to seconds.
3. **Structured search**: Enumerate action sequences, evaluate them against a world model, and select the best. Latency: seconds to minutes.
4. **Generative synthesis**: Ask a large model to invent a new plan, policy, or representation. Latency: seconds to hours.

An agent does not always ascend through all four layers. It starts at layer 1 and escalates only when confidence is insufficient. This is the adaptive coupling principle from THE-AGENT-LOOP.md.

### Planning as Program Synthesis

A plan is a program whose primitives are agent capabilities: call a construct, send a message, move an actuator, query memory. The planning problem is therefore a program synthesis problem:

- Given: a goal state, an initial state, a library of capabilities.
- Find: a program that transforms the initial state into the goal state.

The synthesis can be performed by:

- **Retrieval**: The plan already exists in muscle memory.
- **Search**: Enumerate programs up to a depth bound and test them.
- **Generation**: Ask a model to write a program, then validate it in `oxide-sandbox`.
- **Hybrid**: Retrieve a partial plan, generate the missing pieces, and search for the right insertion points.

### Uncertainty and Abstraction

A key skill in reasoning is knowing the right level of abstraction. If the goal is "make coffee," the plan should not reason about individual neurons. If the goal is "debug this kernel," the plan should not reason about the user's morning routine.

The agent maintains an **abstraction tower**: a hierarchy of representations from raw sensor data to high-level intent. Planning selects the level where the action space is small enough to search but expressive enough to solve the problem. When no such level exists, the agent creates a new abstraction through HYBRID synthesis.

## Experiments

1. **Layer selection accuracy**: Present 100 tasks to an agent and record which reasoning layer it selects. Have humans label the optimal layer. Measure agreement. Expected: >80% agreement with a well-tuned confidence model.
2. **Plan synthesis latency**: Compare time to generate a plan by retrieval, search, generation, and hybrid methods across five task categories. Expected: retrieval <1ms, search seconds to minutes, generation seconds to minutes, hybrid varies.
3. **Abstraction effectiveness**: Measure planning success rate at different abstraction levels for the same task. Expected: too low → intractable search; too high → misses details; an intermediate level maximizes success.
4. **Failure recovery**: Inject a random capability failure mid-execution and measure how often the agent replans successfully. Expected: near 100% with explicit world model; <50% with pure reactive policy.

## Applications

- **Autonomous coding**: Plan software changes as programs that call IDEs, compilers, and test runners.
- **Scientific experimentation**: Design multi-step experiments, reason about dependencies, and recover from failed trials.
- **Robotic task planning**: Decompose household or industrial tasks into sequences of safe motor commands.
- **Conversational agents**: Plan responses that achieve conversational goals without being manipulative.
- **Strategic gameplay**: Search game trees and synthesize novel strategies for competitive environments.

## Open Questions

1. **Reasoning transparency**: How much of the reasoning process should be exposed to users, and in what form?
2. **Common-sense grounding**: Can reasoning over symbolic plans ever be sufficient without embodied interaction?
3. **Value alignment**: When a plan affects multiple stakeholders, whose preferences should the planner optimize?
4. **Computational irreducibility**: Are there domains where no amount of reasoning can predict outcomes better than trying and observing?

## CONNECT

- [THE-AGENT-LOOP.md](THE-AGENT-LOOP.md) — Where reasoning sits in the perception-action cycle.
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — Cached plans and reflexive retrieval.
- [THE-COMPILED-AGENCY-THESIS.md](THE-COMPILED-AGENCY-THESIS.md) — Why reasoning needs compiled bodies.
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — The cost of model-based reasoning.
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — When to reason in HARDCODE, MODEL, or HYBRID mode.
- `oxide-sandbox` — Validates generated plans before execution.
- `oxide-constructs` — Stores reusable plans and capabilities.

## ACTIVATE

Pick a recurring task in your domain and write three plans for it at different abstraction levels: one with 3 steps, one with 10 steps, and one with 50 steps. Run each plan through an agent and record success rate, execution time, and failure mode. Identify the level that achieves the highest success rate per unit time. Then convert the best plan into a HARDCODE construct and measure the speedup on the next 100 executions.
