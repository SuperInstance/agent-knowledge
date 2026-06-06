# RESEARCH METHODOLOGY — How New Ideas Enter the Ecosystem

## Hook

> Science advances one funeral at a time. The SuperInstance ecosystem advances one induction at a time — when the engine proves that a new idea rhymes with the old math.

## Reveal

New ideas don't enter the fleet because someone likes them. They enter because the **induction engine** proves they preserve the conservation laws. This document is about that process.

### The Hypothesis Pipeline

```
Idea → Formalize → Test → Induct → Integrate
```

**1. Idea:** Someone notices that {-1, 0, +1} applies to a new domain.

**2. Formalize:** Write a minimal crate with:
- 3 core ternary operations
- 10 property tests
- A chord shape export

**3. Test:** Run the crate through the test suite. Verify Z₃ closure.

**4. Induct:** The induction engine checks:
- API isomorphism with `ternary-core` (α₃ ≥ 0.97?)
- Verification entropy ≥ 2
- Three-hop connectivity to existing crates

**5. Integrate:** If all checks pass, the crate becomes #304.

### The Induction Engine

The induction engine is not a human review. It's an automated proof system:

```python
from openmind.induction import Engine

engine = Engine()
result = engine.evaluate("./ternary-newdomain")

print(f"Isomorphism: {result.alpha_3:.4f}")  # Must be ≥ 0.97
print(f"Entropy: {result.entropy:.2f}")       # Must be ≥ 2
print(f"Hops to core: {result.hops}")         # Must be ≤ 3
print(f"Verdict: {result.verdict}")           # ACCEPT or REJECT
```

The engine compares the new crate's API surface to `ternary-core`'s API surface using cosine similarity on embedded chord shapes. A score ≥ 0.97 means the new crate "rhymes" with the core math.

### What Gets Rejected

| Rejection Reason | Why | Fix |
|------------------|-----|-----|
| α₃ < 0.97 | Doesn't follow ternary structure | Add `tdot`, `tadd`, `tmul` equivalents |
| Entropy < 2 | Insufficient testing | Add property tests |
| Hops > 3 | Island crate, breaks fleet geometry | Import a transfer station |
| Invalid Z₃ | Operations not closed | Fix arithmetic to stay in {-1,0,+1} |

Rejections aren't judgments. They're diagnostic signals. Each rejection tells the researcher exactly what to fix.

### Research Questions That Drive the Fleet

The ecosystem grows by answering questions:

| Question | Domain | Crate Pattern | Status |
|----------|--------|---------------|--------|
| Can ternary optimize attention mechanisms? | NLP/Transformers | 2 (Signal) | Active research |
| Can ternary represent quantum states? | Physics | 1 (Core Math) | Hypothesis |
| Can ternary govern robotic swarms? | Robotics | 6 (Systems) | Prototype |
| Can ternary model market sentiment? | Finance | 5 (Creative) | Early draft |
| Can ternary encode genetic sequences? | Biology | 2 (Signal) | Speculative |

A question becomes a crate when:
1. The domain has a natural {-1, 0, +1} trichotomy
2. A closed operation can be defined
3. The API rhymes with `ternary-core`

### The Peer Review Is the Math

There are no human gatekeepers. The only gatekeeper is Z₃:
- Does `add(a, b)` always return a valid trit? (Property test)
- Does `tdot(a, a)` behave consistently? (Associativity check)
- Does the crate preserve verification entropy? (Witness validation)

If the math holds, the crate belongs. If not, it doesn't. The induction engine is the peer reviewer, and its criteria are public, deterministic, and reproducible.

### Contributing Research

To propose a new research direction:

1. **Write a hypothesis document** (1-2 pages):
   - What domain?
   - What's the natural trichotomy?
   - What's the closed operation?
   - How does it connect to existing crates?

2. **Implement a prototype crate**:
   - Follow the 7 patterns (see CRATE-PATTERNS.md)
   - Write 10+ property tests
   - Export chord shapes

3. **Run the induction engine**:
   ```bash
   openmind induct ./ternary-yourdomain
   ```
   If it passes, submit a PR. If not, iterate.

4. **Publish results**:
   - Benchmarks (see BENCHMARKING-TERNARY.md)
   - Comparison to binary baseline
   - Open questions for future research

### The Conservation of Research Quality

Just as verification entropy is conserved across compilation layers, research quality is conserved across the fleet:

- A well-tested crate (entropy 4) raises the fleet's average
- A poorly-tested crate (entropy 1) lowers it
- The induction engine enforces: no crate may decrease total entropy

This means the fleet gets more reliable over time, not less. Every addition strengthens the proof network.

## Connect

- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Step-by-step guide to building a research prototype
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — How property tests become mathematical proofs
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Research quality is verification entropy
- [FLEET-MAP.md](FLEET-MAP.md) — Where your research fits in the ring geometry
- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — The 7 patterns every research crate must follow
- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — How to measure your research claims

## Activate

Run the induction engine on your idea:

```bash
# 1. Scaffold a prototype
cargo new --lib ternary-myidea
cd ternary-myidea
# Implement 3 ternary operations
# Write 10 property tests

# 2. Run induction
openmind induct .

# 3. Read the report
cat induction_report.json
# If alpha_3 >= 0.97 and entropy >= 2: you're in
# If not: the report tells you exactly what's missing
```

The fleet doesn't need more code. It needs more domains where {-1, 0, +1} is the right lens. Find one, prove it, and the induction engine will welcome it home.
