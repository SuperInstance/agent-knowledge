# COST ECONOMICS — The Dollar Math of Ternary Computing

## Hook

> A 16× density advantage sounds abstract until you translate it to cloud bills. At scale, ternary isn't just faster — it's cheaper by orders of magnitude.

## Reveal

Performance benchmarks measure speed. Cost economics measure survival. A system that is fast but unaffordable dies in production. This document translates ternary advantages into dollars.

### The Cost Model

Cloud computing has three cost dimensions:

| Dimension | Binary Cost | Ternary Cost | Savings |
|-----------|------------|--------------|---------|
| Compute | $3.06/hour (A100) | $3.06/hour (same hardware) | 0% hardware, 4× throughput |
| Memory | $0.80/GB/month | $0.80/GB/month (same price) | 16× less memory needed |
| Bandwidth | $0.09/GB | $0.09/GB (same price) | 16× less data moved |

The hardware cost is the same. The savings come from doing more work with the same hardware.

### Scenario 1: Embedding Search Service

**Requirements:** 10M documents, 1000 QPS, 99th percentile < 50ms

**Binary (FP32):**
- Embeddings: 10M × 768 dims × 4 bytes = 29.3 GB
- GPU memory: 8× A100 (40GB each) = $24.48/hour
- Query throughput: 250 QPS per A100 → need 4 A100s for 1000 QPS
- Total: $12.24/hour, $8,978/month

**Ternary:**
- Embeddings: 10M × 768 dims × 0.25 bytes = 1.83 GB
- GPU memory: 1× A100 (fits in 2GB)
- Query throughput: 1000 QPS per A100 (4× speedup)
- Total: $3.06/hour, $2,244/month

**Savings: $6,734/month (75%)**

### Scenario 2: IoT Sensor Network

**Requirements:** 10,000 sensors, 1 message/minute, 1-year retention

**Binary (JSON):**
- Message size: 200 bytes (JSON with metadata)
- Monthly data: 10,000 × 200B × 43,200 = 86.4 GB/month
- MQTT broker: $0.50/GB = $43.20/month
- Storage (1 year): 86.4 × 12 × $0.023/GB = $23.85/month
- Total: $67.05/month

**Ternary (trits):**
- Message size: 3 bytes (3 trits)
- Monthly data: 10,000 × 3B × 43,200 = 1.3 GB/month
- MQTT broker: $0.50/GB = $0.65/month
- Storage (1 year): 1.3 × 12 × $0.023/GB = $0.36/month
- Total: $1.01/month

**Savings: $66.04/month (98.5%)**

### Scenario 3: LLM API Service

**Requirements:** Serve a 7B parameter model, 500 QPS

**Binary (FP16):**
- Model size: 7B × 2 bytes = 14 GB
- GPUs: 2× A100 (each loaded with 14GB) = $6.12/hour
- Throughput: 250 QPS → need 2 GPUs
- Total: $6.12/hour, $4,488/month

**Ternary (hybrid):**
- Model size: 7B × 0.25 bytes = 1.75 GB (weights only, activations still FP16)
- GPUs: 1× A100 (fits easily)
- Throughput: 400 QPS (1.6× speedup from memory bandwidth)
- Need 2 GPUs for 500 QPS, but each GPU is 4× less loaded
- Total: $6.12/hour, $4,488/month (same cost, higher margin)

**Insight:** For LLMs, ternary doesn't reduce GPU count because activations dominate. But it enables larger batch sizes, lower latency, and higher throughput per dollar. The economic win is **capacity**, not cost reduction.

### The Hidden Costs

| Cost | Binary | Ternary | Notes |
|------|--------|---------|-------|
| Development time | Baseline | +20% | Need to learn Z₃, write property tests |
| Debugging time | Baseline | -40% | Property tests catch bugs earlier |
| Maintenance | Baseline | -30% | Muscle memory makes updates safer |
| Talent acquisition | Baseline | +50% | Fewer ternary engineers available |

Net development cost: roughly equal after 6 months. Ternary has higher upfront learning but lower ongoing maintenance.

### Break-Even Analysis

When does ternary pay for itself?

```python
def break_even_months(development_cost, monthly_savings):
    return development_cost / monthly_savings

# Embedding search: $10k dev cost, $6,734/month savings
print(break_even_months(10000, 6734))  # 1.5 months

# IoT network: $5k dev cost, $66/month savings
print(break_even_months(5000, 66))  # 76 months (too long!)

# LLM service: $50k dev cost, $0/month direct savings
print(break_even_months(50000, 0))  # Never (but capacity doubles)
```

Ternary makes economic sense when:
- Memory is the bottleneck (embedding search, IoT at scale)
- Query volume is high (throughput gains translate to fewer machines)
- The domain has a natural trichotomy (development cost is low)

### The Economic Conservation Law

Just as verification entropy is conserved, **economic value is conserved** across transformations:

```
Value = (Throughput × Quality) / (Cost × Latency)
```

Ternary increases throughput (16× density) and decreases cost (fewer machines). Quality may decrease slightly (3.8% accuracy drop in worst case). Latency decreases (faster memory).

For most applications, the value equation improves 2-4×.

## Connect

- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — The performance numbers behind the cost model
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — Token economics for agents
- [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md) — Running ternary systems at scale
- [CASE-STUDIES.md](CASE-STUDIES.md) — Real deployments with real cost savings

## Activate

Calculate your break-even:

```python
import openmind

analysis = openmind.cost_analysis(
    current_system="my_fp32_service",
    proposed_system="my_ternary_service",
    qps=1000,
    data_size_gb=29.3,
    development_cost_usd=15000
)

print(f"Monthly savings: ${analysis.monthly_savings:,.2f}")
print(f"Break-even: {analysis.break_even_months:.1f} months")
print(f"3-year ROI: {analysis.roi_3year:.1f}%")

if analysis.break_even_months < 6:
    print("Ternary is economically compelling.")
else:
    print("Consider a pilot project to validate assumptions.")
```

At scale, ternary isn't an optimization. It's a business model.
