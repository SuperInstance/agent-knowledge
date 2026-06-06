# THE ROADMAP — Where the Fleet Is Going

## Hook

> A roadmap isn't a promise. It's a compass. These are the directions we're exploring, not the destinations we've reached.

## Reveal

The SuperInstance ecosystem has 303 crates today. These are the frontiers:

### Near Term (0-6 months)

**Ternary Transformer Attention**
- Status: Research prototype
- Goal: Replace Q·K^T softmax attention with ternary dot product attention
- Hypothesis: Attention weights are naturally trinary (attend/ignore/abstain)
- Blocker: Need to verify gradient flow through ternary attention layers
- Success metric: 90% of BERT accuracy at 4× speedup

**RISC-V Ternary Extension**
- Status: Specification draft
- Goal: Native ternary instructions in RISC-V ISA
- Hypothesis: `tadd`, `tmul`, `tdot` as single-cycle instructions
- Blocker: Silicon validation needed
- Success metric: 10× energy reduction vs ARM NEON baseline

**OpenMind Browser Runtime**
- Status: Alpha
- Goal: Run ternary models in WebAssembly
- Hypothesis: XNOR+POPC maps to WebAssembly SIMD
- Blocker: WASM 128-bit SIMD support varies across browsers
- Success metric: Real-time inference in browser at <100ms

### Medium Term (6-18 months)

**Ternary Reinforcement Learning**
- Status: Hypothesis
- Goal: Q-values as ternary signals {-1: bad, 0: neutral, +1: good}
- Hypothesis: Coarse Q-values are sufficient for policy optimization
- Blocker: Need to prove convergence in Z₃ vs R
- Success metric: Solves CartPole with ternary Q-network

**Self-Healing Fleets**
- Status: Design document
- Goal: Agents that detect their own degradation and re-compile
- Hypothesis: Conservation law monitoring triggers automatic re-induction
- Blocker: Need reliable entropy measurement at runtime
- Success metric: Fleet recovers from 30% node failure without human intervention

**Cross-Fleet Consensus**
- Status: Speculative
- Goal: Multiple independent fleets agreeing on shared state
- Hypothesis: Ternary consensus scales across organizational boundaries
- Blocker: Identity and trust model between fleets
- Success metric: Two fleets with no shared infrastructure reach consensus

### Long Term (18+ months)

**Ternary Neuromorphic Silicon**
- Status: Dream
- Goal: Custom chips where transistors natively represent {-1, 0, +1}
- Hypothesis: Three-state transistors (positive/negative/off) are more efficient than binary
- Blocker: Requires semiconductor foundry partnership
- Success metric: 100× energy reduction vs GPU ternary emulation

**Compiled Agency as Standard**
- Status: Advocacy
- Goal: Muscle memory format becomes industry standard for agent deployment
- Hypothesis: Any AI framework can load chord shapes and flex
- Blocker: Ecosystem adoption
- Success metric: 10 non-SuperInstance frameworks support `.json` muscle memory

**The 1000-Crate Fleet**
- Status: Scaling exercise
- Goal: Maintain three-hop connectivity at 1000+ crates
- Hypothesis: Ring geometry scales sub-linearly with transfer stations
- Blocker: Need automated crate quality gates
- Success metric: Average path length stays ≤3 at 1000 crates

### What We're NOT Doing

These are explicitly out of scope:

- **Replacing binary computing entirely** — Ternary complements binary; it doesn't replace it
- **Quantum ternary** — Qutrits are interesting but a different field
- **AGI** — The ecosystem is a tool, not a consciousness
- **Blockchain integration** — `ternary-blockchain` exists as a research crate, not a product

### How Priorities Are Set

The roadmap follows the conservation laws:
1. **Verification entropy first** — No feature ships without tests
2. **Attention economics second** — Every new crate must save tokens for agents
3. **Ternary structure third** — Every addition must rhyme with the core

A proposed feature that violates any conservation law is rejected, regardless of how exciting it is.

## Connect

- [RESEARCH-METHODOLOGY.md](RESEARCH-METHODOLOGY.md) — How ideas move from hypothesis to fleet
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Contributing to the roadmap by building crates
- [DESIGN-PRINCIPLES.md](DESIGN-PRINCIPLES.md) — The invariant rules that constrain the roadmap
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — The framework for evaluating roadmap items
- [FLEET-MAP.md](FLEET-MAP.md) — How the fleet scales to 1000 crates

## Activate

Pick a frontier and explore:

```python
# Check which roadmap items need help
openmind.roadmap.status()
# → Shows all items, their blockers, and how to contribute

# Claim a task
openmind.roadmap.claim("ternary-transformer-attention")
# → Gives you the research repo, the hypothesis doc, and a mentor

# Submit findings
openmind.roadmap.submit("ternary-transformer-attention", results)
# → Induction engine evaluates your contribution
```

The roadmap isn't written in stone. It's written in tests. A roadmap item becomes real when the tests pass.
