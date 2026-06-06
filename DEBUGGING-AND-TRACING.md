# DEBUGGING AND TRACING — Following Intent Through Five Layers

## Hook

> When a ternary program fails, the error isn't in the silicon. It's in one of four transformations between intent and electricity. Debugging is archaeology: dig through the layers until you find where the meaning changed.

## Reveal

A bug in SuperInstance isn't "line 42 is wrong." It's "the meaning at Layer 2 diverged from the meaning at Layer 3." Debugging requires tracing intent through all five layers and finding the first layer where the semantics changed.

### The Debugging Stack

```
Layer 5 (Silicon):    "The GPU produced the wrong bits"
Layer 4 (PTX):        "The PTX doesn't match the MIR"
Layer 3 (MIR):        "The optimizer broke an invariant"
Layer 2 (Flux IR):    "The lowering dropped a constraint"
Layer 1 (Source):     "The source code has a logic error"
```

Most bugs live at the boundaries between layers. The source is correct. The PTX is correct. But the lowering from source to IR lost a sign. Or the optimizer assumed associativity that Z₃ doesn't guarantee.

### Tool 1: The Compilation Trace

Every `openmind.compile()` call can emit a full trace:

```python
result = openmind.compile("my_kernel.flux", trace=True)

for stage in result.trace:
    print(f"{stage.name}: {stage.witness}")
    if not stage.witness.is_valid():
        print(f"BREAKDOWN AT {stage.name}")
        print(stage.diff_from_previous())
```

The trace shows every transformation with a witness. If a witness is invalid, that's your bug. No guessing.

### Tool 2: Layer-by-Layer Diff

When output is wrong, compare layer N against layer N-1:

```bash
# Extract each layer's representation
cuda-oxide compile my_kernel.flux --emit ast > layer1.ast
cuda-oxide compile my_kernel.flux --emit ir > layer2.ir
cuda-oxide compile my_kernel.flux --emit mir > layer3.mir
cuda-oxide compile my_kernel.flux --emit ptx > layer4.ptx

# Diff adjacent layers
diff <(cat layer2.ir | grep tdot) <(cat layer3.mir | grep xnor)
# If they differ semantically, the optimizer is wrong
```

### Tool 3: Symbolic Verification

For critical kernels, verify symbolically:

```python
from openmind.verify import symbolic_eq

# Prove: PTX semantics = IR semantics
assert symbolic_eq(
    source="layer2.ir",
    target="layer4.ptx",
    strategy="ternary-pack"
)
```

If this fails, the compiler has a bug. File an issue against `cuda-oxide`.

### Common Bugs by Layer

**Layer 1 (Source):**
- Using integer arithmetic instead of Z₃: `(a + b) % 3` is WRONG for trits
- Correct: `tadd(a, b)` which handles {-1, 0, +1} correctly
- Symptom: `pack_20` produces invalid `11` patterns

**Layer 2 (Flux IR → MIR):**
- Loop-invariant code motion breaking ternary closure
- Example: hoisting `tdot(a, b)` outside a loop where `b` changes
- Symptom: Correct for iteration 1, wrong for iteration 2+

**Layer 3 (MIR → PTX):**
- Register allocation clobbering packed trits
- Example: reusing `%r3` for two different 16-trit vectors
- Symptom: XNOR produces garbage because one operand is wrong

**Layer 4 (PTX → SASS):**
- `ptxas` reordering instructions across memory barriers
- Example: `ld.global` followed by `xnor` becomes `xnor` followed by `ld.global`
- Symptom: Non-deterministic results, race conditions within a warp

**Layer 5 (Execution):**
- Memory alignment mismatch
- Example: loading a u32 from a non-4-byte-aligned address
- Symptom: `cudaErrorMisalignedAddress` or silent corruption

### The `debug_tdot` Diagnostic

When `tdot` produces wrong results, run this diagnostic:

```python
import openmind

def debug_tdot(a, b, expected):
    # Layer 1: Check source math
    source_result = openmind.reference_tdot(a, b)
    assert source_result == expected, f"Source bug: {source_result} != {expected}"
    
    # Layer 2: Check IR lowering
    ir_result = openmind.ir_tdot(a, b)
    assert ir_result == expected, f"Lowering bug: {ir_result} != {expected}"
    
    # Layer 3: Check MIR optimization
    mir_result = openmind.mir_tdot(a, b)
    assert mir_result == expected, f"Optimizer bug: {mir_result} != {expected}"
    
    # Layer 4: Check PTX generation
    ptx_result = openmind.ptx_tdot(a, b)
    assert ptx_result == expected, f"Codegen bug: {ptx_result} != {expected}"
    
    # Layer 5: Check GPU execution
    gpu_result = openmind.gpu_tdot(a, b)
    assert gpu_result == expected, f"Execution bug: {gpu_result} != {expected}"
    
    print("All layers match. Bug is elsewhere.")
```

The first assertion that fails tells you exactly which layer introduced the bug.

### Debugging Multi-Agent Systems

When agents disagree, trace the ternary signals:

```python
# Log every vote in the conductor
conductor = openmind.Conductor(score=mm, debug=True)
conductor.start()

# After a fault:
for agent, votes in conductor.debug_log.items():
    print(f"{agent}: {votes}")
    # Look for {-1} votes that should be {0}
    # Look for agents stuck on one value (sensor failure)
    # Look for oscillation: +1, -1, +1, -1 (control loop instability)
```

### Debugging Muscle Memory

When `mm.flex()` returns the wrong chord:

```python
reflex = mm.flex("spi_write")
print(reflex.chord.name)       # Should be "spi_write"
print(reflex.chord.module)     # Should be "drivers.spi"
print(reflex.chord.decision)   # Should be "hardcode" (if tested)
print(reflex.confidence)       # Should be > 0.9

# If confidence is low, the chord might be ambiguous
alternatives = mm.recall("spi_write", top_k=5)
for alt in alternatives:
    print(f"  {alt.name}: {alt.confidence}")
```

## Connect

- [FLUX-TO-PTX.md](FLUX-TO-PTX.md) — The six-stage pipeline; debugging is finding which stage diverges
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Tests catch layer-boundary bugs before deployment
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — When bugs happen, ternary systems degrade gracefully
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Debugging preserves verification entropy by tracing witnesses
- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — Hardware-level debugging: PTX, SASS, registers

## Activate

Enable full tracing on any compilation:

```python
import openmind

result = openmind.compile("my_kernel.flux", 
    trace=True,
    verify_each_stage=True,
    emit_intermediates=True
)

# If compilation succeeds but output is wrong:
for i, stage in enumerate(result.trace):
    stage.save(f"debug_layer_{i}.txt")

# Diff adjacent stages to find the divergence
# The bug is always at the first layer where witnesses don't match
```

When something fails:
1. Don't guess which layer. Trace all five.
2. Don't fix the symptom. Find the first divergent witness.
3. Don't patch around it. The conservation law says correctness is preserved — if it's broken, find where the chain broke.
