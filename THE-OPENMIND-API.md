# THE OPENMIND API — The Unifying Layer That Binds the Stack

## Hook

> You don't need to know 303 crates. You need to know one API. openmind is the spinal cord that connects the agent's intent to every muscle in the fleet.

## Reveal

The SuperInstance ecosystem has five layers, seven patterns, and 303 crates. That's too much for any agent to hold in working memory. openmind solves this by providing **a single surface** — one Python API that compresses the entire stack into chord shapes.

### What openmind Actually Is

openmind is three things in one package:

1. **The Ingestion Engine** (`openmind.ingest`) — turns any codebase into chord shapes
2. **The Muscle Memory Store** (`openmind.MuscleMemory`) — indexes and retrieves chord shapes
3. **The Conductor** (`openmind.Conductor`) — orchestrates multi-agent workflows

These three modules map to the three scales of conservation (see CONSERVATION-LAWS.md):
- Ingestion creates verification entropy
- Muscle memory conserves attention
- Conductor preserves ternary structure across agents

### The API Surface

```python
import openmind

# === INGESTION ===
# Turn any codebase into structured chord shapes
result = openmind.ingest("./my-firmware")
# result.chords: list of compressed function signatures
# result.stats: coverage, language breakdown, complexity metrics
# result.graph: call graph as adjacency list

# === MUSCLE MEMORY ===
# Build, save, load, and flex
mm = openmind.MuscleMemory.build(result)
mm.save("my_muscles.json")
mm = openmind.MuscleMemory.load("my_muscles.json")

reflex = mm.flex("spi_write", data=b"\x01")
# reflex.chord: the matched chord shape
# reflex.exec_strategy: "direct" | "cached" | "generate" | "hybrid"
# reflex.confidence: 0.0 to 1.0

# === CONDUCTOR ===
# Multi-agent orchestration through shared scores
conductor = openmind.Conductor(score=mm, tempo_ms=100)
conductor.add_section("sensors", agents=[agent_a, agent_d])
conductor.add_section("actuators", agents=[agent_c])
conductor.start()  # Sense → Think → Act loop
```

That's the entire API. Three entry points. Everything else is implementation detail.

### How It Wraps the Rust Core

The Python API is a thin wrapper over Rust libraries:

| Python Call | Rust Library | What It Does |
|-------------|--------------|--------------|
| `openmind.ingest()` | `openmind-induction` | Parse, analyze, compress |
| `MuscleMemory.build()` | `openmind-memory` | Index chord shapes |
| `mm.flex()` | `openmind-synchronizer` | Tripartite decision + execution |
| `Conductor.start()` | `openmind-conductor` | Async orchestration |

The Rust core handles parsing (tree-sitter), indexing (sled/rocksdb), and synchronization (tokio async). The Python layer adds ergonomic APIs, Jupyter magics, and JSON serialization.

### The One-Line Principle

Every common operation should be one line:

```python
# One line: ingest a repo
result = openmind.ingest("./firmware")

# One line: build muscle memory
mm = openmind.MuscleMemory.build(result)

# One line: flex a capability
reflex = mm.flex("read_temperature")

# One line: orchestrate N agents
conductor = openmind.Conductor(score=mm).start()
```

If an operation takes more than one line, it's either:
- Advanced usage (custom synchronizer profiles)
- A sign that the API needs improvement
- Something that belongs in a HOW-TO doc, not the core API

### The Score Format

Muscle memory files (`.json`) are the lingua franca between agents:

```json
{
  "crate": "ternary-core",
  "version": "0.47.1",
  "chords": [
    {
      "name": "tdot",
      "module": "ternary_core::math",
      "signature": "pub fn tdot(a: &[Trit], b: &[Trit]) -> Trit",
      "decision": "hardcode",
      "docstring": "Ternary dot product in Z₃",
      "has_tests": true,
      "quality_score": 0.98
    }
  ]
}
```

Any agent that can read JSON can load a score. Any agent that can load a score can flex its chords. The format is trivial because the intelligence is in the chord shapes, not in the serialization.

### Why This Isn't "Just Another Framework"

Most ML/agent frameworks are monolithic: you import the framework, you do things the framework's way. openmind is different:
- It doesn't replace your code. It **describes** your code.
- It doesn't dictate architecture. It **indexes** whatever architecture you have.
- It doesn't run on a server. It runs **in your process**, next to your agent.

openmind is a lens, not a platform. You point it at any codebase — Rust, Python, C, JavaScript — and it shows you the chord shapes. The codebase doesn't need to know openmind exists.

## Connect

- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — Practical examples of the MuscleMemory API
- [HOW-TO-INGEST.md](HOW-TO-INGEST.md) — Deep dive on the ingestion engine
- [HOW-TO-CONDUCT.md](HOW-TO-CONDUCT.md) — Multi-agent orchestration with the Conductor
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — How `mm.flex()` decides execution strategy
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — Theory behind chord shapes and compression
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — How openmind embodies all three conservation laws

## Activate

Install and use in 30 seconds:

```bash
pip install openmind
```

```python
import openmind

# Ingest any repo
mm = openmind.MuscleMemory.build(openmind.ingest("."))

# Flex any function
reflex = mm.flex("main")
print(reflex.chord.signature)
print(reflex.exec_strategy)  # "direct" if tested, "generate" if novel

# Save for later
mm.save("my_project.json")
# Load elsewhere
mm = openmind.MuscleMemory.load("my_project.json")
```

To explore the full API:
```python
import openmind
help(openmind.MuscleMemory)
help(openmind.Conductor)
```

The agent doesn't need to understand 303 crates. It needs to understand one API. The fleet is already inside.
