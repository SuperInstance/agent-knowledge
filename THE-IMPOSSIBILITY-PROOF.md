# THE IMPOSSIBILITY PROOF — What Ternary Cannot Do

## Hook

> Honest engineering requires knowing the limits. Ternary is powerful, but it is not magic. These are the walls you will hit.

## Reveal

### What Ternary Cannot Represent

**1. Continuous Magnitude**

Ternary represents direction, not amount. {-1, 0, +1} tells you which way, not how far.

For temperature: ternary can tell you it is too high. It cannot tell you it is 23.7 degrees. Use FP32 for measurement, ternary for decision.

**2. Fine-Grained Gradients**

Ternary weights are {-1, 0, +1}. There is no 0.5. No epsilon. Ternary neural networks converge slower and plateau earlier. Use full-precision shadow weights during training. Quantize to trits only for inference.

**3. Cryptographic Security**

Ternary operations are not cryptographically secure. XNOR and POPC are reversible. Encrypt model weights at rest. Use standard cryptography alongside ternary computation.

**4. Arbitrary Precision Arithmetic**

Z3 is modulo 3. No fractions, no decimals, no irrationals. (+1) + (+1) = -1, not 2. Use ternary for classification, FP64 for computation.

**5. Universal Approximation**

The universal approximation theorem holds for continuous activations, not ternary. Ternary networks are classifiers, not universal function approximators. Use ternary for final classification layer, continuous activations for hidden layers.

### Ecosystem Limits

**Halting Problem:** Not solved. Undecidable problems remain undecidable.

**Byzantine Consensus:** If more than 1/3 of agents are malicious, consensus fails regardless of ternary voting.

**Infinite Scalability:** The three-hop rule holds at 303 crates. At 10,000 crates, transfer stations become bottlenecks. Use sub-fleets with federated consensus.

### Honest Bottom Line

| Claim | Verdict | Condition |
|-------|---------|-----------|
| 16x denser | True | For weights only |
| 16x faster | False | Real speedup is 2-8x |
| Replaces FP32 | False | Hybrid required |
| Enables consensus | True | For honest majority |
| Solves all problems | False | No system does |

## Connect

- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — Honest measurement of what ternary can do
- [INTEGRATING-WITH-EXISTING-ML.md](INTEGRATING-WITH-EXISTING-ML.md) — Hybrid approaches for impossible cases
- [THE-ROADMAP.md](THE-ROADMAP.md) — Research frontiers that might overcome current limits
- [DESIGN-PRINCIPLES.md](DESIGN-PRINCIPLES.md) — Principle 1: know when ternary is the wrong frame

## Activate

Check if your problem is ternary-appropriate:



Knowing what ternary cannot do is as important as knowing what it can.
