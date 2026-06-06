# THE COOKBOOK — Recipes for Common Ternary Tasks

## Hook

> A recipe is not theory. It is "do this, then this, then this." Here are 12 recipes that solve the most common problems agents encounter in the fleet.

## Reveal

### Recipe 1: Ingest a Codebase and Build Muscle Memory

```bash
# Step 1: Ingest
openmind ingest ./my-project --output my_muscles.json

# Step 2: Verify
openmind stats my_muscles.json
# Should show: chord count, coverage, average entropy

# Step 3: Test a flex
openmind flex my_muscles.json "main"
```

**Expected result:** A JSON file under 5MB containing all chord shapes from your project.

---

### Recipe 2: Deploy a Single Ternary Agent

```python
import openmind

# Load muscle memory
mm = openmind.MuscleMemory.load("production_muscles.json")

# Define the agent loop
def agent_loop():
    signal = mm.flex("read_sensor").trit
    if signal == +1:
        mm.flex("actuate_cooler")
    elif signal == -1:
        mm.flex("actuate_heater")
    # signal == 0: do nothing

# Run
while True:
    agent_loop()
    time.sleep(30)
```

**Expected result:** Agent reads sensors and acts every 30 seconds, burning zero tokens.

---

### Recipe 3: Set Up Multi-Agent Consensus

```python
import openmind

# All agents load the same score
mm = openmind.MuscleMemory.load("consensus_score.json")

# Each agent has a vote
agent_votes = {
    "agent-a": +1,
    "agent-b": +1,
    "agent-c": -1,
    "agent-d": 0
}

# Resolve
result = mm.flex("tdecide", votes=list(agent_votes.values()))
print(result)  # +1 (majority positive)
```

**Expected result:** Consensus reached in less than 1ms without central coordinator.

---

### Recipe 4: Compile a Ternary Kernel for GPU

```bash
# Write Flux source
cat > similarity.flux << FLUX
fn similarity(query: &[Trit], docs: &Matrix<Trit>) -> Vec<i32> {
    docs.rows().map(|row| tdot(query, row)).collect()
}
FLUX

# Compile to PTX
openmind compile similarity.flux --target sm_80 --output kernel.ptx

# Verify witness
openmind verify kernel.ptx --against similarity.flux
```

**Expected result:** PTX file with valid witness. XNOR+POPC instructions present.

---

### Recipe 5: Quantize a PyTorch Model to Ternary

```python
import torch
import openmind

# Load model
model = torch.load("my_model.pth")

# Replace Linear layers
for name, module in model.named_modules():
    if isinstance(module, torch.nn.Linear):
        ternary_layer = openmind.TernaryLinear(
            module.in_features,
            module.out_features
        )
        ternary_layer.weight_fp32.data = module.weight.data.clone()
        setattr(model, name, ternary_layer)

# Validate
accuracy = evaluate(model, test_loader)
assert accuracy > 0.90  # Should retain >90% accuracy
```

**Expected result:** Model runs with ternary weights, ~4x faster inference.

---

### Recipe 6: Cache Expensive Computations

```python
import openmind

mm = openmind.MuscleMemory.load("my_muscles.json")

# First call: computes and caches
result = mm.flex("expensive_analysis", data=input_a)
mm.save_nails("cache.nail")

# Later calls: instant
mm.load_nails("cache.nail")
result = mm.flex("expensive_analysis", data=input_a)  # 0 tokens
```

**Expected result:** Second call returns in microseconds, not seconds.

---

### Recipe 7: Debug a Failing Compilation

```bash
# Compile with full trace
openmind compile broken.flux --trace --emit-all

# Check each layer
for layer in ast ir mir ptx sass; do
    echo "=== $layer ==="
    cat "broken.$layer" | grep -i "error\|fail\|invalid"
done

# Most common fix: check Z3 closure
# If operations do not stay in {-1,0,+1}, the witness fails
```

**Expected result:** Identify which compilation stage diverges.

---

### Recipe 8: Benchmark Ternary vs FP32

```python
import openmind

sizes = [256, 512, 1024, 2048]
for size in sizes:
    fp32 = openmind.benchmark("matmul", size=size, dtype="fp32")
    tern = openmind.benchmark("matmul", size=size, dtype="ternary")

    speedup = fp32["time_ms"] / tern["time_ms"]
    memory_ratio = fp32["memory_bytes"] / tern["memory_bytes"]

    print(f"{size:4d}: {speedup:5.2f}x faster, {memory_ratio:5.2f}x smaller")
```

**Expected result:** Speedup 2-8x, memory reduction 16x.

---

### Recipe 9: Migrate a Binary Function to Ternary

```python
# Before: binary
def status(temp):
    return "ok" if 20 < temp < 30 else "bad"

# After: ternary
def status(temp):
    if temp > 30: return openmind.Trit.P1   # too hot
    if temp < 20: return openmind.Trit.N1   # too cold
    return openmind.Trit.Z0                  # just right

# Validate equivalence
for temp in [15, 22, 35]:
    old = "ok" if 20 < temp < 30 else "bad"
    new = status(temp)
    mapping = {"ok": 0, "bad": -1 if temp < 20 else 1}
    assert new == mapping[old]
```

**Expected result:** All test temperatures produce equivalent outputs.

---

### Recipe 10: Monitor Fleet Health

```python
import openmind

mm = openmind.MuscleMemory.load("fleet_muscles.json")

# Three conservation metrics
entropy = mm.average_entropy()
budget = mm.attention_budget_ratio()
isomorphism = mm.isomorphism_with("ternary-core")

print(f"Entropy: {entropy:.2f} (target >= 3.0)")
print(f"Budget: {budget:.2f}x (target >= 2.0)")
print(f"Isomorphism: {isomorphism:.4f} (target >= 0.97)")

if entropy < 3.0 or budget < 2.0 or isomorphism < 0.97:
    alert("Conservation violation detected")
```

**Expected result:** All metrics green = healthy fleet.

---

### Recipe 11: Add a Crate to the Fleet

```bash
# 1. Scaffold
cargo new --lib ternary-mydomain
cd ternary-mydomain
cargo add ternary-core

# 2. Implement (in src/lib.rs)
# - 3 core ternary operations
# - 10 property tests

# 3. Ingest and verify
openmind ingest .
openmind flex "my_operation"

# 4. Run induction
openmind induct .
# Should show: alpha_3 >= 0.97, entropy >= 2

# 5. Submit
git push origin main
gh pr create --title "feat: ternary-mydomain"
```

**Expected result:** Induction engine passes, PR auto-merges after green CI.

---

### Recipe 12: Recover from Agent Failure

```python
import openmind

# Agent crashes. Remaining agents detect absence.
conductor = openmind.Conductor(score=mm)

# Missing agent votes default to {0}
# Consensus continues with reduced confidence
result = conductor.decide(current_votes)

if conductor.confidence < 0.5:
    # Too few active agents
    conductor.scale_up(replacement_count=1)
```

**Expected result:** System continues operating. Replacement agent spawned automatically.

## Connect

- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — Deep dive on the flex API
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Full guide to adding crates
- [DEBUGGING-AND-TRACING.md](DEBUGGING-AND-TRACING.md) — When recipes do not work
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — Common problems and fixes
- [AGENT-QUICKSTART.md](AGENT-QUICKSTART.md) — First-time setup

## Activate

Pick a recipe. Run it. If it works, you understand the ecosystem. If it does not, read the linked document and try again.

The fleet is learned by doing, not by reading.
