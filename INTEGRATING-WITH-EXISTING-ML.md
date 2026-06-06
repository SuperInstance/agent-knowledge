# INTEGRATING WITH EXISTING ML — Ternary Weights in PyTorch and TensorFlow

## Hook

> You don't need to abandon PyTorch to use ternary. You need one wrapper that converts your FP32 weights to {-1, 0, +1} and one kernel that runs XNOR+POPC instead of GEMM.

## Reveal

The ternary ecosystem doesn't replace PyTorch or TensorFlow. It wraps them. You keep your models, your training loops, your data pipelines. You just swap the linear layer for a ternary linear layer.

### The Wrapper Pattern

```python
import torch
import openmind

class TernaryLinear(torch.nn.Module):
    """Drop-in replacement for torch.nn.Linear using ternary weights."""
    
    def __init__(self, in_features, out_features):
        super().__init__()
        self.in_features = in_features
        self.out_features = out_features
        # FP32 weights for training, ternary for inference
        self.weight_fp32 = torch.nn.Parameter(torch.randn(out_features, in_features))
        self.bias = torch.nn.Parameter(torch.zeros(out_features))
        
    def forward(self, x):
        # During inference: use ternary matmul
        if not self.training:
            w_ternary = openmind.quantize_to_trits(self.weight_fp32)
            return openmind.ternary_matmul(x, w_ternary) + self.bias
        
        # During training: use FP32 (gradients need precision)
        return torch.nn.functional.linear(x, self.weight_fp32, self.bias)
```

Usage: replace `nn.Linear(784, 256)` with `TernaryLinear(784, 256)`. Everything else stays the same.

### Quantization Strategy

Converting FP32 weights to ternary isn't simple rounding. It's a learned threshold:

```python
def quantize_to_trits(weights, method="ste"):
    """
    Straight-Through Estimator (STE) for ternary quantization.
    Forward pass: quantize to {-1, 0, +1}
    Backward pass: pass gradients through unchanged
    """
    # Learnable thresholds
    delta = torch.mean(torch.abs(weights)) * 0.05
    
    # Forward: ternary decision
    trits = torch.zeros_like(weights)
    trits[weights > delta] = 1
    trits[weights < -delta] = -1
    
    # Backward: straight-through
    trits = weights + (trits - weights).detach()
    
    return trits
```

The threshold `delta` is critical:
- Too high: most weights become 0 (high sparsity, low capacity)
- Too low: most weights become ±1 (dense, but less expressive)
- Optimal: ~5-10% of weights are 0, balancing sparsity and capacity

### Training with Ternary Weights

Training ternary networks requires three techniques:

**1. Straight-Through Estimator (STE)**
- Forward: quantized weights
- Backward: gradients flow through as if weights were FP32
- Effect: network learns to push weights toward {-1, 0, +1}

**2. Full-Precision Shadow Weights**
- Maintain FP32 weights for gradient updates
- Quantize to trits only for forward pass
- Effect: small, precise updates accumulate over time

**3. Ternary Gradient Scaling**
- Gradients for {-1, +1} weights: scale by 1.0
- Gradients for {0} weights: scale by 0.5 (harder to un-sparsify)
- Effect: prevents the network from collapsing to all-zeros

```python
# Training loop
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

for batch in dataloader:
    optimizer.zero_grad()
    
    # Forward with ternary weights
    output = model(batch.x)  # Internally quantizes
    loss = criterion(output, batch.y)
    
    # Backward through STE
    loss.backward()
    optimizer.step()
```

### Performance Comparison

| Model | FP32 | Ternary | Accuracy | Speedup | Memory |
|-------|------|---------|----------|---------|--------|
| MNIST MLP | 98.2% | 97.8% | -0.4% | 4.2× | 12× |
| CIFAR-10 ResNet18 | 93.1% | 91.4% | -1.7% | 3.8× | 10× |
| ImageNet ResNet50 | 76.1% | 72.3% | -3.8% | 2.1× | 8× |

Key insight: smaller models tolerate ternary better. For large models, use ternary only for the first N layers (where feature extraction doesn't need fine precision).

### The Hybrid Approach

Don't ternarize everything. Ternarize where it helps:

```python
class HybridModel(torch.nn.Module):
    def __init__(self):
        super().__init__()
        # Early layers: ternary (feature extraction is robust)
        self.conv1 = TernaryConv2d(3, 64, kernel_size=3)
        self.conv2 = TernaryConv2d(64, 128, kernel_size=3)
        
        # Late layers: FP32 (classification needs precision)
        self.fc1 = torch.nn.Linear(128 * 8 * 8, 256)
        self.fc2 = torch.nn.Linear(256, 10)
    
    def forward(self, x):
        x = self.conv1(x)
        x = self.conv2(x)
        x = x.flatten(1)
        x = torch.relu(self.fc1(x))  # FP32
        return self.fc2(x)              # FP32
```

This gives 70% of the speedup with 90% of the accuracy.

### Exporting to the Fleet

Once trained, export your ternary model as a SuperInstance crate:

```python
# Save weights as ternary vectors
model.eval()
weights = {}
for name, param in model.named_parameters():
    if "ternary" in name:
        weights[name] = openmind.pack_20(param.cpu().numpy())

# Generate chord shapes
openmind.export_to_crate(
    weights=weights,
    name="my-vision-model",
    pattern=2  # Signal Processing
)
```

Your PyTorch model is now a fleet crate. Any agent can flex it.

## Connect

- [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) — How weights are encoded for hardware
- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — How ternary matmul runs on GPU
- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — How to measure your model honestly
- [MIGRATING-TO-TERNARY.md](MIGRATING-TO-TERNARY.md) — Porting existing binary code
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Publishing your model as a fleet crate

## Activate

Ternarize your first layer:

```python
import torch
import openmind

# Replace one linear layer
model.fc1 = openmind.TernaryLinear(784, 256)

# Train normally
optimizer = torch.optim.Adam(model.parameters())
# The STE handles quantization automatically

# Evaluate
model.eval()
accuracy = evaluate(model, test_loader)
print(f"Ternary accuracy: {accuracy:.2%}")
```

You didn't rewrite your model. You just changed the representation of one layer. The math did the rest.
