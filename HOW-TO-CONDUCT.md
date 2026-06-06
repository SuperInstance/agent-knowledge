# HOW TO CONDUCT — Orchestrating Multiple Agents With Shared Muscle Memory

## Hook

> A conductor doesn't play every instrument — they create a shared score that every musician interprets. Multi-agent orchestration isn't about controlling agents; it's about giving them the same muscle memory so they harmonize without a central brain.

## Reveal

Most multi-agent systems are orchestrated like marionettes: a central controller pulls strings, agents react. This doesn't scale. When you have 10 agents, the controller becomes the bottleneck. When you have 100, it's impossible.

SuperInstance uses **score-based orchestration** instead. Every agent reads from the same muscle memory. They don't need to coordinate with each other because they already know the same chord shapes.

### The Symphony Architecture

```
Shared Muscle Memory (the score)
       ↓
  ┌────┴────┐
Agent A   Agent B   Agent C   ...
(sensor)  (planner) (actuator)
       ↓       ↓       ↓
   flex()  flex()  flex()
       ↓       ↓       ↓
   GPIO    tdot()   PWM
```

Agent A reads a temperature sensor. It flexes `read_dht22` from the shared muscle memory. The result is a ternary signal: {-1: too cold, 0: just right, +1: too hot}.

Agent B is the planner. It receives the ternary signal and flexes `decide_action` from the same muscle memory. It doesn't need to understand DHT22 protocols. It only understands {-1, 0, +1}.

Agent C is the actuator. It receives the action and flexes `set_fan_speed` or `set_heater` from the shared muscle memory. It doesn't need to understand planning. It only understands the chord shape.

### Why Shared Muscle Memory Replaces Message Passing

Traditional multi-agent orchestration:
```python
# Agent A sends a message
agent_b.send({"temperature": 23.5, "unit": "celsius", "timestamp": ...})

# Agent B parses it, validates it, converts it
raw = agent_a.receive()
temp = parse_temperature(raw)  # Error-prone
action = plan(temp)             # Context-heavy
agent_c.send(action)
```

Score-based orchestration:
```python
# All agents load the same score
mm = openmind.MuscleMemory.load("greenhouse_score.json")

# Agent A
signal = mm.flex("read_dht22")  # Returns: -1, 0, or +1

# Agent B
action = mm.flex("decide_action", signal)  # Already ternary, no parsing

# Agent C
mm.flex("set_fan_speed", action)  # Direct execution
```

The difference:
- **No serialization.** Ternary signals are the native language.
- **No parsing.** Every agent knows the chord shapes.
- **No context waste.** No JSON schemas, no validation logic, no error handling.
- **No central controller.** Agents coordinate through shared structure, not messages.

### The Conductor's Role

The conductor doesn't make music. The conductor ensures:
1. **Every agent has the same score.** Muscle memory files are distributed before any operation.
2. **Tempo is synchronized.** The shared `open-parallel` scheduler provides a global clock.
3. **Dynamics are balanced.** The tripartite synchronizer ensures no agent burns MODEL tokens on what should be HARDCODE.

```python
from openmind import Conductor, MuscleMemory

# Load the score
score = MuscleMemory.load("orchestra_score.json")

# Create the conductor
conductor = Conductor(score=score, tempo_ms=100)

# Register sections (agents)
conductor.add_section("sensors", agents=[agent_a, agent_d])
conductor.add_section("planners", agents=[agent_b])
conductor.add_section("actuators", agents=[agent_c, agent_e, agent_f])

# Start the performance
conductor.start()
```

The conductor runs a loop:
1. **Sense phase:** All sensor agents flex their reading chords
2. **Think phase:** All planner agents flex their decision chords on the ternary signals
3. **Act phase:** All actuator agents flex their action chords
4. **Sync:** Global barrier, next measure

Each phase is ternary:
- {-1}: Abort / emergency stop
- {0}: Hold / no change
- {+1}: Proceed / execute

### Conflict Resolution Without a Controller

What if two agents want to set the fan to different speeds? In marionette systems, the controller decides. In score-based systems, the ternary math decides:

```python
# Agent B says: +1 (increase cooling)
# Agent G says: -1 (decrease cooling, save power)
# The score contains a consensus chord:
result = mm.flex("ternary_vote", votes=[+1, -1])
# Returns: 0 (hold steady — disagreement detected)
```

The `ternary-consensus` crate (Pattern 4) provides the chord shapes for multi-agent agreement. The agents don't negotiate. They vote, and the math resolves it. This scales to any number of agents because ternary voting is O(n) and can be parallelized.

### Fault Tolerance

If an agent crashes, the others don't notice. The score is shared. The tempo is global. The missing agent's section simply produces a {0} (neutral) signal, and the symphony continues with reduced instrumentation.

If the conductor crashes, the agents can self-conduct using the shared scheduler. The score contains all the information needed for synchronization.

## Connect

- [ESP32-AS-BODY.md](ESP32-AS-BODY.md) — Each ESP32 is an instrument; this document is about the orchestra
- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — The fundamental API that every agent in the orchestra uses
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — How the synchronizer ensures no agent wastes tokens on reflexes
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — The shared score: how chord shapes become collective knowledge
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — Why ternary voting resolves conflicts without a central authority

## Activate

Build a two-agent orchestra:

```python
import openmind

# Step 1: Create a shared score
score = openmind.MuscleMemory.build(openmind.ingest("./greenhouse-firmware"))
score.save("greenhouse_score.json")

# Step 2: Distribute to agents
# Agent A (sensor node)
mm_a = openmind.MuscleMemory.load("greenhouse_score.json")

# Agent B (actuator node)
mm_b = openmind.MuscleMemory.load("greenhouse_score.json")

# Step 3: Run the sense-act loop
while True:
    signal = mm_a.flex("read_dht22")  # Returns -1, 0, or +1
    mm_b.flex("set_climate", signal)   # Acts on the ternary signal
    # No JSON. No parsing. No controller. Just shared muscle memory.
```

To scale to N agents:
1. Build the score once
2. Distribute `score.json` to every agent
3. Each agent flexes only the chords in its section
4. Ternary signals propagate through the shared namespace

The orchestra doesn't need a conductor. It needs a score.
