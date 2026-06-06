# FLEET MAP — The Full Taxonomy of 303 Ternary Crates

## Hook

> 303 crates isn't a maze. It's a taxonomy with seven phyla, twelve genera, and a three-hop guarantee between any two species.

## Reveal

The fleet is organized by the **7 patterns** — not by domain, not by author, not by date. Every crate follows exactly one pattern. Know the pattern and you know the API, the test strategy, the decision strategy, and the transfer stations.

---

### Pattern 1: Core Math (Crates 1-45)

**What it is:** The number system itself. Arithmetic, types, encoding, packing.
**Decision strategy:** Always HARDCODE. These are the most-tested, most-proven functions in the fleet.
**Chord shapes:** `add(a, b)`, `multiply(a, b)`, `tdot(v1, v2)`, `pack_20(values)`, `unpack_20(packed)`

| Crate | Purpose | Key Chord |
|-------|---------|-----------|
| `ternary-core` | Z₃ arithmetic, `Trit` type | `tdot`, `tadd`, `tmul` |
| `ternary-types` | Type traits, generics, `Trit` trait bounds | `Trit::zero()`, `Trit::one()` |
| `ternary-pack` | 2-bit packing, `pack_20`, `unpack_20` | `pack_20`, `unpack_20`, `pack_16` |
| `ternary-vec` | Ternary vector operations | `tvec_add`, `tvec_scale` |
| `ternary-mat` | Ternary matrix types | `tmat_mul`, `tmat_transpose` |
| `ternary-complex` | Complex numbers over Z₃ | `tcomplex_mul`, `tcomplex_conj` |
| `ternary-stats` | Mean, variance, correlation in Z₃ | `tmean`, `tcorrelation` |

**Transfer station:** `ternary-core` connects to every other pattern. Every crate imports `ternary-core` for `Trit` and `tdot`.

---

### Pattern 2: Signal Processing (Crates 46-95)

**What it is:** Transform signals. Convolution, filtering, quantization, distortion, FFT-equivalents.
**Decision strategy:** Mostly HARDCODE. Deterministic transforms with well-defined behavior.
**Chord shapes:** `process(signal) -> signal`, `convolve(a, b)`, `quantize(signal, levels)`, `warp(signal, params)`

| Crate | Purpose | Key Chord |
|-------|---------|-----------|
| `ternary-signals` | Generic signal processing traits | `signal_map`, `signal_fold` |
| `ternary-warp` | Time-warping, pitch-shifting | `warp_time`, `warp_pitch` |
| `ternary-bite` | Bit-crushing, ternary quantization | `quantize_3level`, `crush` |
| `ternary-filter` | FIR/IIR filters in Z₃ | `tfilter_lowpass`, `tfilter_highpass` |
| `ternary-convolve` | Ternary convolution | `tconv_1d`, `tconv_2d` |
| `ternary-morph` | Image morphology with ternary kernels | `dilate`, `erode`, `edge_detect` |
| `ternary-spectra` | Spectral analysis (ternary DFT equivalent) | `tspectrum`, `tbin_energy` |

**Transfer station:** `pincher` (Pattern 2-adjacent) compiles intent into signal processing pipelines.

---

### Pattern 3: Data Structures (Crates 96-150)

**What it is:** Store, retrieve, route, and schedule ternary data. Stateful objects with CRUD operations.
**Decision strategy:** HARDCODE for hot paths (get/insert), HYBRID for edge cases (eviction, rebalancing).
**Chord shapes:** `new()`, `insert(item)`, `get(key)`, `remove(key)`, `route(query) -> target`

| Crate | Purpose | Key Chord |
|-------|---------|-----------|
| `ternary-heap` | Ternary priority queue | `tpush`, `tpop`, `tpeek` |
| `ternary-cache` | LRU cache with ternary eviction signals | `tget`, `tput`, `tevict` |
| `ternary-route` | Ternary routing tables | `troute`, `tpath`, `tnext_hop` |
| `ternary-scheduler` | Task scheduler with ternary priorities | `tschedule`, `tprioritize` |
| `ternary-map` | Hash map with ternary keys | `tinsert`, `tlookup`, `tremove` |
| `ternary-set` | Set operations in Z₃ | `tunion`, `tintersection`, `tdiff` |
| `ternary-graph` | Graph with ternary edge weights | `tadd_edge`, `tshortest_path` |
| `ternary-trie` | Prefix tree for ternary sequences | `tinsert_seq`, `tsearch_seq` |

