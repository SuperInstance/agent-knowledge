# Ternary Quantization

> The most radical quantization is not 4-bit or 2-bit; it is {-1, 0, +1} bit.

## HOOK

Ternary quantization asks: what if a neural network weight could only be negative, zero, or positive? The answer is not merely a compression trick. It is a different geometry of computation in which sparsity, symmetry, and information capacity are unified in a single symbol set.

## REVEAL

In conventional deep learning, weights are high-precision scalars. Quantization reduces precision: FP32 → FP16 → INT8 → INT4. Each step saves memory and bandwidth but sacrifices dynamic range and expressivity.

Ternary quantization takes a different path. It replaces the real line with three symbols:

- `-1` — inhibitory connection
- `0` — absent connection
- `+1` — excitatory connection

This is not INT2. INT2 has four states: `-2, -1, 0, +1` or `0, 1, 2, 3`. Ternary deliberately discards magnitude and keeps only sign and absence. The result is a weight matrix that is simultaneously:

1. **Extremely compact**: Each weight needs ~1.585 bits in theory; packed trit encodings store 20 trits in 32 bits (THE-PACKED-FORMAT.md).
2. **Naturally sparse**: Zero weights require no computation, no memory bandwidth, and no energy.
3. **Symmetrically balanced**: The distribution is centered at zero, which eliminates weight-scale calibration drift.

### From Real Weights to Ternary Weights

The standard training recipe has three stages:

1. **Full-precision pretraining**: Train a model with FP32 or BF16 weights until convergence.
2. **Ternary projection**: Replace each real weight `w` with `sign(w)` if `|w|` exceeds a learned threshold `Δ`, else `0`. The threshold is trained with straight-through estimators.
3. **Fine-tuning in ternary space**: Continue training with ternary weights, updating only the scaling factors and thresholds while the sign pattern is regularized.

The key insight is that the *pattern* of zeros and signs carries most of the model's representational capacity, while the magnitudes can be absorbed into layer-wise scaling factors.

### Activation Quantization

Ternary activations are harder than ternary weights because activations vary with input. Two strategies coexist in the ecosystem:

- **Full-precision activations with ternary weights**: The cheapest path to deployment. Weights are ternary; activations remain FP16 or INT8. This gives most of the memory savings with minimal accuracy loss.
- **Ternary activations via learned thresholds**: Each neuron emits `-1, 0, +1` based on a learned threshold. This enables pure ternary multiply-accumulate: the multiplication `a * w` becomes a sign lookup and a sparse accumulate.

### Multiply-Accumulate in Z₃

A ternary MAC operation is structurally simpler than a real MAC:

- If weight is `0`, skip entirely.
- If weight is `+1`, add activation to accumulator.
- If weight is `-1`, subtract activation from accumulator.

This maps directly to GPU warp shuffle and tensor-core-like operations. The PTX emission in `oxide-sandbox` encodes exactly this logic for `TADD` and `TMUL` operations, verifying that ternary arithmetic is closed in Z₃.

## Experiments

1. **Accuracy vs. compression**: Ternarize a 1B-parameter transformer and measure perplexity degradation relative to FP16, INT8, and INT4 baselines. Expected: ternary weights with FP16 activations typically lose 1-3% accuracy at 16× compression.
2. **Throughput scaling**: On an A100, benchmark a ternary matmul kernel against cuBLAS FP16 for square matrices of size 512, 1024, 2048, 4096. Expected: ternary wins on memory-bound shapes; FP16 wins on compute-bound shapes.
3. **Sparsity emergence**: Track the percentage of zero weights during ternary fine-tuning. Expected: 30-70% sparsity emerges without explicit pruning pressure.
4. **Conservation check**: Verify that a ternary neural network's forward pass preserves the conservation laws documented in CONSERVATION-LAWS.md — in particular, that information flow is neither created nor destroyed by the quantization mapping.

## Applications

- **Edge deployment**: Run large models on GPUs with limited VRAM by storing weights at 1.6 bits each.
- **Inference cost reduction**: Reduce memory bandwidth by 10-16×, which often dominates transformer latency at batch size 1.
- **Sparse attention**: Ternary attention masks naturally encode block-sparse or pattern-sparse attention structures.
- **Model distillation**: Train small ternary students from full-precision teachers for environments where every watt matters.
- **Federated learning**: Ternary gradients reduce communication overhead between edge devices and central servers.

## Open Questions

1. **Dynamic thresholds**: Should thresholds be per-layer, per-channel, or per-token? Each choice trades accuracy for implementation complexity.
2. **Ternary training from scratch**: Can we train high-quality models without the full-precision pretraining stage?
3. **Nonlinearities in Z₃**: ReLU and GELU do not map cleanly to {-1, 0, +1}. What is the right ternary activation function?
4. **Hardware support**: Current GPUs lack native trit operations. How much speedup is possible with custom CUDA kernels versus what would require silicon changes?

## CONNECT

- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The arithmetic foundation of {-1, 0, +1}.
- [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) — How 20 trits fit in 32 bits for storage.
- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — Why GPUs are the right execution substrate for ternary computation.
- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — How to measure the 16× claim rigorously.
- [INTEGRATING-WITH-EXISTING-ML.md](INTEGRATING-WITH-EXISTING-ML.md) — Practical PyTorch and TensorFlow deployment paths.
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Invariants that must hold before and after quantization.
- `oxide-sandbox` — Validates ternary operations in the Flux→PTX pipeline.

## ACTIVATE

Take a model you already have trained in FP16 or FP32. Implement ternary projection on one layer: replace weights with `sign(w)` if `|w| > Δ`, else `0`. Choose `Δ` as the mean absolute weight of that layer. Run inference on 100 examples and measure the accuracy delta. Then pack 20 ternary weights into a 32-bit integer using the encoding in THE-PACKED-FORMAT.md. Compute the memory reduction. If the accuracy drop is acceptable, expand ternary projection to the entire model and benchmark end-to-end latency.
