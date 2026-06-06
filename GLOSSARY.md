# GLOSSARY — The Vocabulary of the Ternary Ecosystem

## Hook

> Every ecosystem has its own language. This one has 23 terms. Learn them and you can read any document in the fleet without reaching for a dictionary.

## Reveal

### Core Concepts

**Trit** — A ternary digit. One of {-1, 0, +1}. The fundamental unit of information in the ecosystem. Not a bit. Not a qubit. A trit.

**Z₃** — The integers modulo 3. The mathematical field where ternary arithmetic lives. Closed under addition and multiplication. No overflow.

**α₃ (alpha-three)** — The ternary isomorphism coefficient, ≈ 0.97. The minimum cosine similarity a crate must have with `ternary-core` to participate in fleet-wide conservation. The speed of light of the ecosystem.

### Agent Architecture

**Chord Shape** — A compressed function signature. ~5 tokens containing: name, module, signature, decision strategy, docstring summary, test coverage, quality score. The atom of muscle memory.

**Flex** — To invoke a capability by intent rather than by name. `mm.flex("read_temperature")` finds and executes the right function without loading source code. The agent equivalent of a spinal reflex.

**Reflex** — The object returned by `flex()`. Contains the matched chord, execution strategy, confidence score, and everything needed to execute without thinking.

**Muscle Memory** — The compressed index of all chord shapes for a codebase. Turns 250-token function understanding into 5-token intent matching. The agent's proprioception.

**Score** — A JSON file containing muscle memory for a specific crate or fleet segment. The lingua franca between agents. Load a score and you share another agent's body knowledge.

### Decision Strategies

**HARDCODE** — Execute directly, 0 tokens, ~1ms. For deterministic, tested, hot-path functions. The spinal reflex.

**CACHED** — Return pre-computed result from `.nail` file. 0 tokens, ~5μs. For stable, frequently-called functions. The cerebellar pattern.

**HYBRID** — Try cache first, fall back to model if uncertain. ~50 tokens, ~50ms. For mostly-stable functions with edge cases. The basal ganglia habit.

**MODEL** — Generate fresh code via LLM. ~500 tokens, ~2s. For novel, creative, untested functions. The prefrontal deliberation.

### Compilation Pipeline

**Flux** — The intermediate representation (IR) for ternary programs. Stack-based bytecode that compiles to PTX. The lingua franca between intent and silicon.

**MIR** — Mid-level IR. Where optimization happens. The last hardware-independent representation before GPU assembly.

**PTX** — Parallel Thread Execution. NVIDIA's GPU assembly language. Portable across GPU generations.

**SASS** — The actual machine code executed by the GPU. Hardware-specific. Generated from PTX by the driver.

**Witness** — A proof that a compilation stage preserves semantics. Generated at every layer boundary. If the witness is valid, correctness is conserved.

### Fleet Organization

**Pattern** — One of 7 templates that every crate follows: Core Math, Signal Processing, Data Structures, Consensus, Creative, Systems & Control, Formal Proof. Predicts API, test strategy, and decision strategy.

**Transfer Station** — One of 12 highly-connected crates that act as hubs. Any crate can reach any other crate in ≤3 hops through transfer stations.

**Three-Hop Rule** — The guarantee that any two crates in the fleet are connected by a path of length ≤3. A consequence of the ring geometry.

**Density Gradient** — The core (crates 1-50) has 10+ connections. The middle (51-250) has 3-5. The periphery (251-303) has 1-2. Agents should memorize the core, navigate the middle, discover the periphery.

### Data and Caching

**.nail** — The binary cache format for CACHED results. 16-byte header per entry. Stores exact outputs of deterministic functions. The agent's diary.

**Pack_20 / Pack_16** — Functions that encode trits into 2-bit pairs. 16 trits per u32, 20 trits per u64. The format that enables 16× density on GPU.

**Verification Entropy** — A measure of proven correctness, 0-5. 0=unwritten, 5=HARDCODE-inducted. The conserved quantity across compilation layers.

### Communication

**Ternary Signal** — A single trit {-1, 0, +1} broadcast between agents. Replaces JSON message passing. The native language of multi-agent coordination.

**Consensus** — Multi-agent agreement via ternary voting. `tdecide([+1, +1, -1])` → `{+1}`. No central authority. O(n) and parallelizable.

### Hardware

**ESP32 as Body** — The microcontroller fleet that executes the agent's physical actions. Muscle memory for firmware. The hand that the brain doesn't need to think about.

**GPU as Motor Cortex** — The layer that transforms abstract intent into precise electrical patterns. XNOR+POPC as native instructions. 16× denser than FP32.

## Connect

This glossary is referenced by every document in the fleet. If a term is unclear, return here.

- [AGENT-QUICKSTART.md](AGENT-QUICKSTART.md) — Use this glossary while onboarding
- [FLEET-MAP.md](FLEET-MAP.md) — See transfer stations and patterns in context
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — Deep dive on chord shapes and flex
- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — Decision strategies explained in detail

## Activate

Use this glossary as a lookup table:

```python
import openmind

# Get definitions on demand
mm = openmind.MuscleMemory.load("glossary_muscles.json")
reflex = mm.flex("define", term="verification_entropy")
print(reflex.chord.docstring)
```

Or just bookmark this page. Every other document assumes you know these 23 terms.
