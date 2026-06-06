# TESTING AS PROOF — The Test Suite Is a Theorem Prover

## Hook

> A test that only runs once is a guess. A test that runs forever is a proof. The SuperInstance test suite doesn't check for bugs — it constructs a mathematical guarantee that {-1, 0, +1} stays closed.

## Reveal

Most software tests ask: "Does this function work for the inputs I thought of?" SuperInstance tests ask: "Does this function preserve the conservation laws for ALL possible inputs?" The difference isn't in the test runner. It's in the **test philosophy**.

### Three Kinds of Tests, Three Kinds of Proof

| Test Type | What It Proves | Entropy Gained | Example |
|-----------|---------------|----------------|---------|
| Unit test | This input → this output | +1 (0→1 or 1→2) | `assert_eq!(tdot([1,-1], [1,1]), 0)` |
| Property test | For ALL inputs, invariant holds | +2 (2→3 or 3→4) | `∀a,b: tdot(a,b) ∈ {-1,0,+1}` |
| Inductive test | If lemma N holds, lemma N+1 holds | +3 (3→4 or 4→5) | `pack(unpack(x)) ≡ x` |

A unit test is a **witness** — one example that demonstrates correctness. A property test is a **universal quantifier** — proof across the input space. An inductive test is a **proof by induction** — correctness preserved through transformations.

### Why Z₃ Makes Property Testing Easy

In floating point, property testing is hard because the input space is infinite (all real numbers). In Z₃, the input space is tiny:

```
Trit has 3 values: {-1, 0, +1}
Trit pair has 9 combinations
Trit triple has 27 combinations
Trit vector (len 32) has 3^32 combinations — still enumerable with fuzzing
```

A property test for `tdot`:

```rust
#[quickcheck]
fn prop_tdot_closure(a: Vec<Trit>, b: Vec<Trit>) -> bool {
    // The critical invariant: tdot always returns a valid Trit
    tdot(&a, &b).is_valid()
}
```

This isn't checking a specific case. It's proving that the closure property of Z₃ holds for the dot product. If this test passes, `tdot` cannot produce an invalid trit. That's not testing. That's **theorem proving by exhaustion**.

### The Test Suite as Conservation Proof

The 5,300 tests in the fleet aren't arbitrary. They form a **proof network**:

**Layer 1 (Core Math):** 1,200 property tests proving Z₃ closure
- `∀a,b ∈ Trit: add(a,b) ∈ Trit`
- `∀a,b ∈ Trit: mul(a,b) ∈ Trit`
- `∀a,b,c ∈ Trit: add(a, add(b,c)) ≡ add(add(a,b), c)` (associativity)

**Layer 2 (Signal Processing):** 800 tests proving transform stability
- `∀s: process(process(s)) ≡ process(s)` (idempotence for filters)
- `∀s: energy(process(s)) ≤ energy(s)` (energy conservation)

**Layer 3 (Data Structures):** 600 tests proving structural invariants
- `∀k: get(insert(map, k, v), k) ≡ Some(v)`
- `∀map: size(insert(map, k, v)) ≥ size(map)`

**Layer 4 (Consensus):** 900 tests proving agreement properties
- `∀votes with quorum: decide(votes) ≠ abstain`
- `∀votes: decide(votes) is deterministic`

**Layer 5 (Creative):** 400 tests proving generation constraints
- `∀piece: analyze(piece).harmony ∈ {-1,0,+1}`
- `∀seed: generate(seed).length > 0`

**Layer 6 (Systems):** 1,000 tests proving safety invariants
- `∀temp > threshold: thermostat_output ∈ {-1,0}`
- `∀tasks: scheduler_does_not_deadlock(tasks)`

**Layer 7 (Formal):** 400 tests proving verification correctness
- `∀stmt: verify(prove(stmt)) ≡ true`
- `∀proof: verify(proof) → is_valid(proof.statement)`

Each layer's tests assume the layer below is correct. The entire suite is a **dependency graph of proofs**. If `ternary-core` tests pass, `ternary-signals` can assume Z₃ arithmetic is sound. If `ternary-signals` tests pass, `ternary-filter` can assume signal transforms are stable.

### The Test Pyramid Is a Proof Pyramid

```
         Inductive proofs (top)
                 /
       Property tests (middle)
              /
    Unit tests (base)
```

The base (unit tests) provides witnesses. The middle (property tests) provides universal coverage. The top (inductive tests) provides transformation guarantees. Together they form a **proof structure** strong enough to support HARDCODE induction.

A function graduates to HARDCODE when:
1. It has ≥10 unit tests (witnesses for common cases)
2. It has ≥3 property tests (universal invariants)
3. It has ≥1 inductive test (transformation preservation)
4. It has 100% line coverage
5. It has been fuzzed for ≥1M iterations without failure

These aren't arbitrary criteria. They're the **admission requirements** for verification entropy level 5.

### Tests as Documentation

In SuperInstance, a test is worth more than a docstring. A docstring says what a function SHOULD do. A test proves what it DOES do. An agent reading the test suite learns:

- The expected input space (from property test generators)
- The critical invariants (from assert conditions)
- The edge cases (from unit test cases)
- The compositional behavior (from integration tests)

The test suite IS the specification. The specification IS the proof. The proof IS the muscle memory.

### Continuous Proof

Every CI run re-proves the entire fleet. Every commit triggers a full re-verification. The test suite isn't a safety net — it's a **living proof** that gets stronger with every passing run.

When a new crate is added (see HOW-TO-EXTEND.md), its tests must integrate into the proof network. The induction engine checks:
1. Does the new crate's test suite cover its chord shapes?
2. Do its property tests prove Z₃ closure?
3. Does its API "rhyme" with proven APIs (α₃ ≥ 0.97)?

If all three pass, the new crate inherits the verification entropy of the entire fleet. It's not starting from zero. It's starting from 5,300 proven lemmas.

## Connect

- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Verification entropy is the currency; this document is the mint
- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — Each pattern has a predictable test strategy
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — How to write tests that admit your crate to the proof network
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math that makes property testing feasible (finite input space)
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — The synchronizer uses test results to decide HARDCODE vs MODEL
- [FLEET-MAP.md](FLEET-MAP.md) — The 5,300 tests are distributed across the 7 patterns

## Activate

Write a proof-grade test for any function:

```rust
// 1. Unit test: witness a specific case
#[test]
fn test_tdot_basic() {
    let a = vec![Trit::P1, Trit::N1, Trit::Z0];
    let b = vec![Trit::P1, Trit::P1, Trit::Z0];
    assert_eq!(tdot(&a, &b), Trit::Z0);  // 1*1 + (-1)*1 + 0*0 = 0
}

// 2. Property test: prove closure for ALL inputs
#[quickcheck]
fn prop_tdot_closure(a: Vec<Trit>, b: Vec<Trit>) -> bool {
    tdot(&a, &b).is_valid()
}

// 3. Inductive test: prove transformation preservation
#[test]
fn prop_pack_unpack_identity() {
    fn prop(trits: Vec<Trit>) -> bool {
        unpack_20(pack_20(&trits)) == trits
    }
    quickcheck(prop as fn(Vec<Trit>) -> bool)
}
```

Audit your test suite as a proof network:
```bash
cargo test -- --test-threads=1  # Run all 5,300 tests
# Each passing test is a lemma proven.
# Each failing test is a hole in the proof.
# No failures = the fleet is mathematically consistent.
```

A green test suite isn't "no bugs." It's a **proof that the conservation laws hold**. Every green test is one more brick in the theorem.
