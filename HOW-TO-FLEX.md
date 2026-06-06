# HOW TO FLEX — Using openmind's Muscle Memory API

## Hook

> Three lines of code and your agent has body memory for an entire codebase.

## Quick Start

```python
import openmind

# Step 1: Ingest a codebase (any repo, any language)
result = openmind.ingest("./my-firmware")

# Step 2: Build muscle memory
mm = openmind.MuscleMemory.build(result)

# Step 3: Flex
reflex = mm.flex("spi_write", data=b"\x01\x02")
```

That's it. The agent now has proprioception for every function in `./my-firmware`.

## The Flex Response

Every `flex()` returns a `Reflex` with:

```python
reflex.chord.name           # "spi_write"
reflex.chord.module         # "drivers.spi"
reflex.chord.signature      # "def spi_write(data: bytes) -> None"
reflex.chord.decision       # "hardcode" (muscle memory!)
reflex.chord.docstring_summary  # "Write data to SPI bus"
reflex.chord.has_tests      # True
reflex.exec_strategy        # "direct"
reflex.confidence           # 0.9
```

The agent reads `exec_strategy`:
- `"direct"` → call the function directly (muscle memory, 0 tokens)
- `"cached"` → return pre-computed result (replay, 0 tokens)
- `"generate"` → ask the LLM to generate (improvise, ~500 tokens)
- `"hybrid"` → try cache first, fall back to model

## Recall: Finding Chords

```python
# Exact match
chord = mm.recall_one("spi_write")

# Fuzzy search
chords = mm.recall("i2c", top_k=10)

# By module
chords = mm.recall("drivers.gpio")

# By intent keywords
chords = mm.recall("temperature reading")
```

Matching scores:
- 1.0: exact name match
- 0.8: query substring of name
- 0.6: keyword match
- 0.3: module match
- 0.0: no match

## Persistence: Save & Load

```python
# Build once (takes seconds)
mm = openmind.MuscleMemory.build(openmind.ingest("./firmware"))
mm.save("firmware_muscles.json")

# Load instantly later (zero compute)
mm = openmind.MuscleMemory.load("firmware_muscles.json")
mm.flex("spi_write", data=b"\x01")  # Immediate
```

## Multi-Repo: The Orchestra Pattern

```python
# Build muscle memory for each node
mm_sensor = openmind.MuscleMemory.build(openmind.ingest("./sensor-firmware"))
mm_motor = openmind.MuscleMemory.build(openmind.ingest("./motor-firmware"))
mm_light = openmind.MuscleMemory.build(openmind.ingest("./light-firmware"))

# Conduct
reading = mm_sensor.flex("read_temperature")
if reading.chord.decision == "hardcode":
    temp = execute(reading)
    action = decide(temp)
    mm_motor.flex("set_speed", speed=action)
    mm_light.flex("set_color", color=temp_to_color(temp))
```

## CLI: Quick Exploration

```bash
# Ingest and see what you've got
openmind ingest ./my-firmware

# Flex a specific function
openmind flex ./my-firmware "spi_write"

# Search for functions
openmind recall ./my-firmware "gpio"

# Save for later
openmind save ./my-firmware muscles.json

# Check your muscle memory stats
openmind stats muscles.json
```

## Jupyter: Visual Exploration

```python
%load_ext openmind.jupyter

%%openmind analyze ./my-firmware
# Rich HTML dashboard: function count, decision breakdown, test coverage

%%openmind recall spi
# Matching functions in a table

%%openmind flex gpio_toggle
# Execution strategy card
```

## Common Patterns

### Pattern 1: Pre-Build for Production

```python
# At build time: ingest all repos, save muscle memories
for repo in glob("firmware/*/"):
    name = os.path.basename(repo)
    mm = openmind.MuscleMemory.build(openmind.ingest(repo))
    mm.save(f"muscles/{name}.json")

# At runtime: load what you need
mm = openmind.MuscleMemory.load("muscles/sensor-node.json")
```

### Pattern 2: Dynamic Ingestion

```python
# Agent discovers a new device
new_device = discover_device()
result = openmind.ingest(f"https://github.com/firmware/{new_device}")
mm = openmind.MuscleMemory.build(result)
mm.flex("identify")  # What can this device do?
```

### Pattern 3: Chord Chain

```python
# Agent chains chords together without thinking
steps = ["read_sensor", "normalize", "classify", "actuate"]
for step in steps:
    reflex = mm.flex(step)
    result = execute(reflex)
```

## Connect

- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — Theory behind the API
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — How decisions are made
- [HOW-TO-CONDUCT.md](HOW-TO-CONDUCT.md) — Multi-agent orchestration
