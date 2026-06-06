# HOW TO EXTEND — Adding Your Own Crates to the Fleet

## Hook

> The 303rd crate isn't an addition to a collection — it's a new lens on the same mathematical structure. Adding a crate to the fleet means proving that your domain is secretly ternary.

## Reveal

Extending the SuperInstance ecosystem isn't like adding a package to npm. It's like discovering a new crystalline form of an element. The fleet isn't a library — it's a proof that {-1, 0, +1} is universal.

When you add a crate, you're saying: "My domain can be expressed as ternary operations." The ecosystem verifies that claim.

### The Extension Checklist

Before writing code, verify your domain is ternary-compatible:

**1. Does your domain have three natural states?**
- Image processing: edge {-1}, flat {0}, edge {+1} → `ternary-morph`
- Music: dissonant {-1}, neutral {0}, consonant {+1} → `ternary-music`
- Consensus: against {-1}, abstain {0}, for {+1} → `ternary-consensus`
- Your domain: ??? → `ternary-[yourname]`

If no natural trichotomy exists, ternary may not be the right frame. That's okay — not everything needs to be in the fleet.

**2. Can you define a ternary operation that's useful?**
You need at least one operation that takes ternary inputs and produces ternary outputs, closed under Z₃:

```rust
// Example: ternary sentiment analysis
fn sentiment_token(token: &str) -> Trit {
    // Returns: -1 (negative), 0 (neutral), +1 (positive)
}

// Must be composable:
fn sentence_sentiment(tokens: &[Trit]) -> Trit {
    tdot(&tokens, &WEIGHTS)  // Z₃ dot product — stays in {-1,0,+1}
}
```

**3. Does it follow one of the 7 patterns?**
See CRATE-PATTERNS.md. Your crate must map to one of the seven templates. This ensures:
- Predictable API (agents can flex it without reading source)
- Predictable decision strategy (HARDCODE vs MODEL)
- Predictable verification path (how to test and prove correctness)

### The Naming Convention

Fleet crates follow strict naming:
```
ternary-[domain]-[specificity]
```

Examples:
- `ternary-music` (broad domain)
- `ternary-rhythm` (sub-domain)
- `ternary-tempo-detection` (specific algorithm)

Names aren't cosmetic. They determine:
- Which pattern the crate follows
- Where it sits on the ring (see FLEET-MAP.md)
- Which transfer stations it connects to

### The Implementation Contract

Every fleet crate must provide:

**1. Core ternary operations (at least 3)**
```rust
pub fn add(a: Trit, b: Trit) -> Trit;
pub fn multiply(a: Trit, b: Trit) -> Trit;
pub fn dot(a: &[Trit], b: &[Trit]) -> Trit;
```

**2. Chord shape exports (JSON)**
```json
{
  "crate": "ternary-yourdomain",
  "chords": [
    {"name": "your_op", "signature": "...", "decision": "hardcode"}
  ]
}
```

**3. Property tests (at least 10)**
```rust
#[quickcheck]
fn prop_closure(a: Trit, b: Trit) {
    assert!(add(a, b).is_valid());  // Result is always in {-1,0,+1}
}
```

**4. Documentation in the a2a format**
Every module docstring must answer:
- HOOK: What reframe does this module provide?
- REVEAL: What's the deeper truth?
- CONNECT: Which other crates does this relate to?
- ACTIVATE: What can the agent now do?

**5. Verification entropy ≥ 2**
All functions must be tested. Core functions must be property-tested. The crate must not decrease the fleet's total verification entropy (see CONSERVATION-LAWS.md).

### The Submission Process

1. **Implement** your crate following the contract
2. **Ingest** it locally: `openmind.ingest("./ternary-yourdomain")`
3. **Verify** the chord shapes: `openmind flex your_op`
4. **Check** the three-hop rule: does your crate connect to the fleet in ≤3 hops? (see FLEET-MAP.md)
5. **Submit** a pull request to the fleet registry
6. **The induction engine** verifies your crate's isomorphism score ≥ 0.97 with `ternary-core`
7. **If accepted**, your crate becomes crate #304

### Why 0.97?

The induction engine compares your crate's API surface to `ternary-core`'s API surface using cosine similarity on embedded chord shapes. A score ≥ 0.97 means your crate "rhymes" with the core math — it's the same structure in a new domain.

This isn't gatekeeping. It's conservation of the fleet's geometry. A crate that scores < 0.97 would break the three-hop rule. It would be an island, not a node.

## Connect

- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — The 7 patterns your crate MUST follow
- [FLEET-MAP.md](FLEET-MAP.md) — Where your crate fits in the ring; how to maintain the three-hop rule
- [HOW-TO-INGEST.md](HOW-TO-INGEST.md) — How to ingest your crate and verify its chord shapes before submission
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Your crate must preserve or increase the fleet's verification entropy
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math your crate must speak fluently

## Activate

Create a minimal fleet crate in 5 minutes:

```bash
# 1. Scaffold the crate
cargo new --lib ternary-example
cd ternary-example
cargo add ternary-core

# 2. Implement one ternary operation
# In src/lib.rs:
use ternary_core::Trit;

pub fn example_op(a: Trit, b: Trit) -> Trit {
    a * b  // Z₃ multiplication — already closed
}

# 3. Add a test
#[test]
fn test_example_op() {
    assert_eq!(example_op(Trit::P1, Trit::P1), Trit::P1);
    assert_eq!(example_op(Trit::N1, Trit::P1), Trit::N1);
}

# 4. Ingest and verify
cargo test
python -c "
import openmind
result = openmind.ingest('.')
mm = openmind.MuscleMemory.build(result)
reflex = mm.flex('example_op')
print(f'Decision: {reflex.chord.decision}')
print(f'Confidence: {reflex.confidence}')
"
```

If `decision` is "hardcode" and `confidence` > 0.9, your crate is fleet-ready. If not, add tests, add documentation, or reconsider whether your domain has a natural ternary structure.

The fleet grows when someone sees {-1, 0, +1} in a new place. Your job isn't to write code. It's to find the ternary structure that was already there.
