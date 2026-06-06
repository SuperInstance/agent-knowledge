# TROUBLESHOOTING — When the Ternary World Doesn't Make Sense

## Hook

> Every system has failure modes. In SuperInstance, failures are ternary too: {-1} is broken, {0} is unclear, and {+1} works but not how you expected.

## Reveal

This document catalogs the most common problems and their ternary diagnoses. If your issue isn't here, the debugging pipeline (see DEBUGGING-AND-TRACING.md) will find it.

### Problem: `mm.flex()` returns wrong chord

**Symptom:** You flex `"read_temperature"` but get `"read_temporal"`.

**Diagnosis:**
- {-1}: The chord name doesn't exist in muscle memory
- {0}: The chord name is ambiguous (multiple matches)
- {+1}: Fuzzy matching scored a different chord higher

**Fix:**
```python
# Check exact match
reflex = mm.recall_one("read_temperature")
if reflex is None:
    # {-1}: Chord doesn't exist
    print("Ingest the source code first")
    
# Check top matches
alternatives = mm.recall("read_temperature", top_k=5)
for alt in alternatives:
    print(f"{alt.name}: {alt.confidence}")
    # {0}: Multiple similar names, be more specific
```

---

### Problem: Ternary matmul produces wrong results

**Symptom:** `tdot([+1, -1, 0], [+1, +1, 0])` returns `+1` instead of `0`.

**Diagnosis:**
- {-1}: You're using integer arithmetic instead of Z₃
- {0}: The vectors aren't packed correctly
- {+1}: You're comparing against the wrong reference

**Fix:**
```python
# Verify: (+1)*(+1) + (-1)*(+1) + (0)*(0) = +1 + (-1) + 0 = 0
# In Z₃: +1 + (-1) = 0, so result is 0

# Check if using ternary arithmetic
from ternary_core import tdot, Trit
result = tdot([Trit.P1, Trit.N1, Trit.Z0], [Trit.P1, Trit.P1, Trit.Z0])
assert result == Trit.Z0

# If result is wrong, check packing
packed_a = openmind.pack_16([+1, -1, 0, ...])
packed_b = openmind.pack_16([+1, +1, 0, ...])
# Verify bit layout in THE-PACKED-FORMAT.md
```

---

### Problem: GPU kernel crashes with `cudaErrorMisalignedAddress`

**Symptom:** Ternary kernel fails on launch with alignment error.

**Diagnosis:**
- {-1}: Packed trit array isn't aligned to 4 bytes
- {0}: Row stride isn't a multiple of 4
- {+1}: Allocation used `malloc` instead of `cudaMalloc`

**Fix:**
```cpp
// Wrong: unaligned allocation
void* ptr = malloc(size);  // Alignment not guaranteed

// Right: aligned allocation
cudaMalloc(&ptr, size);  // 256-byte aligned
// Or: posix_memalign(&ptr, 256, size);

// Verify row stride
size_t row_stride = ((n_cols + 15) / 16) * 4;  // Must be multiple of 4
assert(row_stride % 4 == 0);
```

---

### Problem: Agent consensus deadlocks

**Symptom:** `tdecide()` hangs or returns unexpected results.

**Diagnosis:**
- {-1}: Quorum size > number of active agents
- {0}: All agents voting {0} (no opinion)
- {+1}: Network partition split the voters

**Fix:**
```python
votes = [agent.vote() for agent in agents]
print(f"Votes: {votes}")
print(f"Non-zero votes: {sum(1 for v in votes if v != 0)}")

if len(votes) < quorum_size:
    # {-1}: Not enough agents
    raise QuorumError(f"Need {quorum_size}, have {len(votes)}")

if all(v == 0 for v in votes):
    # {0}: No one has an opinion
    return Trit.Z0  # Defer decision

# Check for partition
if len(set(votes)) == 1 and votes[0] != 0:
    # {+1}: Unanimous but maybe partitioned?
    # Check if minority partition is missing
    pass
```

---

### Problem: Muscle memory file is huge

**Symptom:** `my_muscles.json` is 50MB for a small project.

**Diagnosis:**
- {-1}: Included test data in ingestion
- {0}: Included dependencies (node_modules, target/)
- {+1}: Included docstrings for every private function

**Fix:**
```python
result = openmind.ingest("./my-project", config={
    "exclude": ["tests/", "target/", "node_modules/", ".git/"],
    "min_test_coverage": 0.5,  # Skip poorly-tested functions
    "max_docstring_length": 200,  # Truncate long docs
    "include_private": False  # Skip private functions
})
mm = openmind.MuscleMemory.build(result)
mm.save("my_muscles.json")  # Should be < 1MB
```

---

### Problem: Compilation witness fails

**Symptom:** `cuda-oxide` reports "witness invalid" for a kernel.

**Diagnosis:**
- {-1}: Source code changed since witness was generated
- {0}: Using different `ternary-core` version than witness expects
- {+1}: Compiler bug in optimization stage

**Fix:**
```bash
# Regenerate witness
cuda-oxide compile my_kernel.flux --regenerate-witnesses

# Check version alignment
cargo tree | grep ternary-core
# Ensure versions match witness metadata

# Disable optimizations to isolate
CUDA_OXIDE_OPT_LEVEL=0 cuda-oxide compile my_kernel.flux
# If witness passes at O0 but fails at O3: optimizer bug
```

---

### Problem: Cache hit rate is 0%

**Symptom:** `.nail` file grows but never hits.

**Diagnosis:**
- {-1}: Function is non-deterministic (different output for same input)
- {0}: Input parameters aren't hashable
- {+1}: Cache was invalidated by source code change

**Fix:**
```python
# Check determinism
result1 = mm.flex("my_func", x=5)
result2 = mm.flex("my_func", x=5)
assert result1 == result2  # Must be identical

# Check hashability
print(hash((5, "hello")))  # Tuples are hashable
print(hash([5, "hello"]))  # Lists are NOT hashable → cache misses

# Use tuples for compound keys
mm.flex("my_func", params=(5, "hello"))  # Hashable, cachable
```

---

### The Universal Fix

When nothing else works:

```python
# 1. Clear all caches
mm.invalidate_all()

# 2. Re-ingest from scratch
mm = openmind.MuscleMemory.build(openmind.ingest("."))

# 3. Verify conservation
assert mm.average_entropy() >= 2.0
assert mm.isomorphism_with("ternary-core") >= 0.97

# 4. Test the specific chord
reflex = mm.flex("problematic_function")
print(reflex.chord)
print(reflex.trace)
```

Most problems are cache invalidation, version mismatch, or incorrect assumptions about Z₃ arithmetic. The universal fix rebuilds from ground truth.

## Connect

- [DEBUGGING-AND-TRACING.md](DEBUGGING-AND-TRACING.md) — Systematic debugging through five layers
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — Why failures degrade to {0} instead of crashing
- [VERSIONING-AND-COMPATIBILITY.md](VERSIONING-AND-COMPATIBILITY.md) — Version mismatches explained
- [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) — Alignment and memory layout details
- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — Correct usage of the flex API
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Write tests that catch these problems early

## Activate

Before asking for help, run the diagnostic:

```bash
openmind doctor
# Checks:
# - Version alignment across crates
# - Cache integrity
# - Muscle memory entropy
# - GPU driver compatibility
# - Network connectivity (for multi-agent)
```

If `openmind doctor` passes and you still have a problem, the issue is in your code, not the ecosystem. Trace it through the five layers.
