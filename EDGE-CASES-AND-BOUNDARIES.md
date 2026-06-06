# EDGE CASES AND BOUNDARIES — Where Z₃ Gets Interesting

## Hook

> Mathematicians study edge cases to find the limits of a system. In Z₃, the edge cases aren't failures — they're the places where the closure property shines brightest.

## Reveal

Ternary arithmetic is closed: every operation on trits produces a trit. But that doesn't mean all trits are created equal. Some combinations are more common, some are degenerate, and some reveal deep structure.

### The Degenerate Cases

**Multiplying by zero:**
```
(-1) × 0 = 0
0 × 0 = 0
(+1) × 0 = 0
```

Zero is the annihilator. Any input multiplied by zero becomes zero. This isn't a bug — it's structural sparsity. In neural networks, zero weights mean "ignore this connection." The hardware ignores them for free.

**Adding opposites:**
```
(-1) + (+1) = 0 (mod 3)
(+1) + (-1) = 0 (mod 3)
```

Opposites cancel perfectly. In consensus, a {-1} vote and a {+1} vote neutralize each other. In signal processing, opposite phases cancel. The system self-stabilizes.

**Multiplying by self:**
```
(-1) × (-1) = +1
(+1) × (+1) = +1
0 × 0 = 0
```

Squaring any non-zero trit always yields +1. This is the ternary equivalent of "magnitude squared." `tdot(v, v)` for a unit vector is always +1.

### The Boundary of Three

What happens with three inputs?

```
(-1) + (-1) + (-1) = 0 (mod 3)
(-1) + (-1) + 0 = +1 (mod 3)
(-1) + 0 + 0 = -1 (mod 3)
(-1) + (-1) + (+1) = -1 (mod 3)
(-1) + 0 + (+1) = 0 (mod 3)
0 + 0 + 0 = 0 (mod 3)
(+1) + (+1) + (+1) = 0 (mod 3)
```

Interesting properties:
- Three identical non-zero trits sum to 0 (mod 3). This is why ternary consensus with 3 voters and unanimous votes produces "no decision" — the system requires a 4th voter to break symmetry.
- Two {-1}s and one {+1} sum to {-1} (the majority wins).
- One of each ({-1, 0, +1}) sums to 0 (perfect balance).

### Overflow? What Overflow?

In binary, `255 + 1 = 0` (overflow). In ternary mod 3:
```
(+1) + (+1) = -1
(-1) + (-1) = +1
```

There's no overflow — there's **wrap-around**. The result is always a valid trit. The system never produces garbage. It just keeps cycling through {-1, 0, +1}.

This is why ternary neural networks don't need gradient clipping. The weights can't explode out of range because the range IS the entire space.

### Edge Cases in Real Code

**Empty vector dot product:**
```rust
fn tdot(a: &[Trit], b: &[Trit]) -> Trit {
    if a.is_empty() {
        return Trit::Z0;  // Empty sum is neutral
    }
    // ... compute dot product
}
```

The empty dot product is 0 by convention. This matches mathematics (empty sum = identity) and ternary philosophy (no information = neutral).

**Mismatched lengths:**
```rust
fn safe_tdot(a: &[Trit], b: &[Trit]) -> Trit {
    if a.len() != b.len() {
        return Trit::Z0;  // Uncertain, not wrong
    }
    tdot(a, b)
}
```

In a binary system, mismatched lengths might panic or return an error. In ternary, they return {0} — the safe, neutral state. The system degrades gracefully (see FAULT-TOLERANCE.md).

**All-zero vector:**
```
tdot([0, 0, 0, 0], [+1, -1, +1, -1]) = 0
```

An all-zero input silences any output. This is the ternary equivalent of a disconnected neuron. It costs nothing to compute and contributes nothing to the result.

### The Most Interesting Boundary: {0} Itself

{0} isn't just "zero." It's:
- The identity for addition: `a + 0 = a`
- The annihilator for multiplication: `a × 0 = 0`
- The default for missing data: absent vote = {0}
- The safe state for errors: corruption detected = fall back to {0}
- The pause signal: {0} = hold, wait, observe

{0} is the most powerful trit because it does nothing — and doing nothing is often the right action.

## Connect

- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The mathematical foundation that makes these edge cases safe
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — Edge cases degrade to {0}, not crashes
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Property tests prove edge case behavior
- [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) — `11` encoding as the invalid boundary
- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — Hardware handles edge cases via XNOR masking

## Activate

Explore edge cases in code:

```python
import openmind

# Test the degenerate cases
print(openmind.tdot([], []))           # Trit::Z0 (empty)
print(openmind.tmul(Trit.N1, Trit.Z0)) # Trit::Z0 (annihilation)
print(openmind.tadd(Trit.N1, Trit.P1)) # Trit::Z0 (cancellation)

# Test boundary conditions
a = [Trit.P1, Trit.P1, Trit.P1]  # All +1
b = [Trit.P1, Trit.P1, Trit.P1]
print(openmind.tdot(a, b))  # +1 (3 × +1 = +3 ≡ 0 mod 3... wait)
# Actually: (+1)*(-1) in Z₃? No, dot product sums products
# (+1)*(+1) + (+1)*(+1) + (+1)*(+1) = +1 + +1 + +1 = 0 (mod 3)
print(openmind.tdot(a, b))  # Trit::Z0!
```

The edge cases aren't edge cases. They're the structure revealing itself.