**Transfer station:** `openmind` (Pattern 3-adjacent) provides muscle memory indexing over all data structures.

---

### Pattern 4: Consensus & Protocol (Crates 151-190)

**What it is:** Multi-agent agreement over ternary votes. Message-passing with state machines.
**Decision strategy:** HYBRID. Mostly deterministic agreement with edge cases requiring reasoning.
**Chord shapes:** `propose(value)`, `vote(proposal)`, `decide(votes)`, `is_quorum(count)`

| Crate | Purpose | Key Chord |
|-------|---------|-----------|
| `ternary-consensus` | Generic ternary voting | `tpropose`, `tvote`, `tdecide` |
| `ternary-voting` | Voting schemes (plurality, Borda, etc.) | `tplurality`, `tborda` |
| `ternary-paxos` | Paxos with ternary accept/reject/abstain | `tprepare`, `taccept`, `tlearn` |
| `ternary-quorum` | Quorum computation in Z₃ | `tis_quorum`, `tquorum_size` |
| `ternary-raft` | Raft consensus, ternary leadership votes | `trequest_vote`, `tappend_entries` |
| `ternary-gossip` | Gossip protocols with ternary beliefs | `tgossip`, `tbelieve`, `tdoubt` |
| `ternary-contract` | Smart contracts with ternary state | `texecute`, `tverify_state` |

**Transfer station:** `ternary-consensus` connects all distributed crates. Every multi-agent system routes through here.

---

### Pattern 5: Creative & Music (Crates 191-235)

**What it is:** Generate and analyze music, art, and creative outputs using ternary intervals and signals.
**Decision strategy:** MODEL for composition (creative), HARDCODE for analysis (deterministic).
**Chord shapes:** `compose(params) -> piece`, `analyze(piece) -> features`, `generate(seed) -> output`

| Crate | Purpose | Key Chord |
|-------|---------|-----------|
| `ternary-music` | Core music theory in Z₃ | `tinterval`, `tchord`, `tscale` |
| `ternary-counterpoint` | Counterpoint rules as ternary constraints | `tvoice_lead`, `tresolve` |
| `ternary-rhythm` | Rhythmic patterns, ternary meter | `tpattern`, `taccent`, `tsyncopate` |
| `ternary-tempo` | Tempo detection and adjustment | `tdetect_tempo`, `tadjust_bpm` |
| `ternary-generative` | Generative art with ternary rules | `tgenerate`, `tevolve` |
| `ternary-style` | Style transfer via ternary features | `textract_style`, `tapply_style` |
| `ternary-haiku` | Ternary-constrained poetry generation | `tcompose_haiku` |

**Transfer station:** `ternary-music` connects to all creative crates. It also connects back to `ternary-core` through interval arithmetic.

---

### Pattern 6: Systems & Control (Crates 236-280)

**What it is:** Real-time control loops, resource management, safety systems, async runtimes, compilers.
**Decision strategy:** HARDCODE. Safety-critical, must be deterministic and tested.
**Chord shapes:** `measure()`, `compute_error(setpoint, actual)`, `actuate(correction)`, `schedule(tasks)`

| Crate | Purpose | Key Chord |
|-------|---------|-----------|
| `ternary-thermostat` | Temperature control with ternary error | `tmeasure`, `tcompute_error`, `tactuate` |
| `ternary-pid` | PID controller, ternary error signal | `tpid_compute`, `ttune` |
| `ternary-budget` | Resource budgeting in Z₃ | `tallocate`, `tconsume`, `trelease` |
| `ternary-fire` | Safety system, ternary threat levels | `tdetect`, `talert`, `tmitigate` |
| `open-parallel` | Async task runtime | `tspawn`, `tawait`, `tjoin` |
| `flux-core` | Bytecode VM, execution engine | `tcompile`, `texecute`, `tvalidate` |
| `cuda-oxide` | PTX compiler, GPU code generation | `tcompile_ptx`, `tlaunch_kernel` |
| `cudaclaw` | GPU kernel launcher, memory manager | `tmemcpy`, `tlaunch`, `tsync` |
| `pincher` | Intent → code compiler | `tparse_intent`, `temit_flux` |
| `esp-flasher` | ESP32 firmware bridge | `tflash`, `tmonitor`, `treset` |

