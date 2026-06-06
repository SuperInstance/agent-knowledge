# HOW TO CONDUCT — Orchestrating Multiple Agents with Shared Muscle Memory

## Hook

> A conductor doesn't play every instrument. They make every instrument play together.
> Your agent should conduct, not execute.

## The Orchestra Model

Imagine 12 ESP32s scattered around a workshop. Each one has different capabilities — temperature sensors, motors, lights, speakers, displays. Your agent needs to coordinate all of them.

Without a conductor: the agent tries to control each device individually, burning context on each one's API. 12 devices × 200 tokens each = 2,400 tokens just to know what's available.

With a conductor: each device has its own muscle memory. The agent sends high-level intents. Each device flexes its own chords. The conductor's context cost: **zero tokens per device**.

### The Three Roles

```
┌─────────────┐
│  CONDUCTOR   │  ← Your agent (high-level intents)
│  (prefrontal)│
└──────┬───────┘
       │ baton (MQTT/WebSocket/serial)
       ├─── Agent 1: Sensors  (feel)
       ├─── Agent 2: Motors   (move)
       ├─── Agent 3: Lights   (express)
       ├─── Agent 4: Sound    (speak)
       ├─── Agent 5: Display  (show)
       └─── Agent 6: Network  (communicate)
```

**Conductor**: Holds the score. Sends intents. Doesn't know HOW to read a sensor or drive a motor — just knows WHEN each should happen.

**Agent**: Holds muscle memory for its domain. Receives intents. Flexes chord shapes. Doesn't know WHY it's doing something — just knows HOW.

**Baton**: The communication channel. Could be MQTT (network), WebSocket (browser), serial (USB), or in-process channels (local).

## Building a Score

A score is a pre-computed plan — like sheet music for the orchestra:

```python
from openmind_conductor import Score, Step, Condition

score = Score.builder() \
    .step("sensor", "read_temperature", [], immediate=True) \
    .step("sensor", "read_humidity", [], immediate=True) \
    .branch(
        Condition.last("sensor", "read_temperature") > 25.0,
        then=[
            Step("motor", "fan_on", [Trit.PLUS_ONE]),
            Step("light", "set_color", ["red"]),
        ],
        otherwise=[
            Step("motor", "fan_off", [Trit.ZERO]),
            Step("light", "set_color", ["cool_blue"]),
        ]
    ) \
    .step("display", "show_status", ["monitoring"], after_ms=500) \
    .build()
```

This says:
1. Read temperature and humidity simultaneously
2. If temp > 25°C: turn on fan, set lights red
3. Else: turn off fan, set lights blue
4. After 500ms: update display

The conductor doesn't know how to read a DHT22 or drive a fan. It just follows the score.

## Running the Orchestra

```python
from openmind_conductor import Ensemble, Baton

# Load muscle memories
ensemble = Ensemble()
ensemble.add("sensor", MuscleMemory.load("sensor_muscles.json"))
ensemble.add("motor", MuscleMemory.load("motor_muscles.json"))
ensemble.add("light", MuscleMemory.load("light_muscles.json"))
ensemble.add("display", MuscleMemory.load("display_muscles.json"))

# Connect batons (communication channels)
ensemble.connect_baton("sensor", Baton.serial("/dev/ttyUSB0"))
ensemble.connect_baton("motor", Baton.serial("/dev/ttyUSB1"))
ensemble.connect_baton("light", Baton.mqtt("lights/control"))
ensemble.connect_baton("display", Baton.websocket("ws://display.local"))

# Conduct the score
result = ensemble.perform(score)
print(f"Completed in {result.duration_ms}ms")
print(f"All steps: {result.all_succeeded}")
```

## The Ternary Conductor

Because the fleet uses {-1, 0, +1} everywhere, the conductor's language is ternary:

- **-1**: Stop, cancel, reject, cool down, close
- **0**: Hold, wait, maintain, neutral
- **+1**: Start, approve, accept, heat up, open

```python
# Fan control
ensemble.flex("motor", "fan_speed", [Trit.PLUS_ONE])   # Full speed
ensemble.flex("motor", "fan_speed", [Trit.ZERO])       # Off
ensemble.flex("motor", "fan_speed", [Trit.MINUS_ONE])  # Reverse

# Decision
ensemble.flex("display", "show_vote", [Trit.PLUS_ONE])  # "YES"
ensemble.flex("display", "show_vote", [Trit.MINUS_ONE]) # "NO"
ensemble.flex("display", "show_vote", [Trit.ZERO])      # "ABSTAIN"
```

## Harmony: When Agents Disagree

What happens when three temperature sensors disagree?

```python
from openmind_conductor import Harmony

# Three sensors, three readings
readings = [22.1, 23.4, 19.8]

# Ternary voting: is it too hot?
votes = [
    Trit.from_bool(readings[0] > 25.0),  # -1 (no)
    Trit.from_bool(readings[1] > 25.0),  # -1 (no)
    Trit.from_bool(readings[2] > 25.0),  # -1 (no)
]

consensus = Harmony.majority(votes)  # -1 (not too hot)
action = ensemble.decide_from_vote(consensus)
# → fan_off, lights_cool
```

Three voting strategies:
- **Majority**: most votes win (2 of 3)
- **Consensus**: ALL must agree (unanimous)
- **Weighted**: trusted sensors count more

## The Learning Loop

```
Compose → Perform → Observe → Calibrate → Compose
   ↑                                         ↓
   └───────── The agent improves ─────────────┘
```

1. **Compose**: Write a score (or generate one from intent)
2. **Perform**: Execute through the orchestra
3. **Observe**: Watch results via Proprioceptor (openmind-mirror)
4. **Calibrate**: Adjust decisions based on outcomes
5. **Compose**: Write a better score next time

Over time, the conductor learns:
- Which agents are reliable (high confidence scores)
- Which scores work well (low error rates)
- Which patterns to avoid (high dissent rates)

## Connect

- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — Individual agent muscle memory
- [ESP32-AS-BODY.md](ESP32-AS-BODY.md) — The physical instruments
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — When each agent should think vs act
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How chord shapes work

## Activate

The conductor crate: https://github.com/SuperInstance/openmind-conductor
The bridge crate: https://github.com/SuperInstance/openmind-esp32-bridge

Start simple: one sensor, one actuator. Then add instruments one at a time.
