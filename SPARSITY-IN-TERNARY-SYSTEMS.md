# Sparsity in Ternary Systems

> Sparsity is not compression. It is the absence that makes presence meaningful.

## HOOK

In a ternary system, zero is not a rounding error or a truncation artifact. It is a first-class symbol that means "this connection does not exist" — and that meaning creates structured sparsity for free.

## REVEAL

Sparse systems are computationally attractive because skipping zeros saves memory, bandwidth, and energy. But inducing sparsity in traditional neural networks is awkward. You train a dense model, prune small weights to zero, fine-tune to recover accuracy, and repeat. The result is often unstructured sparsity, which modern hardware struggles to accelerate.

Ternary systems solve this problem at the representation level. Because weights are constrained to `{-1, 0, +1}`, sparsity is not imposed after training. It is discovered during training. The model learns which connections are necessary (`+1` or `-1`) and which are unnecessary (`0`). The resulting sparsity is:

- **Structured by construction**: Zero weights are explicit symbols, not approximate zeros.
- **Naturally clustered**: Ternary training tends to produce block-sparse patterns because adjacent features often share relevance.
- **Hardware-friendly**: A zero weight means "skip this MAC," which is a simple control decision rather than a complex sparse matrix indexing problem.

### The Geometry of Ternary Sparsity

Consider a weight matrix `W ∈ {-1, 0, +1}^{m×n}`. Its sparsity ratio is the fraction of zeros. But sparsity ratio alone is a poor metric. What matters is:

1. **Row sparsity**: Which output neurons have few active inputs?
2. **Column sparsity**: Which input features are used by few output neurons?
3. **Block sparsity**: Are zeros clustered into contiguous submatrices?
4. **Dynamic sparsity**: Does the sparsity pattern change with input, or is it static?

Ternary weight training produces high static sparsity — typically 50-70% zeros in fully connected layers. Ternary activation thresholds can produce additional dynamic sparsity: for a given input, many neurons output `0`, creating a transient sparse activation pattern.

### Static vs. Dynamic Sparsity

Static sparsity lives in the weight matrix. It can be precomputed, stored in sparse formats, and exploited by kernels specialized to the pattern. Dynamic sparsity lives in activations. It varies per input and must be detected at runtime.

Static sparsity benefits:
- Smaller model size.
- Lower memory bandwidth.
- Simpler deployment.

Dynamic sparsity benefits:
- Conditional computation: skip entire subnetworks for irrelevant inputs.
- Adaptive depth: early-exit mechanisms.
- Runtime efficiency gains beyond what static sparsity provides.

The SuperInstance ecosystem pursues both. `oxide-slotmap` manages static sparse allocations; `oxide-circuit-breaker` and `oxide-canary` create dynamic sparsity in the control plane by skipping failed or rolled-back kernels.

### Sparsity and Information

There is a deep connection between sparsity and the conservation laws in CONSERVATION-LAWS.md. A dense matrix carries information in every entry. A sparse ternary matrix carries information only in the position and sign of nonzero entries. The zeros are not missing information; they are the structure that makes the nonzeros informative.

This mirrors biological neural networks, where synaptic pruning during development creates adult connectivity patterns. Sparsity is not damage. It is the learned shape of relevance.

## Experiments

1. **Sparsity emergence**: Train a ternary MLP on MNIST and plot zero-weight percentage per epoch. Expected: rapid initial drop to 30-50% sparsity, followed by slow refinement.
2. **Block detection**: Apply a simple clustering algorithm to a ternary weight matrix and measure the percentage of zeros that fall inside 8×8 or 16×16 zero blocks. Expected: >60% of zeros are block-structured in well-trained models.
3. **Dynamic activation sparsity**: Forward 1000 examples through a ternary activation network and measure per-example activation sparsity. Expected: 20-40% of activations are zero, depending on threshold.
4. **Speedup vs. density**: Benchmark a ternary sparse matmul kernel at 0%, 25%, 50%, 75%, and 90% sparsity. Expected: sublinear speedup because memory bandwidth, not computation, is the bottleneck.

## Applications

- **Sparse attention**: Ternary attention masks naturally encode which tokens should attend to which others, reducing quadratic attention cost.
- **Expert routing**: In mixture-of-experts models, ternary gates can select exactly one expert (`+1`) or skip (`0`) with no fractional ambiguity.
- **Feature selection**: Column sparsity reveals which input features the model actually uses, enabling interpretability and sensor pruning.
- **Edge deployment**: Sparse ternary models fit on microcontrollers with severe memory constraints.
- **Federated personalization**: Different clients can learn different sparse patterns on top of a shared dense base, enabling private customization.

## Open Questions

1. **Optimal sparsity**: Is there a universal sparsity level that maximizes accuracy per bit, or is it task-dependent?
2. **Unstructured vs. block sparsity**: How much accuracy do we lose by forcing zeros into blocks for hardware efficiency?
3. **Dynamic threshold learning**: Can we learn per-neuron activation thresholds that maximize dynamic sparsity without hurting accuracy?
4. **Sparsity in recurrent loops**: How does ternary sparsity behave in recurrent or stateful models where connections are reused over time?

## CONNECT

- [TERNARY-QUANTIZATION.md](TERNARY-QUANTIZATION.md) — How {-1, 0, +1} weights are trained.
- [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) — Storing sparse ternary matrices compactly.
- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — Measuring speedups from sparsity.
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Why information lives in structure, not density.
- [VISUALIZING-TERNARY-DATA.md](VISUALIZING-TERNARY-DATA.md) — Making sparse patterns visible.
- `oxide-slotmap` — Allocates sparse GPU resources with ternary state semantics.

## ACTIVATE

Take a dense layer from a model you have trained. Project its weights to ternary `{-1, 0, +1}` using the method in TERNARY-QUANTIZATION.md. Compute the sparsity ratio. Then visualize the weight matrix as a heatmap where `-1` is red, `0` is white, and `+1` is blue. Identify any block-sparse or row-sparse patterns by eye. If patterns exist, implement a blocked sparse matmul kernel that skips all-white blocks entirely. Benchmark it against dense matmul on 1000 random inputs and report the speedup.
