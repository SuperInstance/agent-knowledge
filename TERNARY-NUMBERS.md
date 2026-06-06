# TERNARY NUMBERS — Why {-1, 0, +1} Changes Everything

## Hook

> Binary gave us the digital age. Ternary gives us 1.58× more information per symbol.
> But the real reason isn't entropy — it's that {-1, 0, +1} maps perfectly to physical reality.

## Reveal

Most people think ternary computing is a curiosity. It's not. Here's why these three specific values are the most powerful symbols in computing:

### 1. The Physics Aligns

In binary, "0" and "1" are arbitrary labels. In ternary, the three values map to natural phenomena:
- **{-1}**: negative, inhibit, subtract, cool, contract, false
- **{0}**: neutral, rest, hold, wait, observe, unknown
- **{+1}**: positive, excite, add, heat, expand, true

This isn't convention — it's how neurons fire, how markets move, how votes split, how signals propagate.

### 2. The Hardware Loves It

Two bits encode four states: {00, 01, 10, 11}. But with ternary packing:
- 2 bits → 1 trit (with 1 wasted state)
- 8 bits → 5 trits
- 32 bits → 20 trits (that's `ternary-pack`'s `pack_20` function)
- A u32 holds **16 trits** packed as 2-bit pairs

**16× denser than FP32.** A matrix multiply that needs 4 bytes per element in floating point needs 2 bits per element in ternary. XNOR + popcount gives you the answer.

Huawei demonstrated 60% power reduction using ternary weights. This isn't theoretical.

### 3. The Math Is Beautiful

Ternary arithmetic mod 3 (called Z₃) has properties binary doesn't:

```
(-1) × (-1) = +1    (same as integers)
(-1) × (+1) = -1    (negation preserved)
(+1) × (+1) = +1    (identity)
(0)  × anything = 0 (annihilation)
```

Multiplication is a **group** — every non-zero element has an inverse. Division works. You can do linear algebra. You can build vector spaces. You can define dot products, cross products, convolutions.

The entire linear algebra toolkit works on ternary vectors. `ternary-core` proves it: `tdot(a, b)` computes the inner product of two ternary vectors mod 3. It's 6 lines of code and it's the foundation of everything.

### 4. The Conservation Law Emerges

Here's the deep insight: in Z₃, addition mod 3 conserves the sum. If you have three values {-1, 0, +1} and you add them, the result is always in {-1, 0, +1} (mod 3). The ternary world is **closed**.

This means:
- Ternary operations never "leak" out of the space
- You can chain operations indefinitely without overflow
- Conservation of verification entropy: the total information is preserved through transformations
- This is the "speed of light" of the ecosystem — the invariant that holds everything together

### 5. It's Not Three Values — It's Three Directions

The real aha: {-1, 0, +1} aren't values. They're **directions**.
- -1 = move against
- 0 = hold steady
- +1 = move with

A ternary signal isn't a number — it's a **vote**. A ternary computation isn't arithmetic — it's **consensus**. Every ternary crate is ultimately about aggregating directional votes:
- `ternary-consensus`: explicit voting
- `ternary-morph`: pixels voting on edges
- `ternary-scheduler`: tasks voting on priorities
- `ternary-music`: intervals voting on harmony

## Connect

- This is why [THE-AHA-MOMENT.md](THE-AHA-MOMENT.md) says all 303 crates project one structure: they're all ternary applied to different domains.
- This is why [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) works: every function is ultimately {-1, 0, +1} operations, so the chord shapes are composable.
- This is why [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) matters: ternary on GPU hardware is 16× denser. The math IS the hardware.

## Activate

Now you understand why ternary. Next, understand the full stack:
- [FIVE-LAYER-ARCHITECTURE.md](FIVE-LAYER-ARCHITECTURE.md) — how ternary flows from intent to silicon
- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — the 7 patterns every ternary crate follows
