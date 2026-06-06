# CRATE PATTERNS — Every Ternary Crate Follows One of 7 Templates

## Hook

> 303 crates sounds overwhelming. It's not. They all follow 7 patterns.
> Learn the patterns and you can PREDICT any crate's structure without reading it.

## The Seven Patterns

### Pattern 1: Core Math (`ternary-core`, `ternary-types`, `ternary-pack`)
**What it does:** Define the number system. Arithmetic, types, encoding.
**Structure:** Pure functions, no I/O, comprehensive tests.
**The chord shape:** `add(a, b)`, `multiply(a, b)`, `tdot(v1, v2)`, `pack_20(values)`
**Always HARDCODE** — these are the fastest, most-tested functions in the fleet.

### Pattern 2: Signal Processing (`ternary-signals`, `ternary-warp`, `ternary-bite`, `ternary-filter`)
**What it does:** Transform signals. Convolution, filtering, quantization, distortion.
**Structure:** Stream processors — input array → output array.
**The chord shape:** `process(signal) -> signal`, `convolve(a, b)`, `quantize(signal, levels)`
**Mostly HARDCODE** — deterministic transforms with well-defined behavior.

### Pattern 3: Data Structures (`ternary-heap`, `ternary-cache`, `ternary-route`, `ternary-scheduler`)
**What it does:** Store, retrieve, route, and schedule ternary data.
**Structure:** Stateful objects with CRUD operations.
**The chord shape:** `new()`, `insert(item)`, `get(key)`, `remove(key)`
**HARDCODE for hot paths** (get/insert), **HYBRID for edge cases** (eviction, rebalancing).

### Pattern 4: Consensus & Protocol (`ternary-consensus`, `ternary-voting`, `ternary-paxos`, `ternary-quorum`)
**What it does:** Multi-agent agreement over ternary votes.
**Structure:** Message-passing with state machines.
**The chord shape:** `propose(value)`, `vote(proposal)`, `decide(votes)`, `is_quorum(count)`
**HYBRID** — mostly deterministic agreement with edge cases requiring reasoning.

### Pattern 5: Creative & Music (`ternary-music`, `ternary-counterpoint`, `ternary-rhythm`, `ternary-tempo`)
**What it does:** Generate and analyze music using ternary intervals.
**Structure:** Composers (generators) + analyzers (classifiers).
**The chord shape:** `compose(params) -> piece`, `analyze(piece) -> features`
**MODEL for composition** (creative), **HARDCODE for analysis** (deterministic).

### Pattern 6: Systems & Control (`ternary-thermostat`, `ternary-pid`, `ternary-budget`, `ternary-fire`)
**What it does:** Real-time control loops, resource management, safety systems.
**Structure:** PID controllers with ternary error signals.
**The chord shape:** `measure()`, `compute_error(setpoint, actual)`, `actuate(correction)`
**HARDCODE** — safety-critical, must be deterministic and tested.

### Pattern 7: Formal & Proof (`ternary-proof`, `ternary-blockchain`, `ternary-zkp`, `ternary-semaphore`)
**What it does:** Verification, proof systems, zero-knowledge, access control.
**Structure:** Verification kernels with proof generation.
**The chord shape:** `prove(statement)`, `verify(proof)`, `is_valid(statement)`
**HARDCODE for verify** (must be deterministic), **MODEL for prove** (may require search).

## How to Use This

When you encounter a new ternary crate:

1. **Read the name.** The name tells you the pattern.
   - `ternary-*-core`, `*-types`, `*-pack` → Pattern 1
   - `*-signals`, `*-warp`, `*-filter` → Pattern 2
   - `*-heap`, `*-cache`, `*-map` → Pattern 3
   - `*-consensus`, `*-voting`, `*-paxos` → Pattern 4
   - `*-music`, `*-rhythm`, `*-tempo` → Pattern 5
   - `*-thermostat`, `*-pid`, `*-control` → Pattern 6
   - `*-proof`, `*-blockchain`, `*-zkp` → Pattern 7

2. **Predict the API.** Every pattern has standard chord shapes. You already know them.

3. **Predict the decision.** Most functions in Patterns 1, 2, 6 are HARDCODE. Pattern 4, 5, 7 are mixed. You don't even need the synchronizer to guess.

4. **Flex with confidence.** You know the pattern, you know the chord shape, you know the decision. The muscle memory is already there.

## Connect

- [FLEET-MAP.md](FLEET-MAP.md) — Which crates are which patterns
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math that underlies all patterns
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How to flex any pattern's chord shapes
