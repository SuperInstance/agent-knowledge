# FLUID CHAINS — Every Node Decides Its Own Strategy

## Hook

> The consumer doesn't know whether the producer was an LLM, a cached file, a compiled function, or a sensor reading.
> It just gets a result. The boundary between model and code was never real.

## Reveal

In a traditional pipeline, each stage has a fixed implementation: "this stage uses the model" or "this stage is hardcoded." You choose at design time and you're stuck.

In a **fluid chain**, every node independently chooses its strategy at runtime:

```
Input ──→ Parse ──→ Transform ──→ Decide ──→ Act ──→ Output
 │         │           │           │         │
MODEL    HARDCODE     CACHED      MODEL    HYBRID
```

The data contracts are stable. The implementations are fluid.

### The Principle

**Every function that produces an output can independently choose HOW to produce it:**
- **HARDCODE**: Compiled, deterministic, sub-millisecond. The function knows its output.
- **CACHED**: Pre-computed, identical to last time. The function remembers its output.
- **HYBRID**: Try the cache, verify with model or sensor. The function checks its output.
- **MODEL**: Ask the LLM, think creatively, generate novel response. The function invents its output.

**The downstream function receives the same data structure regardless of which strategy produced it.**

### Why This Works

1. **Data contracts are stable.** A ternary signal is {-1, 0, +1} whether it came from a Rust function, an LLM, a cached file, or a physical sensor.

2. **Decisions are local.** Each node runs its own tripartite evaluation. No central planner needed.

3. **Adaptation is automatic.** When the GPU goes away, MODEL nodes switch to CACHED. When novel data arrives, CACHED nodes switch to MODEL. The chain never stops.

4. **Calibration is continuous.** openmind-mirror watches every node's outputs. If HARDCODE starts producing errors, it downgrades to HYBRID. If MODEL produces 100 perfect outputs, it upgrades to HARDCODE.

### Not Just ESP32 — Anywhere on the Chain

The muscle memory vision started with ESP32s (hardware as hands). But the fluid chain applies to EVERY node:

- **LLM layer**: "Generate response" can be HARDCODE (template), CACHED (previous answer), or MODEL (fresh generation)
- **Database layer**: "Query results" can be HARDCODE (compiled query), CACHED (query cache), or MODEL (LLM-generated SQL)
- **API layer**: "External data" can be HARDCODE (well-known endpoint), CACHED (recent response), or MODEL (novel endpoint discovery)
- **Control layer**: "Actuator command" can be HARDCODE (deterministic), HYBRID (sensor-verified), or MODEL (novel situation)
- **Output layer**: "Final result" can be HARDCODE (known format), CACHED (previous format), or MODEL (adaptive formatting)

Every single one of these can switch strategies without breaking the chain. The data flows forward. The strategy stays local.

### Implementation

```python
from openmind import MuscleMemory, flex

# Every function in the chain is a chord shape
mm = MuscleMemory.load("pipeline_muscles.json")

# The chain — each flex() independently chooses strategy
parsed = mm.flex("parse_input", data=raw_data)      # Maybe HARDCODE
transformed = mm.flex("transform", data=parsed)       # Maybe CACHED
decision = mm.flex("decide", data=transformed)         # Maybe MODEL
action = mm.flex("actuate", decision=decision)         # Maybe HYBRID

# The consumer never knows which strategy each node used
# It just gets a result that satisfies the data contract
```

## Connect

- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How chord shapes enable fluid switching
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — The decision engine for each node
- [CELLULAR-JUPYTER.md](CELLULAR-JUPYTER.md) — Resource-adaptive computation at notebook scale
- [FIVE-LAYER-ARCHITECTURE.md](FIVE-LAYER-ARCHITECTURE.md) — Where fluid chains live in the stack

## Activate

The fluid chain is already in openmind. Every `flex()` call is a node making an independent decision:

```python
import openmind
mm = openmind.MuscleMemory.build(openmind.ingest("./your-pipeline"))
result = mm.flex("any_function")  # HARDCODE? CACHED? MODEL? The node decides.
```
