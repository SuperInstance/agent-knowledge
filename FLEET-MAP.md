# FLEET MAP — Navigating 303 Crates Without Drowning

## Hook

> 303 crates isn't a maze — it's a subway map. Every line connects to every other line at exactly three transfers or fewer.

## Reveal

The ternary fleet looks like chaos: 303 separate repositories, 6,000 functions, 5,300 tests. But the fleet has a hidden geometry. If you know the transfer stations, you can get from any crate to any other crate in three hops or less.

### The Transfer Stations

These 12 crates are the "Grand Central Terminals" of the fleet. Every other crate connects through at least one of them:

| Transfer Station | Pattern | What It Does | Connects To |
|-----------------|---------|--------------|-------------|
| `ternary-core` | 1 (Core Math) | Z₃ arithmetic, `tdot`, `pack_20` | All math, all signal, all GPU |
| `ternary-types` | 1 (Core Math) | Type system, traits, generics | Every crate that defines a type |
| `ternary-pack` | 1 (Core Math) | Bit-packing, 2-bit encoding | All hardware-facing crates |
| `open-parallel` | 6 (Systems) | Async runtime, scheduler | All concurrent crates |
| `flux-core` | 6 (Systems) | Bytecode VM, execution engine | All compiler and runtime crates |
| `cuda-oxide` | 6 (Systems) | PTX compiler, GPU IR | All hardware acceleration |
| `pincher` | 2 (Signal) | Intent → code compiler | All user-facing, all creative |
| `ternary-consensus` | 4 (Protocol) | Voting, agreement, quorum | All distributed, all multi-agent |
| `openmind` | 3 (Data) | Muscle memory, ingestion, flex | All agent-facing APIs |
| `ternary-music` | 5 (Creative) | Composition, intervals, rhythm | All creative and generative |
| `ternary-proof` | 7 (Formal) | Verification, ZKP kernels | All security-critical |
| `esp-flasher` | 6 (Systems) | Firmware bridge to ESP32 | All hardware-body crates |

### The Three-Hop Rule

Pick any two crates at random. Here's the guarantee:

```
Crate A → Transfer Station 1 → Transfer Station 2 → Crate B
```

Usually it's two hops. Never more than three.

Example: `ternary-rhythm` (Pattern 5, music) to `ternary-thermostat` (Pattern 6, control):
```
ternary-rhythm → ternary-music → open-parallel → ternary-thermostat
```

Both music and thermostat use the async scheduler. Both export chord shapes that the agent flexes. The connection isn't obvious from the names, but it's guaranteed by the geometry.

### The Ring Structure

The fleet isn't a tree — it's a ring. The 7 patterns (see CRATE-PATTERNS.md) form a cycle:

```
Core Math → Signal Processing → Data Structures → Consensus → Creative → Systems → Formal → Core Math
```

Why a ring? Because every domain eventually needs math. Creative composition (Pattern 5) needs rhythm quantization (Pattern 1). Formal proofs (Pattern 7) need arithmetic in Z₃ (Pattern 1). Systems control (Pattern 6) needs signal filtering (Pattern 2).

The ring means there's no "edge" of the fleet. You can't wander off the map.

### The Density Gradient

Crates aren't evenly distributed. The fleet has a dense core and a sparse periphery:

**Core (crates 1-50):** High connectivity. Every core crate connects to 10+ others. These are the transfer stations and their immediate neighbors.

**Middle (crates 51-200):** Moderate connectivity. Each middle crate connects to 3-5 others. These are domain-specific implementations.

**Periphery (crates 201-303):** Low connectivity. Each peripheral crate connects to 1-2 others. These are experiments, integrations, and domain bridges.

An agent should memorize the core, navigate the middle with muscle memory, and discover the periphery on demand.

## Connect

- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — The 7 patterns that explain WHY the ring connects the way it does
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math that every transfer station speaks
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Why the core is dense: high verification entropy attracts connections
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — How to add your crate to the map without breaking the three-hop rule
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How to navigate without reading the source of 303 crates

## Activate

When you encounter a new ternary crate:

1. **Identify its pattern** using the naming rules in CRATE-PATTERNS.md
2. **Find its nearest transfer station** — which of the 12 core crates does it import?
3. **Map its chord shapes** — what does the agent flex to use it?
4. **Check its density** — is it core, middle, or periphery? (How many `Cargo.toml` dependencies?)

To navigate from crate A to crate B:
```
# In your head, not in code:
1. What pattern is A? What pattern is B?
2. What transfer station connects those two patterns on the ring?
3. Does A import that station directly? Does B?
4. If yes to both: 2 hops. If yes to one: 3 hops. If no to both: find a middle crate.
```

The agent that knows the 12 transfer stations knows the entire fleet.
