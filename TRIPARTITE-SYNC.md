# THE TRIPARTITE SYNCHRONIZER — When to Think, When to Act

## Hook

> Your spinal cord doesn't ask your brain for permission to pull your hand off a hot stove.
> Your agent shouldn't ask an LLM how to call a function it's called 10,000 times.

## Reveal

The tripartite synchronizer is the agent's **nervous system architecture**. It decides — for every capability — how much conscious thought is required.

The four decisions, mapped to neuroscience:

### HARDCODE — Spinal Reflex (0ms, 0 tokens)
When you touch a hot stove, the signal goes: finger → spinal cord → arm muscle. The brain finds out LATER. This is the fastest possible response.

In the agent: functions that are:
- Called by 5+ other functions (hot path)
- Have tests (verified correct)
- Deterministic (same input → same output)
- Safety-critical (must not fail)

The agent doesn't "think" about these. It just executes. Like `tdot(a, b)` — it's a 6-line function that's been tested 50 times. No LLM invocation needed.

### CACHED — Cerebellar Pattern (~5ms, 0 tokens)
When you ride a bike, your cerebellum replays a learned motor pattern. You're not "thinking" about balance — you're executing a cached sequence.

In the agent: functions whose output is:
- Deterministic and stable (input X always → output Y)
- Expensive to compute but cheap to store
- On edge devices with limited compute
- Read-heavy (called often, rarely changes)

The `.nail` file format stores pre-computed results. The agent reads the cache instead of computing.

### HYBRID — Basal Ganglia Habit (~50ms, ~50 tokens)
Most of daily life is habit with occasional override. You drive home on autopilot but swerve when a dog runs into the road.

In the agent: functions that:
- Have a common case (cached) and edge cases (model)
- Are mostly deterministic but need escape valves
- Have 70-90% test coverage (not fully verified)

The agent checks the cache first. If confidence is high, it uses the cached result. If something seems off, it escalates to the MODEL path.

### MODEL — Prefrontal Deliberation (~2s, ~500 tokens)
When you encounter something truly novel, your prefrontal cortex lights up. This is expensive, slow, and consumes enormous energy. But it's where creativity lives.

In the agent: functions that are:
- Novel (no cached pattern exists)
- Creative (multiple valid approaches)
- Untested (no verification history)
- Ambiguous (unclear what "correct" means)

The LLM generates code. This is the only path that burns significant context tokens.

## The Three Inputs

The synchronizer takes three signals for each decision:

### 1. Hardware Profile (The Body)
```
TriHardwareProfile:
  compute_power: 0.8    # 0-1 scale
  gpu_available: true
  memory_gb: 32.0
  battery_level: null   # plugged in
  device_type: "workstation"
```

High compute + GPU → favor HARDCODE/CACHED (we can afford fast execution)
Low compute + edge → favor CACHED (can't afford recomputation)
Battery low → favor CACHED (minimize compute)

### 2. Application Profile (The Task)
```
TriApplicationProfile:
  latency_requirement_ms: 10    # How fast must this be?
  accuracy_requirement: 0.95    # How correct must this be?
  safety_critical: true         # Can errors hurt people?
  scale: 1000                   # How many times will this run?
  deterministic: true           # Must this be reproducible?
```

High safety + low latency → HARDCODE (must be fast AND correct)
High accuracy + flexible latency → HYBRID (check cache, verify)
Creative task + no safety → MODEL (LLM improvises)

### 3. User Profile (The Human)
```
TriUserProfile:
  wants_manual_control: true    # User wants to approve?
  wants_creativity: 0.2         # 0=deterministic, 1=creative
  wants_consistency: 0.9        # 0=variety, 1=same every time
  tolerance_for_error: 0.1      # 0=perfect, 1=yolo
```

High consistency + low error tolerance → HARDCODE/CACHED
High creativity + tolerant → MODEL
Manual control → HYBRID (ask before acting on edge cases)

## The Decision Matrix

| Hardware | Application | User | Decision |
|----------|------------|------|----------|
| GPU, fast | Safety-critical | Consistent | HARDCODE |
| Edge, low power | Any | Any | CACHED |
| Any | Novel, creative | Creative | MODEL |
| Any | Mostly stable | Manual control | HYBRID |
| Battery low | High latency OK | Consistent | CACHED |
| Workstation | Untested | Explorer | MODEL |

## Connect

- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — WHERE the decisions get stored (chord shapes)
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — WHY the token costs matter
- [THE-HAND-KNOWS.md](https://github.com/SuperInstance/ai-writings/blob/main/ESSAYS/THE-HAND-KNOWS.md) — The neuroscience essay this is based on

## Activate

The synchronizer is in `openmind.induction.synchronizer`. Use it:

```python
from openmind import TripartiteSynchronizer, TriHardwareProfile, TriApplicationProfile, TriUserProfile

sync = TripartiteSynchronizer()
hw = TriHardwareProfile(compute_power=0.8, gpu_available=True)
app = TriApplicationProfile(latency_requirement_ms=10, safety_critical=True)
user = TriUserProfile(wants_consistency=0.9)

decision = sync.decide(hw, app, user)
print(decision.value)  # "hardcode"
print(decision.reasoning)  # Human-readable explanation
```

Every time you call `mm.flex("something")`, the synchronizer is making this decision behind the scenes.