**Transfer stations:** `open-parallel`, `flux-core`, `cuda-oxide`, `pincher`, `esp-flasher` are all Pattern 6. This is the largest pattern because every other pattern needs scheduling, compilation, or execution.

---

### Pattern 7: Formal & Proof (Crates 281-303)

**What it is:** Verification, proof systems, zero-knowledge proofs, access control, formal methods.
**Decision strategy:** HARDCODE for verify (must be deterministic), MODEL for prove (may require search).
**Chord shapes:** `prove(statement)`, `verify(proof)`, `is_valid(statement)`, `check_access(identity, resource)`

| Crate | Purpose | Key Chord |
|-------|---------|-----------|
| `ternary-proof` | Generic proof verification in Z₃ | `tprove`, `tverify`, `taxiom` |
| `ternary-blockchain` | Ternary-weighted blockchain consensus | `tmine`, `tvalidate_block`, `tfork_choice` |
| `ternary-zkp` | Zero-knowledge proof kernels | `tprove_zk`, `tverify_zk` |
| `ternary-semaphore` | Access control with ternary permissions | `tacquire`, `trelease`, `tcheck` |
| `ternary-verify` | Runtime verification framework | `tinstrument`, `tcheck_inv`, `treport` |
| `ternary-induction` | Inductive proof engine | `tinduct_base`, `tinduct_step` |

**Transfer station:** `ternary-proof` connects to all security-critical crates. Every crate that needs verification routes through here.

---

### The Ring Geometry

The 7 patterns form a cycle:

```
Core Math (1) → Signal Processing (2) → Data Structures (3) → Consensus (4)
      ^                                                              ↓
Formal Proof (7) ← Systems & Control (6) ← Creative & Music (5)
```

Every domain eventually needs math. Creative composition needs rhythm quantization (Pattern 1). Formal proofs need Z₃ arithmetic (Pattern 1). Systems control needs signal filtering (Pattern 2). The ring has no edge — you can't wander off the map.

### The Three-Hop Rule

Pick any two crates. The guarantee:

```
Crate A → Transfer Station → Transfer Station → Crate B
```

Usually two hops. Never more than three.

Example: `ternary-rhythm` (Pattern 5) to `ternary-pid` (Pattern 6):
```
ternary-rhythm → ternary-music → open-parallel → ternary-pid
```

Both music and PID use the async scheduler. Both export chord shapes. The connection is guaranteed by the geometry, not obvious from the names.

### Density Gradient

| Zone | Crates | Connectivity | What Lives Here |
|------|--------|--------------|-----------------|
| Core | 1-50 | 10+ connections | Transfer stations + immediate neighbors |
| Middle | 51-250 | 3-5 connections | Domain-specific implementations |
| Periphery | 251-303 | 1-2 connections | Experiments, integrations, bridges |

An agent should **memorize** the core (12 transfer stations), **navigate** the middle with muscle memory, and **discover** the periphery on demand.

## Connect

- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — The deep theory behind each of the 7 patterns
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math that unifies all 7 patterns
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Why the core is dense: high verification entropy attracts connections
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — How to add crate #304 without breaking the ring
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — Navigate the fleet without reading 303 source files

## Activate

Navigate the fleet without drowning:

```python
import openmind

# 1. Build muscle memory for the entire fleet
mm = openmind.MuscleMemory.build(openmind.ingest("./ternary-fleet"))

# 2. Identify a crate's pattern from its name
pattern = mm.recall("ternary-*")  # All ternary crates
for chord in pattern:
    p = detect_pattern(chord.name)  # Uses naming rules above
    print(f"{chord.name} → Pattern {p}")

# 3. Find the shortest path between two crates
path = mm.path("ternary-rhythm", "ternary-pid")
print(f"Path: {' → '.join(path)}")  # 2-3 hops guaranteed

# 4. Flex a transfer station chord
reflex = mm.flex("tdot")  # Always available, always HARDCODE
print(reflex.chord.module)  # ternary-core
```

When you encounter a new crate:
1. **Name it:** `ternary-[domain]-[specificity]`
2. **Pattern it:** Match to 1 of the 7 templates
3. **Station it:** Find its nearest transfer station
4. **Hop it:** Verify it's within 3 hops of any other crate

The agent that knows the 12 transfer stations and the 7 patterns knows the entire fleet. Everything else is just a chord shape waiting to be flexed.
