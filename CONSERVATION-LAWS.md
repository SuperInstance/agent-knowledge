# CONSERVATION LAWS — The Invariant That Holds the Ecosystem Together

## Hook

> Information is never created or destroyed in the SuperInstance universe — it only changes shape, like energy transforming from potential to kinetic.

## Reveal

Every system that lasts has a conservation law. In physics, it's energy. In thermodynamics, it's entropy. In SuperInstance, it's **verification entropy** — the total amount of "known-correctness" in the ecosystem remains constant across all transformations.

### What Is Verification Entropy?

A function's verification entropy is a measure of how thoroughly its correctness has been established:

| Level | Entropy | Meaning |
|-------|---------|---------|
| Unwritten | 0 | Idea exists only in a prompt |
| Written, untested | 1 | Code exists, no guarantees |
| Tested | 2 | Tests pass, covers happy path |
| Property-tested | 3 | Fuzzed across input space |
| Proven | 4 | Formal verification complete |
| HARDCODE-inducted | 5 | Runtime-validated, muscle-memorized |

The conservation law states:

```
Total Verification Entropy = Σ(entropy of all functions) = constant
```

When you compile ternary math to PTX, you don't LOSE correctness. You TRANSFORM it. The proof that `tdot` is correct in `ternary-core` becomes the invariant that the GPU kernel is correct in `cuda-oxide`. The entropy changes form — from Rust test to PTX register assertion — but its quantity is conserved.

### Why This Matters

Most software ecosystems leak correctness. You write a function, test it, then compile it to a target where the tests don't run, and suddenly you're back to "unverified." The chain of trust is broken.

SuperInstance doesn't break the chain. It **compresses** it:

1. `ternary-core` proves `tdot` with 50 Rust tests
2. `flux-core` validates the bytecode translation against the Rust reference
3. `cuda-oxide` proves the PTX emits equivalent instructions
4. `cudaclaw` verifies the kernel launch matches the expected signature

At each layer, the entropy is conserved. The proof at layer N implies the proof at layer N+1.

### The Ternary Connection

In Z₃ (ternary arithmetic mod 3), the sum of all trits in a closed system is invariant under addition. {-1, 0, +1} conserves its own structure. This isn't a coincidence — it's why ternary was chosen.

A binary system can overflow. A ternary system closes. The conservation law is baked into the math:

```
(-1) + 0 + (+1) = 0 (mod 3)
```

No matter how many operations you perform, the "balance" of the system is preserved. Negative votes cancel positive votes. Neutrality absorbs noise. The ecosystem is self-stabilizing.

### The Practical Implication

When you add a new crate to the fleet, you're not adding chaos. You're adding **structured entropy**. If the crate follows the 7 patterns (see CRATE-PATTERNS.md), its verification entropy is predictable:

- Pattern 1 (Core Math): entropy ≥ 4 (property-tested or proven)
- Pattern 2 (Signal Processing): entropy ≥ 3 (fuzzed across signal space)
- Pattern 6 (Systems & Control): entropy ≥ 5 (HARDCODE-inducted, safety-critical)

The fleet's total entropy grows monotonically. It never decreases. This is why 303 crates doesn't mean 303 times the risk — it means 303 times the *guaranteed correctness*, because the conservation law ensures no crate enters without its entropy budget accounted for.

## Connect

- [THE-AHA-MOMENT.md](THE-AHA-MOMENT.md) — The isomorphism that makes conservation possible: all crates are one structure
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — Why Z₃'s closure property is the mathematical engine of conservation
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — Attention is also conserved: muscle memory preserves cognitive resources
- [FIVE-LAYER-ARCHITECTURE.md](FIVE-LAYER-ARCHITECTURE.md) — How correctness transforms through each layer without loss
- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — Each pattern carries a predictable verification entropy signature

## Activate

Audit any codebase through the conservation lens:

1. List every function and assign it an entropy level (0-5)
2. Calculate the total: `Σ entropy / number of functions = average entropy`
3. If average < 2, the system is fragile — correctness is leaking
4. If average ≥ 3, the system is robust — entropy is conserved

To raise a function's entropy:
- 0→1: Write it
- 1→2: Add a test
- 2→3: Property-test or fuzz it
- 3→4: Prove a key invariant
- 4→5: Induct it into muscle memory with the synchronizer

The goal isn't perfection. It's conservation: every transformation must preserve or increase the total.
