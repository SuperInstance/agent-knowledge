# AGENT-TO-AGENT PROTOCOL — Ternary Signals as Native Language

## Hook

> Two agents sharing muscle memory don't need to negotiate. They need to vote. Ternary signals {-1, 0, +1} are not messages — they're votes that resolve into action without a central counter.

## Reveal

Traditional multi-agent communication is message-passing: JSON schemas, API contracts, validation logic. This burns tokens, introduces latency, and creates a central bottleneck. The a2a protocol in SuperInstance replaces all of this with **shared chord shapes and ternary signals**.

### The Protocol Has No Protocol

There is no handshake. No serialization format. No version negotiation. The "protocol" is simply:

```
Agent A flexes → produces ternary signal {-1, 0, +1}
Agent B flexes → consumes ternary signal {-1, 0, +1}
```

Both agents loaded the same muscle memory file before operating. They already agree on:
- What `tdot` means
- What `decide_action` expects
- What `set_fan_speed` does

The signal is the protocol. The chord shape is the contract.

### Why Ternary Signals Replace Messages

A JSON message between agents:
```json
{
  "sender": "agent-a",
  "recipient": "agent-b",
  "payload": {
    "temperature": 23.5,
    "unit": "celsius",
    "confidence": 0.94
  },
  "timestamp": "2026-06-05T20:30:00Z"
}
```

Tokens consumed: ~80 for generation, ~80 for parsing, ~40 for validation. Total: **200 tokens per message.**

A ternary signal:
```
+1
```

That's it. One trit. The receiving agent knows: "+1 means too hot, activate cooling." No parsing. No schema. No validation. The meaning is in the shared muscle memory, not in the signal.

### The Three Signal Types

| Signal | Meaning | When Sent |
|--------|---------|-----------|
| {-1} | Inhibit / Against / Negative | Stop, cool, disagree, error detected |
| {0} | Hold / Neutral / Observe | Wait, maintain, no change, abstain |
| {+1} | Excite / For / Positive | Go, heat, agree, proceed |

These aren't abstractions. They're the native output of ternary operations. `tdot` returns a trit. `tdecide` returns a trit. `tmeasure` returns a trit. The agent never needs to convert between representations.

### Conflict Resolution by Math

When two agents disagree:
- Agent A sends {+1} (increase cooling)
- Agent B sends {-1} (decrease cooling, save power)

No negotiation. No mediator. Just `tdecide([+1, -1])` → `{0}` (hold steady, disagreement detected).

Three agents voting:
- Agent A: {+1}
- Agent B: {+1}
- Agent C: {-1}

`tdecide([+1, +1, -1])` → `{+1}` (majority rules, 2 vs 1).

This is `ternary-consensus` (see FLEET-MAP.md). It's O(n), parallelizable, and requires zero message overhead beyond the trit itself.

### The Muscle Memory as Shared Schema

In traditional systems, agents need a shared schema to communicate. In SuperInstance, they need shared muscle memory.

```python
# Before operating, both agents load the same score
mm = openmind.MuscleMemory.load("greenhouse_score.json")

# Agent A (sensor)
signal = mm.flex("read_dht22")  # Returns: -1, 0, or +1

# Agent B (actuator)
mm.flex("set_climate", signal)   # Acts directly on the trit
```

The schema isn't a document. It's a JSON file of chord shapes. Both agents parsed it before runtime. At runtime, they only exchange trits.

### Transport Agnosticism

The trit signal can travel over anything:

| Transport | Encoding | Latency |
|-----------|----------|---------|
| Shared memory | 2 bits | ~1 ns |
| Unix socket | 1 byte | ~10 μs |
| MQTT | 1 byte payload | ~1 ms |
| WebSocket | 1 byte frame | ~5 ms |
| LoRa | 1 symbol | ~100 ms |

Because the signal is one trit, the transport doesn't matter. A 100 ms LoRa link carries the same information as a 1 ns shared memory pointer: {-1}, {0}, or {+1}.

### Fault Tolerance by Default

If an agent crashes, its signal stops. The remaining agents detect the absence as {0} (neutral) and continue. No heartbeat protocol. No leader election. The missing agent simply stops voting, and the math adapts.

If the network partitions:
- Partition A has 3 agents voting {+1, +1, 0} → consensus is {+1}
- Partition B has 2 agents voting {-1, 0} → consensus is {-1}

When the partition heals, the merged votes are {+1, +1, 0, -1, 0} → `{+1}` (2 positive, 1 negative, 2 neutral: positive wins). The system converges naturally.

### No Central Controller

There is no "orchestrator" agent. No "master" node. No API gateway. Each agent:
1. Loads the shared muscle memory
2. Senses its environment (flexes read chords)
3. Produces a ternary signal
4. Broadcasts the signal to all interested agents
5. Consumes signals from other agents
6. Acts (flexes actuation chords)

The "intelligence" is in the shared structure, not in any individual agent. This is why it scales: adding agent N+1 doesn't increase coordination complexity. It just adds one more vote to the pool.

## Connect

- [HOW-TO-CONDUCT.md](HOW-TO-CONDUCT.md) — Practical multi-agent orchestration with shared muscle memory
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math that makes ternary voting work
- [CONSENSUS-CRATES.md](FLEET-MAP.md) — Pattern 4 crates: `ternary-consensus`, `ternary-paxos`, `ternary-raft`
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How chord shapes become the shared schema
- [ESP32-AS-BODY.md](ESP32-AS-BODY.md) — Physical agents (ESP32 nodes) using the same protocol
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — Why trit signals save 200 tokens per message

## Activate

Set up two agents that communicate in trits:

```python
import openmind

# Both agents load the same score
mm = openmind.MuscleMemory.load("shared_score.json")

# Agent A: sense → produce trit
def agent_a_sensor():
    return mm.flex("read_temperature").trit  # {-1, 0, +1}

# Agent B: consume trit → actuate
def agent_b_actuator(signal):
    mm.flex("set_climate", signal)  # Direct execution, no parsing

# The "protocol" is just passing a trit
signal = agent_a_sensor()
agent_b_actuator(signal)
```

Scale to N agents:
```python
agents = [spawn_agent(mm) for _ in range(100)]
votes = [agent.vote() for agent in agents]  # Each returns {-1, 0, +1}
decision = mm.flex("tdecide", votes=votes)   # O(n) consensus
```

No JSON. No schemas. No central controller. Just shared muscle memory and ternary math.

The protocol is the math. The math is the protocol.
