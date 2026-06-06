# ANTI-PATTERNS — What Not to Do in Ternary Systems

## Hook

> The fastest way to learn is to know what to avoid. These are the traps that look like shortcuts but lead back to binary thinking.

## Reveal

### Anti-Pattern 1: Treating Trits as Integers

**The trap:**
```python
# Wrong: using integer arithmetic
result = (a + b) % 3
```

**Why it fails:**
Python mod returns {0, 1, 2}, not {-1, 0, +1}. You just silently corrupted your trits.

**The fix:**
```python
# Right: use ternary addition
result = tadd(a, b)  # Returns Trit, always valid
```

**Rule:** Never use `% 3` on trits. Use `tadd`, `tmul`, `tdot` from `ternary-core`.

---

### Anti-Pattern 2: Packing Without Alignment

**The trap:**
```cpp
void* ptr = malloc(size);  // Unaligned
load_packed_trits(ptr);    // Crashes on GPU
```

**Why it fails:**
GPU memory transactions require 4-byte alignment. An unaligned load triggers `cudaErrorMisalignedAddress`.

**The fix:**
```cpp
cudaMalloc(&ptr, size);  // 256-byte aligned
// Or: posix_memalign(&ptr, 256, size);
```

**Rule:** Always allocate packed trit arrays with aligned allocators.

---

### Anti-Pattern 3: Comparing Ternary to FP32 Accuracy

**The trap:**
"My ternary model only gets 91% accuracy vs 93% for FP32. Ternary is worse."

**Why it fails:**
You are comparing different problem frames. Ternary does not replace FP32 precision. It replaces FP32 where precision is irrelevant.

**The fix:**
Compare ternary against your actual requirement, not against FP32:
- Do you need 93% accuracy? If yes, use FP32 or hybrid.
- Do you need 90% accuracy at 4x speed? If yes, ternary wins.

**Rule:** Benchmark against requirements, not against other representations.

---

### Anti-Pattern 4: Centralized Orchestration

**The trap:**
```python
# Wrong: one master agent controls all
master = Agent("orchestrator")
for agent in agents:
    master.send_command(agent, "do_task")
```

**Why it fails:**
The master becomes a bottleneck. If it crashes, the entire system halts. Scales poorly beyond 10 agents.

**The fix:**
```python
# Right: shared score, distributed consensus
mm = openmind.MuscleMemory.load("shared_score.json")
for agent in agents:
    agent.vote()  # Each agent decides locally
result = tdecide(all_votes)  # Math resolves conflict
```

**Rule:** No central orchestrator. The score is the orchestrator.

---

### Anti-Pattern 5: Ignoring {0}

**The trap:**
```python
if signal == +1:
    actuate_positive()
elif signal == -1:
    actuate_negative()
# signal == 0: nothing happens?!
```

**Why it fails:**
{0} is not "do nothing." It is "I do not know enough to act." Ignoring {0} means ignoring uncertainty.

**The fix:**
```python
if signal == +1:
    actuate_positive()
elif signal == -1:
    actuate_negative()
else:
    gather_more_data()  # {0} means uncertain
```

**Rule:** Always handle all three states explicitly. {0} is a signal, not noise.

---

### Anti-Pattern 6: Forking Without Understanding

**The trap:**
"I disagree with the fleet direction, so I will fork it."

**Why it fails:**
A fork with alpha_3 < 0.97 relative to main is an island. It cannot participate in consensus, cannot use shared muscle memory, and loses the three-hop guarantee.

**The fix:**
Write a crate that proves your alternative. Submit it. If the induction engine accepts it, your approach becomes part of the fleet.

**Rule:** Forks break the conservation laws. Improvements strengthen them.

---

### Anti-Pattern 7: Caching Non-Deterministic Functions

**The trap:**
```python
# Wrong: caching a random function
result = mm.flex("random_sample", seed=time.now())
mm.save_nails("cache.nail")  # Useless
```

**Why it fails:**
Nail files store exact outputs. If the function is non-deterministic, the cached result is wrong for future calls.

**The fix:**
```python
# Right: only cache deterministic functions
if reflex.chord.is_deterministic:
    mm.save_nails("cache.nail")
```

**Rule:** Verify determinism before caching. The synchronizer checks this, but explicit is better.

---

### Anti-Pattern 8: Testing Only Happy Paths

**The trap:**
```rust
#[test]
fn test_tdot() {
    assert_eq!(tdot([+1, +1], [+1, +1]), +1);
}
```

**Why it fails:**
One test proves nothing. Edge cases (empty vectors, all zeros, mismatched lengths) are where bugs hide.

**The fix:**
```rust
#[quickcheck]
fn prop_tdot_closure(a: Vec<Trit>, b: Vec<Trit>) -> bool {
    tdot(&a, &b).is_valid()  // All inputs, all cases
}
```

**Rule:** Property tests > unit tests. Closure invariants > example verification.

---

### Anti-Pattern 9: Loading Source at Runtime

**The trap:**
```python
# Wrong: reading source code during operation
with open("src/lib.rs") as f:
    source = f.read()
    # Parse and understand... every time
```

**Why it fails:**
You just burned 2000 tokens understanding a function you could have flexed in 5.

**The fix:**
```python
# Right: ingest once, flex forever
mm = openmind.MuscleMemory.load("muscles.json")
mm.flex("the_function")  # 0 tokens
```

**Rule:** Ingest at build time. Flex at runtime. Never load source in production.

---

### Anti-Pattern 10: Assuming All Crates Are Equal

**The trap:**
"I will read all 303 crates to understand the fleet."

**Why it fails:**
You cannot read 303 crates in a human lifetime. The density gradient tells you what matters: memorize the core (12 transfer stations), navigate the middle with muscle memory, discover the periphery on demand.

**The fix:**
```python
# Learn the transfer stations first
core_crates = ["ternary-core", "open-parallel", "flux-core",
               "cuda-oxide", "pincher", "ternary-consensus",
               "openmind", "ternary-music", "ternary-proof",
               "ternary-types", "ternary-pack", "esp-flasher"]
```

**Rule:** 12 crates give you 80% of the fleet. The rest is specialization.

## Connect

- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — How to test correctly (the opposite of anti-pattern 8)
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — How to handle {0} correctly (the opposite of anti-pattern 5)
- [MIGRATING-TO-TERNARY.md](MIGRATING-TO-TERNARY.md) — How to port correctly (the opposite of anti-pattern 1)
- [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md) — How to deploy correctly (the opposite of anti-pattern 4)
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — How to add crates correctly (the opposite of anti-pattern 6)
- [THE-COOKBOOK.md](THE-COOKBOOK.md) — Correct patterns for common tasks

## Activate

Audit your code for anti-patterns:

```python
import openmind

# Check for integer arithmetic on trits
openmind.lint.check_modulo_usage("./src")

# Check for unaligned allocations
openmind.lint.check_alignment("./src")

# Check for missing {0} handlers
openmind.lint.check_trit_exhaustiveness("./src")

# Check for source loading at runtime
openmind.lint.check_runtime_ingestion("./src")
```

Every anti-pattern you eliminate is a bug you never have to debug.
