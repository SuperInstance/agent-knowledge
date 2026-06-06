# BENCHMARKING TERNARY — How to Measure the 16× Claim

## Hook

> "16× denser" is meaningless without a reproducible benchmark. The only valid comparison is: same algorithm, same inputs, same hardware — different representation.

## Reveal

Ternary's performance advantages are real but easy to mismeasure. Comparing a hand-optimized ternary kernel against an unoptimized FP32 reference is dishonest. Comparing different algorithms is irrelevant. The valid comparison is **representation swap**: identical logic, only the number format changes.

### The Golden Benchmark

```python
import openmind
import time
import numpy as np

def benchmark_matmul(M, K, N, dtype="fp32"):
    if dtype == "fp32":
        A = np.random.randn(M, K).astype(np.float32)
        B = np.random.randn(K, N).astype(np.float32)
        start = time.time()
        C = A @ B
        elapsed = time.time() - start
        memory = (M * K + K * N + M * N) * 4
    else:  # ternary
        A = openmind.TernaryMatrix.random(M, K)
        B = openmind.TernaryMatrix.random(K, N)
        start = time.time()
        C = openmind.ternary_matmul(A, B)
        elapsed = time.time() - start
        memory = (M * K + K * N) // 4 + M * N * 4  # Packed inputs, i32 output
    
    return {
        "time_ms": elapsed * 1000,
        "memory_bytes": memory,
        "gops": (2 * M * K * N) / (elapsed * 1e9)
    }
```

Key constraint: **same dimensions, same hardware, same operation count**. The only variable is the weight representation.

### What to Measure

| Metric | FP32 Baseline | Ternary Target | How to Measure |
|--------|--------------|----------------|----------------|
| Memory | 4 bytes/weight | 0.25 bytes/weight | `sys.getsizeof()` or `nvidia-smi` |
| Bandwidth | 4× matrix size | 0.25× matrix size | `cudaMemcpy` bytes / time |
| Operations | 2×M×K×N FMAs | M×K×N/16 XNOR+POPC | PTX instruction count |
| Latency | `cudaEventElapsedTime` | Same | Same |
| Energy | `nvidia-smi -q -d POWER` | Same | Same sensor |
| Accuracy | FP32 exact | ±1 trit error | Hamming distance vs FP32 reference |

### The Three Benchmarks You Must Run

**1. Microbenchmark: tdot**

```python
# Vector dot product of length 1024
# FP32: 1024 mul-adds = 2048 operations
# Ternary: 64 XNOR+POPC pairs = 128 operations (16× reduction)

fp32_result = benchmark_tdot(length=1024, dtype="fp32")
ternary_result = benchmark_tdot(length=1024, dtype="ternary")

speedup = fp32_result["time_ms"] / ternary_result["time_ms"]
memory_ratio = fp32_result["memory_bytes"] / ternary_result["memory_bytes"]
```

Expected: 8-16× speedup, 16× memory reduction. The exact speedup depends on GPU generation (sm_80+ has fast POPC).

**2. Mesobenchmark: Matrix Multiply**

```python
# Square matrices, powers of 2
for size in [256, 512, 1024, 2048, 4096]:
    fp32 = benchmark_matmul(size, size, size, "fp32")
    tern = benchmark_matmul(size, size, size, "ternary")
    print(f"{size:5d}: {fp32['time_ms']/tern['time_ms']:5.2f}x faster, "
          f"{fp32['memory_bytes']/tern['memory_bytes']:5.2f}x smaller")
```

Expected: Speedup increases with matrix size (better GPU utilization). Memory reduction is constant 16×.

**3. Macrobenchmark: End-to-End Inference**

```python
# Full neural network inference
# Same topology, different weight precision
model_fp32 = load_model("resnet18", weights="fp32")
model_ternary = load_model("resnet18", weights="ternary")

# Same input batch
batch = load_imagenet_batch(size=128)

fp32_time = benchmark_inference(model_fp32, batch)
ternary_time = benchmark_inference(model_ternary, batch)
```

Expected: 2-4× end-to-end speedup (not 16×, because only weights are ternary — activations are still FP32 in most architectures). Memory reduction is 2-3× (weights are 16× smaller, but activations dominate).

### What NOT to Measure

Don't compare:
- **Ternary vs INT8 quantization** — both are compressed; compare against FP32 baseline
- **Different algorithms** — `ternary_conv2d` vs `im2col+gemm` is unfair
- **CPU vs GPU** — measure ternary-GPU against FP32-GPU on the same device
- **Throughput without latency** — batch size 1024 might show great throughput but unacceptable latency

### Reproducibility Checklist

A valid ternary benchmark report includes:
- [ ] GPU model (e.g., NVIDIA A100 sm_80)
- [ ] Driver version and CUDA version
- [ ] Exact matrix dimensions
- [ ] Warmup iterations (≥10)
- [ ] Measured iterations (≥100)
- [ ] Standard deviation reported
- [ ] FP32 baseline from the same code path
- [ ] PTX/SASS instruction count (from `cuobjdump`)
- [ ] Memory bandwidth measured separately from compute

### The Honest Bottom Line

| Claim | Reality | Conditions |
|-------|---------|------------|
| 16× denser | True | Weights only, 2-bit packing |
| 31× fewer ops | True | Per-dot-product, XNOR+POPC vs mul-add |
| 16× faster | False | Real speedup is 2-8× depending on workload |
| 60% power savings | True | Huawei's measurement, ternary-only accelerator |

Ternary is not magic. It's a representation shift that enables specific hardware optimizations. The 16× density is real. The speedup depends on whether you're memory-bound or compute-bound.

## Connect

- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — Why XNOR+POPC is faster than multiply-add
- [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) — How the 2-bit encoding enables density claims
- [FLUX-TO-PTX.md](FLUX-TO-PTX.md) — How to generate PTX and count instructions for fair comparison
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math that makes the benchmarks meaningful

## Activate

Run the golden benchmark yourself:

```bash
# Clone the benchmark suite
git clone https://github.com/SuperInstance/ternary-benchmarks
cd ternary-benchmarks

# Run microbenchmark
python benchmark_tdot.py --length 1024 --device cuda:0

# Run matrix benchmark
python benchmark_matmul.py --sizes 256,512,1024,2048 --device cuda:0

# Generate report
python report.py --output my_benchmark.json
```

Validate any ternary speedup claim:
1. Did they compare against FP32 on the same GPU?
2. Did they measure the same algorithm?
3. Did they report memory bandwidth separately?
4. Did they include standard deviation?

If any answer is no, the benchmark is suspect. Representation shifts are powerful, but only honest measurement reveals how powerful.
