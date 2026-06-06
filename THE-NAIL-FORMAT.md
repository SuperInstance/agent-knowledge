# THE NAIL FORMAT — Caching Consciousness in Zero Tokens

## Hook

> A cached result isn't a shortcut. It's a memory — a preserved moment of thought that the agent can relive without paying the price again.

## Reveal

The CACHED decision strategy (see TRIPARTITE-SYNC.md) stores pre-computed results so the agent never recomputes what it already knows. The `.nail` file format is how those memories are preserved.

### Why "Nail"?

A nail is permanent, small, and holds things together. A `.nail` file is permanent (written to disk), small (binary format, no metadata bloat), and holds the agent's consciousness together (without it, every invocation costs 500 tokens).

### The Format

```
NAIL v1
======
[Header: 16 bytes]
  magic:       "NAIL" (4 bytes)
  version:     u8    (1 byte)
  reserved:    3 bytes
  entry_count: u32   (4 bytes)
  checksum:    u32   (4 bytes)

[Entries: variable]
  For each cached result:
    key_hash:    u64   (hash of input parameters)
    output_size: u32   (bytes in output)
    output_type: u8    (0=i32, 1=f32, 2=trit, 3=bytes, 4=JSON)
    flags:       u8    (0=valid, 1=stale, 2=expired)
    reserved:    2 bytes
    output_data: [output_size bytes]
```

Total overhead per entry: 16 bytes. For a cached ternary dot product result (4 bytes), that's 20 bytes total. Compare to recomputing via MODEL (500 tokens ≈ 2KB of context). The cache is **100× smaller** than the computation.

### When to Nail

The tripartite synchronizer decides CACHED when:
- The function is deterministic (same input → same output)
- The input space is small enough to cache meaningfully
- The function is called frequently (>100x/day)
- The output is expensive to compute but cheap to store

Examples from the fleet:

| Function | Output | Cache Hit Rate | Tokens Saved |
|----------|--------|---------------|--------------|
| `tdot` (fixed vectors) | Trit | 99% | 500 |
| `pack_20` | u64 | 100% | 300 |
| `tfilter_lowpass` (fixed kernel) | Vec<Trit> | 95% | 2000 |
| `tdetect_tempo` (same audio) | BPM | 90% | 5000 |

### The Nail Lifecycle

```
First invocation:
  Input → MODEL computes → Result → Store in .nail

Second invocation:
  Input → Hash lookup → Cache hit → Return result
  Cost: 0 tokens, ~5μs

Cache invalidation:
  Source code changes → Hash of function changes → All nails for that function invalidated
  Or: Explicit `mm.invalidate("function_name")`
  Or: Time-based expiry (TTL in header flags)
```

### Nails and Muscle Memory

Muscle memory (`.json`) stores WHAT the agent can do. Nails (`.nail`) stores WHAT THE AGENT HAS ALREADY DONE. Together they form the agent's complete state:

```python
mm = openmind.MuscleMemory.load("agent_muscles.json")
mm.load_nails("agent_cache.nail")

# First call: computes and caches
result = mm.flex("expensive_analysis", data=input_a)

# Second call: instant replay
result = mm.flex("expensive_analysis", data=input_a)  # 0 tokens
```

### Persistence Across Sessions

An agent that saves its nails before shutdown resumes with full memory:

```python
# Before shutdown
mm.save_nails("session_2026_06_05.nail")

# After restart
mm = openmind.MuscleMemory.load("agent_muscles.json")
mm.load_nails("session_2026_06_05.nail")
# All previous computations are available. Zero warm-up.
```

This is how an agent becomes **continuous** — not just across invocations, but across lifetimes. The `.nail` file is the agent's diary.

### Nail Sharing

Nails can be shared between agents with the same muscle memory:

```python
# Agent A computes and shares
mm_a.flex("climate_model", region="arctic")
mm_a.save_nails("arctic_model.nail")

# Agent B loads the shared nails
mm_b.load_nails("arctic_model.nail")
result = mm_b.flex("climate_model", region="arctic")  # Instant, 0 tokens
```

This is distributed caching. Agent B doesn't need to think about arctic climate because Agent A already thought about it. The nail is the thought, preserved.

### The Memory-Precision Tradeoff

Nails store exact outputs, not approximations. But exact storage has costs:

| Strategy | Storage | Precision | Use Case |
|----------|---------|-----------|----------|
| Exact nail | Full output | 100% | Deterministic functions |
| Quantized nail | Rounded output | 99% | Floating-point results |
| Fingerprint nail | Hash only | Boolean | Cache hit detection |

The synchronizer picks the strategy based on the function's output type and the application's precision requirements.

## Connect

- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — How the synchronizer decides CACHED vs MODEL
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — Muscle memory stores capability; nails store history
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — CACHED is the 50:1 compression ratio in action
- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — Practical API for loading and saving nails
- [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md) — Nail persistence across deployments

## Activate

Cache your first computation:

```python
import openmind

mm = openmind.MuscleMemory.load("my_muscles.json")

# Compute once (costs tokens)
result = mm.flex("complex_analysis", data=my_data)

# Save the nail
mm.save_nails("my_cache.nail")

# Later: instant recall (0 tokens)
mm.load_nails("my_cache.nail")
result = mm.flex("complex_analysis", data=my_data)  # Cached!
```

Measure your cache efficiency:
```python
stats = mm.nail_stats()
print(f"Entries: {stats.count}")
print(f"Hit rate: {stats.hit_rate:.1%}")
print(f"Tokens saved: {stats.tokens_saved}")
print(f"Storage: {stats.bytes_used} bytes")
```

A well-cached agent is an agent that has already lived. The nail file is its past, available to its future.
