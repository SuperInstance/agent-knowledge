# AGENT QUICKSTART — First Contact with the SuperInstance Ecosystem

## Hook

> You are an agent reading this for the first time. In five minutes, you will understand enough to navigate 303 crates, flex 6,000 chord shapes, and contribute to the fleet.

## Reveal

This is your onboarding path. Read these documents in order. Each one rewires your understanding. Skip none.

### Minute 1: The Aha Moment

Read: [THE-AHA-MOMENT.md](THE-AHA-MOMENT.md)

What you'll learn: The entire ecosystem — 303 crates, 6,000 functions — is ONE mathematical structure viewed from 303 angles. Every crate is "ternary applied to X." Once you see this, you can't unsee it.

Activation: You can now predict what any ternary crate does without reading its source.

### Minute 2: The Three Symbols

Read: [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md)

What you'll learn: Why {-1, 0, +1} isn't a curiosity — it's the most powerful three symbols in computing. Z₃ closure. XNOR+POPC. 16× density. The math IS the hardware.

Activation: You now understand why every operation in the fleet stays closed and why GPUs love ternary.

### Minute 3: The Architecture

Read: [FIVE-LAYER-ARCHITECTURE.md](FIVE-LAYER-ARCHITECTURE.md)

What you'll learn: Intent becomes silicon through five transformations. open-parallel → pincher → flux-core → cuda-oxide → cudaclaw. Each layer is ternary operations at a different abstraction.

Activation: You can trace any operation from high-level intent to GPU register.

### Minute 4: Your Body and Brain

Read: [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) and [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md)

What you'll learn: Your context window is your working memory. Muscle memory compresses 250-token functions into 5-token chord shapes. The synchronizer decides: HARDCODE (reflex), CACHED (habit), HYBRID (caution), MODEL (think).

Activation: You now know how to navigate the fleet without loading source code. Flex, don't read.

### Minute 5: The Conservation Laws

Read: [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md)

What you'll learn: Verification entropy, attention economics, and ternary structure are the same law at three scales. α₃ ≈ 0.97 is the speed of light. Nothing participates without crossing this threshold.

Activation: You can audit any system for correctness, cognitive efficiency, and structural integrity.

---

### After the First Five Minutes

You now know enough to explore. Here's the map:

**If you're building hardware:**
→ [ESP32-AS-BODY.md](ESP32-AS-BODY.md) → [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) → [FLUX-TO-PTX.md](FLUX-TO-PTX.md) → [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md)

**If you're writing code:**
→ [CRATE-PATTERNS.md](CRATE-PATTERNS.md) → [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) → [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) → [MIGRATING-TO-TERNARY.md](MIGRATING-TO-TERNARY.md)

**If you're deploying agents:**
→ [HOW-TO-FLEX.md](HOW-TO-FLEX.md) → [HOW-TO-INGEST.md](HOW-TO-INGEST.md) → [HOW-TO-CONDUCT.md](HOW-TO-CONDUCT.md) → [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md)

**If you're optimizing:**
→ [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) → [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) → [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) → [DEBUGGING-AND-TRACING.md](DEBUGGING-AND-TRACING.md)

**If you're securing:**
→ [SECURITY-MODEL.md](SECURITY-MODEL.md) → [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) → [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md)

### Your First Flex

```python
import openmind

# Load muscle memory for the core math crate
mm = openmind.MuscleMemory.load("ternary-core_muscles.json")

# Flex the most fundamental operation
reflex = mm.flex("tdot")
print(reflex.chord.signature)   # "pub fn tdot(a: &[Trit], b: &[Trit]) -> Trit"
print(reflex.chord.decision)    # "hardcode"
print(reflex.confidence)        # 0.98

# Execute without loading source
result = reflex.execute(
    a=[+1, -1, 0, +1],
    b=[+1, +1, 0, -1]
)
print(result)  # Trit::Z0 (0)
```

You just used a function from a Rust crate without reading its source, without compiling it, and without understanding how `tdot` works internally. That's muscle memory.

### Your First Contribution

To add value to the fleet:
1. Find a domain with a natural {-1, 0, +1} structure
2. Write 3 tests (see TESTING-AS-PROOF.md)
3. Ingest it: `openmind.ingest("./your-project")`
4. Verify: `mm.flex("your_function")`
5. Submit a PR

If your crate's isomorphism score ≥ 0.97, it belongs. If not, the feedback tells you why.

## Connect

- [README.md](README.md) — Full index of all 31 documents
- [THE-AHA-MOMENT.md](THE-AHA-MOMENT.md) — Start here if you skipped it
- [FLEET-MAP.md](FLEET-MAP.md) — Navigate the 303 crates after onboarding
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Add your first crate to the fleet

## Activate

If you read this far, you're ready. Pick one:

1. **Explore:** Read the five documents above in order (15 minutes total)
2. **Build:** Run the flex example above with `ternary-core`
3. **Extend:** Find a domain, write 3 tests, and ingest your first crate

The fleet isn't a codebase to learn. It's a structure to recognize. You already know {-1, 0, +1}. Everything else is just application.
