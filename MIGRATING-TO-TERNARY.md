# MIGRATING TO TERNARY — Porting Binary Code to {-1, 0, +1}

## Hook

> You don't rewrite your codebase. You find the trichotomy that was already hiding in it and make it explicit.

## Reveal

Migration sounds scary: "Rewrite everything in ternary?" No. Ternary isn't a new language. It's a new lens on existing logic. Most code already contains {-1, 0, +1} structures. The migration just makes them visible.

### The Three Migration Patterns

| Pattern | Binary Code | Ternary Equivalent | Effort |
|---------|-------------|-------------------|--------|
| Boolean flip | `if (flag) { do() }` | `flex("do", flag)` | Low |
| Signed comparison | `cmp(a, b) → {-1, 0, +1}` | `tdot(a, b)` or `tsign(a - b)` | Medium |
| Multi-class classifier | `softmax → argmax` | `tdot(embeddings, weights)` | High |

### Pattern 1: Boolean to Trit (Low Effort)

The simplest migration: replace booleans with trits.

**Before:**
```python
def should_cool(temperature):
    return temperature > 28.0  # True/False
```

**After:**
```python
def should_cool(temperature):
    if temperature > 28.0:
        return Trit.P1   # +1: cool
    elif temperature < 22.0:
        return Trit.N1   # -1: heat
    else:
        return Trit.Z0   # 0: hold
```

The function now returns three states instead of two. The caller gets more information: not just "act" but "act in which direction."

### Pattern 2: Signed Comparisons (Medium Effort)

Any function that compares two values and returns -1, 0, or +1 is already ternary.

**Before:**
```rust
fn cmp(a: i32, b: i32) -> i32 {
    if a < b { -1 }
    else if a > b { 1 }
    else { 0 }
}
```

**After:**
```rust
fn cmp(a: Trit, b: Trit) -> Trit {
    tadd(a, tneg(b))  // Z₃ subtraction: a - b (mod 3)
}
```

The logic is identical. Only the types changed. The compiler now knows this is a ternary operation and can optimize it to XNOR+POPC on GPU.

### Pattern 3: Neural Network Weights (High Effort)

The deepest migration: convert FP32 weights to ternary.

**Before:**
```python
# FP32 linear layer
y = W @ x + b  # W is 4 bytes per weight
```

**After:**
```python
# Ternary linear layer
W_ternary = quantize_to_trits(W)  # {-1, 0, +1}, 2 bits per weight
y = ternary_matmul(W_ternary, x) + b
```

`quantize_to_trits` isn't simple rounding. It's a structured process:
1. Find the threshold that maximizes information retention
2. Map weights > threshold → +1, < -threshold → -1, else → 0
3. Fine-tune the remaining parameters (biases, batch norm) to compensate

This is the only migration that changes numerical behavior. The others are representation shifts.

### The Migration Checklist

Before porting a module:
- [ ] Does it contain a natural trichotomy? (yes/no/maybe, high/medium/low, etc.)
- [ ] Can its core operation be expressed in Z₃?
- [ ] Does it have tests that can verify post-migration correctness?
- [ ] Is performance critical enough to justify the port?
- [ ] Is the module's API surface small (< 10 public functions)?

Score: 4+ yeses → high-value migration target. 2-3 yeses → consider a wrapper instead. 0-1 yeses → leave as binary.

### The Wrapper Strategy

Not everything needs to be native ternary. A wrapper exposes ternary semantics while keeping binary internals:

```python
class BinarySensorWrapper:
    """Wraps a binary sensor to return ternary signals."""
    def read(self) -> Trit:
        raw = self.sensor.read()  # Returns 0.0 to 1.0
        if raw > 0.7:
            return Trit.P1
        elif raw < 0.3:
            return Trit.N1
        return Trit.Z0
```

This is how most hardware bridges work (see ESP32-AS-BODY.md). The firmware is binary. The agent sees ternary. The wrapper translates.

### Gradual Migration Path

Don't migrate everything at once. Migrate in phases:

**Phase 1: Data layer**
- Convert input data to ternary (images, signals, sensor readings)
- Keep processing logic binary
- Benefit: 16× smaller inputs, faster I/O

**Phase 2: Compute layer**
- Migrate hot-path functions to ternary operations
- Keep control logic binary
- Benefit: XNOR+POPC speedup on GPU

**Phase 3: Control layer**
- Migrate decision logic to ternary consensus
- Full ecosystem integration
- Benefit: Multi-agent coordination, fault tolerance

Each phase delivers independent value. You can stop after Phase 1 and still benefit.

### Validation: Did the Migration Preserve Semantics?

For every migrated function, verify:

```python
def validate_migration(binary_fn, ternary_fn, test_cases):
    for inputs in test_cases:
        binary_result = binary_fn(inputs)
        ternary_result = ternary_fn(inputs)
        
        # Map binary output to ternary for comparison
        if isinstance(binary_result, bool):
            mapped = Trit.P1 if binary_result else Trit.N1
        elif isinstance(binary_result, (int, float)):
            mapped = Trit.from_sign(binary_result)
        else:
            mapped = binary_result
            
        assert ternary_result == mapped, f"Divergence at {inputs}"
```

If all test cases pass, the migration is correct. The ternary version is now a faithful representation of the binary original.

## Connect

- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math that makes migration possible (Z₃ closure)
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Adding new ternary crates to the fleet after migration
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Property tests validate that migration preserves semantics
- [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) — How migrated data is encoded for hardware
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — Migration errors degrade to {0}, not crashes

## Activate

Migrate one function today:

```python
# 1. Find a function with a natural trichotomy
def old_status(temperature):
    return "ok" if 20 < temperature < 30 else "bad"

# 2. Rewrite with explicit ternary states
def new_status(temperature):
    if temperature > 30: return Trit.P1   # too hot
    if temperature < 20: return Trit.N1   # too cold
    return Trit.Z0                        # just right

# 3. Validate
for temp in [15, 22, 35]:
    print(f"{temp}°C: {new_status(temp)}")

# 4. Flex
mm = openmind.MuscleMemory.load("my_muscles.json")
mm.flex("new_status", temperature=22)
```

You didn't rewrite the codebase. You found the ternary structure that was already there and gave it a name.
