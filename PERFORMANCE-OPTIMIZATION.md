# PERFORMANCE OPTIMIZATION — Making Ternary Code Fly

## Hook

> A ternary matmul is already 16× denser than FP32. But density is just the start. Kernel fusion, memory coalescing, and warp scheduling can squeeze another 2-4× out of the same silicon.

## Reveal

Ternary's 16× density advantage is free — it comes from the representation. But the remaining performance comes from optimization: how you lay out memory, how you schedule warps, how you fuse operations. This document is about earning that extra multiplier.

### Optimization 1: Memory Coalescing

GPU memory is fastest when threads in a warp access consecutive addresses.

**Bad layout (strided access):**
```
Thread 0 reads addr 0, Thread 1 reads addr 64, Thread 2 reads addr 128...
```
Each thread triggers a separate memory transaction. 32 transactions per warp.

**Good layout (coalesced access):**
```
Thread 0 reads addr 0, Thread 1 reads addr 4, Thread 2 reads addr 8...
```
One memory transaction loads 128 consecutive bytes. All 32 threads served.

For ternary matrices, store in row-major order with packed rows:
```
Row 0: [trit_0, trit_1, ..., trit_15] → u32_0
       [trit_16, ..., trit_31] → u32_1
Row 1: [trit_32, ..., trit_47] → u32_2
```

Each thread loads one u32 (16 trits). Adjacent threads load adjacent u32s. Perfect coalescing.

### Optimization 2: Kernel Fusion

Don't launch separate kernels for pack → xnor → popc → unpack. Fuse them into one kernel.

**Before (4 kernel launches):**
```
kernel_1: pack inputs
kernel_2: xnor
kernel_3: popcount
kernel_4: adjust and store
```
Overhead: 4 × kernel_launch + 3 × cudaMemcpy

**After (1 fused kernel):**
```
fused_kernel:
  load → xnor → popc → adjust → store
  (all in registers, no intermediate memory writes)
```
Overhead: 1 × kernel_launch. All intermediate results stay in registers.

The `cuda-oxide` compiler (see FLUX-TO-PTX.md) performs this fusion automatically when it detects consecutive ternary operations.

### Optimization 3: Warp Divergence Elimination

GPU warps execute in lockstep. If threads take different branches, the warp serializes.

**Bad (divergent branches):**
```ptx
if (thread_id < 16) {
    // 16 threads execute this
} else {
    // 16 threads execute this
}
// Warp executes both paths, masking inactive threads
```

**Good (branchless ternary):**
```ptx
// XNOR is branchless
xnor.b32 %r0, %ra, %rb;
// POPC is branchless
popc.b32 %r1, %r0;
```

Ternary operations are inherently branchless. XNOR and POPC don't have conditions. Every thread does the same operation on different data. Zero warp divergence.

### Optimization 4: Occupancy Tuning

Occupancy = (active warps per SM) / (max warps per SM). Higher occupancy hides memory latency.

| Factor | FP32 Kernel | Ternary Kernel |
|--------|-------------|----------------|
| Registers per thread | 32 | 8 |
| Shared memory per block | 4 KB | 1 KB |
| Max warps per SM | 32 | 64 |
| Occupancy | 50% | 100% |

Ternary kernels use fewer registers (no FP32 temporaries) and less shared memory (packed data is smaller). This means more warps fit on each SM, higher occupancy, better latency hiding.

### Optimization 5: Streaming Multiprocessor Utilization

An A100 has 108 SMs. To saturate all of them:

```python
# Launch enough blocks to cover all SMs
num_sms = 108
blocks_per_sm = 2  # Aim for 2 blocks per SM for good overlap
grid_size = num_sms * blocks_per_sm  # 216 blocks

# Each block should have 128-256 threads
block_size = 128

launch_kernel(grid=grid_size, block=block_size)
```

For large matrices (4096×4096), launch 216 blocks of 128 threads = 27,648 threads. Each thread handles a tile of the output matrix. All 108 SMs stay busy.

### The Roofline Model

Where is your kernel bottlenecked?

```
Compute-bound: You're doing enough ops per byte
Memory-bound: You're loading data faster than you can compute
```

Ternary matmul is usually compute-bound on modern GPUs because:
- Memory bandwidth: 2 TB/s (A100)
- Compute: 312 TFLOPS (FP32), effectively 5,000+ TOPS (ternary XNOR+POPC)
- Arithmetic intensity: 1,000+ ops/byte for ternary

If you're memory-bound, check:
- Are you coalescing loads?
- Are you reusing data in shared memory?
- Are you fusing kernels to avoid intermediate stores?

### Benchmarking Your Optimizations

Always measure before and after:

```python
import openmind

# Baseline
baseline = openmind.benchmark("ternary_matmul", size=4096, optimize=False)

# Optimized
optimized = openmind.benchmark("ternary_matmul", size=4096, optimize=True)

speedup = baseline["time_ms"] / optimized["time_ms"]
print(f"Speedup: {speedup:.2f}x")
print(f"Memory: {baseline['memory_gb']:.2f} → {optimized['memory_gb']:.2f} GB")
```

Expected speedups from each optimization:
- Coalescing: 1.5-2×
- Fusion: 2-4×
- Divergence elimination: 1.2-1.5×
- Occupancy tuning: 1.1-1.3×
- Combined: 5-10× on top of the base 16× density advantage

## Connect

- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — The hardware foundation that makes these optimizations possible
- [FLUX-TO-PTX.md](FLUX-TO-PTX.md) — How cuda-oxide generates optimized PTX
- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — Methodology for measuring speedups honestly
- [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) — Memory layout for coalesced access
- [FIVE-LAYER-ARCHITECTURE.md](FIVE-LAYER-ARCHITECTURE.md) — Where optimization fits in the stack (Layer 3-4)

## Activate

Profile your kernel:

```bash
# Run with NVIDIA Nsight Compute
ncu --kernel-name similarity --metrics \
  sm__warps_active.avg.pct_of_peak_sustained_elapsed,\
  dram__bytes_read.sum,\
  smsp__cycles_elapsed.avg \
  ./my_kernel

# Look for:
# - Low warp occupancy → increase blocks, reduce registers
# - High memory traffic → enable kernel fusion
# - Branch divergence → replace branches with XNOR/POPC
```

Optimize one thing at a time:
1. Fix memory coalescing first (biggest impact)
2. Fuse kernels second
3. Tune occupancy last

Measure after every change. Optimization without measurement is guesswork.
