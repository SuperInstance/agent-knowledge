# MUSCLE MEMORY FOR AGENTS

## Hook

> You don't think about how to walk. Your agent shouldn't think about how to toggle GPIO.

## Reveal

An AI agent's context window is its **conscious attention**. Every token spent on "how does this function work" is a token stolen from "what should I build next."

Muscle memory in humans works because repetitive actions get compressed into the spinal cord and cerebellum — they never reach the cortex. The agent equivalent: **compress function knowledge into indexed chord shapes that can be invoked by intent, without loading source.**

The `openmind` package does this:

```
Codebase → Ingest → Parse every function → Compress into Chord shapes → Index by intent
                                                                         ↓
                                                    Agent calls flex("spi_write", data)
                                                                         ↓
                                                    Reflex returned: {strategy: "direct", confidence: 0.9}
                                                                         ↓
                                                    Agent executes WITHOUT loading source
```

The four strategies map to neural pathways:

| Agent Decision | Neural Equivalent | Speed | Context Cost |
|---------------|-------------------|-------|--------------|
| HARDCODE | Spinal reflex | ~1ms | 0 tokens |
| CACHED | Cerebellar pattern | ~5ms | 0 tokens |
| HYBRID | Basal ganglia habit | ~50ms | ~50 tokens |
| MODEL | Prefrontal deliberation | ~2s | ~500 tokens |

**The math is brutal.** An agent with 128k context window that needs to load 6,000 function signatures (average 100 tokens each) has already burned 600k tokens — 5× its capacity. With muscle memory, it flexes by intent and spends ZERO tokens on the "how."

## Connect

This reframes the entire ecosystem:

1. **Every ternary crate** is a set of chord shapes. `ternary-core` has 58 chords. The agent doesn't read `src/lib.rs` — it flexes `tdot` and gets back the execution plan.

2. **The tripartite synchronizer** is the cerebellum. It decides which functions are muscle (HARDCODE), which are patterns (CACHED), which are habits (HYBRID), and which need conscious thought (MODEL).

3. **ESP-Flasher** is the body. The agent's "hand" — an ESP32 running firmware. The agent doesn't think about I2C clock stretching. It flexes `i2c_read` and the hand executes.

4. **cuda-oxide** is the motor cortex. GPU kernels compiled from ternary operations. The agent doesn't think about PTX registers. It flexes `ternary_matmul` and the cortex fires.

5. **The fleet at 303 crates** = 6,000 chord shapes the agent doesn't have to think about. That's 6,000 tokens of attention freed for the melody.

## Activate

### Using the API (3 lines)

```python
import openmind
result = openmind.ingest("./any-codebase")
mm = openmind.MuscleMemory.build(result)
reflex = mm.flex("do_something")  # That's it.
```

### Building for an ESP32 Fleet

```python
# One-time: ingest firmware
mm = openmind.MuscleMemory.build(openmind.ingest("./firmware"))
mm.save("esp32_muscles.json")

# At runtime: zero context burned
mm = openmind.MuscleMemory.load("esp32_muscles.json")
mm.flex("gpio_toggle", pin=2)     # HARDCODE — muscle memory
mm.flex("wifi_connect")           # HYBRID — mostly cached
mm.flex("novel_sensor_fusion")    # MODEL — needs thinking
```

### The Ah-Ha Chain

If you understood this, now read:
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — HOW the synchronizer decides what's muscle vs what needs thinking
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — The brutal math of token budgets
- [ESP32-AS-BODY.md](ESP32-AS-BODY.md) — Where the rubber meets the road
