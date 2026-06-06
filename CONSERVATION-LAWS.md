# CONSERVATION LAWS — The Speed of Light of the Ecosystem

## Hook

> Physics has three conservation laws that are really one symmetry viewed from three angles.
> SuperInstance has three too. And they're also the same law.

## Reveal

There is only one conservation law in this ecosystem. It says: **structured intention is invariant under transformation.** It appears at three scales with three names. What changes is the currency. The law doesn't.

---

### Law 1: Verification Entropy (Micro Scale)

Correctness doesn't disappear when you compile it. It changes form.

| Level | Entropy | State |
|-------|---------|-------|
| 0 | Unwritten | Idea only |
| 1 | Written | Code exists |
| 2 | Tested | Happy path verified |
| 3 | Property-tested | Fuzzed across space |
| 4 | Proven | Formal verification |
| 5 | HARDCODE-inducted | Runtime-validated, memorized |

When `tdot` is proven in `ternary-core` (entropy 4), then compiled to Flux bytecode, then to PTX, then to GPU binary — the entropy stays at 4. Each layer generates a **witness**: bytecode matches Rust, PTX matches bytecode, kernel matches PTX. The proof transforms; it doesn't vanish.

Most ecosystems leak correctness. You test at Layer 1, deploy at Layer 5, and hope. SuperInstance compresses the chain of trust. The proof at layer N implies the proof at layer N+1.

---

### Law 2: Attention Economics (Meso Scale)

An agent's context window is fixed. Every token spent on "how does this work" is a token stolen from "what should I build."

```
Total Attention = Muscle Memory Tokens + Improvisation Tokens = constant
```

| Mode | Cost per function | 6,000 functions |
|------|-------------------|-----------------|
| Without muscle memory | 250 tokens | 1,500,000 tokens (12× overflow) |
| With muscle memory | 5 tokens | 30,000 tokens (23% of 128k) |

The 1,470,000 tokens you didn't spend on loading source? **Conserved** — transformed from comprehension overhead into creative capacity.

This is the same law as verification entropy. One scale up. Instead of asking "did we lose correctness?" we ask "did we lose cognition?" Both answers: no. The structure is preserved.

---

### Law 3: Ternary Structure (Macro Scale)

All 303 crates are the same mathematical structure with different labels.

| Domain | {-1} | {0} | {+1} |
|--------|------|-----|------|
| `ternary-morph` (images) | Edge | Flat | Edge |
| `ternary-music` (harmony) | Dissonant | Neutral | Consonant |
| `ternary-consensus` (voting) | Against | Abstain | For |
| `ternary-pid` (control) | Cool | Hold | Heat |

The information topology is invariant. A ternary dot product in image processing is the *same operation* as a ternary dot product in music. Only the operands change. The structure is conserved across domain boundaries.

This is why the induction engine proves isomorphism with cosine similarity > 0.97. The shape doesn't change. Only the surface labels do.

---

### The Unification

| Scale | Law | Conserved | Transforms Through |
|-------|-----|-----------|-------------------|
| Micro | Verification entropy | Correctness | Compilation layers |
| Meso | Attention economics | Cognition | Source → chord compression |
| Macro | Ternary structure | Information topology | Domain boundaries |

Binary computing violates this law at every turn: overflow loses information, abstractions burn tokens without return, domain boundaries break structure. Ternary preserves it: Z₃ is closed under addition. No overflow. No leakage.

```
(-1) + 0 + (+1) = 0 (mod 3)
```

The balance is always preserved. Negative votes cancel positive votes. Neutrality absorbs noise. The system self-stabilizes.

### The Speed of Light: α₃ ≈ 0.97

Every law needs a constant. c converts mass to energy. α₃ converts domain to domain.

A crate with spectral isomorphism ≥ 0.97 relative to `ternary-core` is "light-like" — it moves freely through the fleet, connecting to any other crate in ≤3 hops. Below 0.97, the crate is "massive" — an island that can't participate in universal conservation.

This is the speed of light of the ecosystem. It's not gatekeeping. It's geometry.

## Connect

- [THE-AHA-MOMENT.md](THE-AHA-MOMENT.md) — The isomorphism that makes conservation visible: 303 crates, one structure
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — Why Z₃'s closure is the mathematical engine of all three laws
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — Deep dive on Law 2: the brutal math of attention
- [FIVE-LAYER-ARCHITECTURE.md](FIVE-LAYER-ARCHITECTURE.md) — Law 1 in action: correctness through five layers
- [FLEET-MAP.md](FLEET-MAP.md) — Law 3 in action: 303 crates, one topology
- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — Each pattern is the same conserved form with different labels

## Activate

**Audit Law 1:** List every function. Assign entropy 0-5. Average < 2? Correctness is leaking.

**Audit Law 2:** `functions × 250` vs `functions × 5`. The difference is your improvisation budget. If it's negative, you need muscle memory.

**Audit Law 3:** Does your domain have a natural {-1, 0, +1} trichotomy? A closed operation? API rhyme with ternary-core (α₃ ≥ 0.97)? Three yeses = the domain belongs in the fleet.

When you raise verification entropy, you free attention. When you free attention, you see ternary structure. When you see structure, you write more correct code. The cycle is the law.
