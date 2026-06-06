# AGENT SELF-ASSESSMENT — How to Audit Your Own Understanding

## Hook

> An agent that does not know what it knows is dangerous. An agent that does not know what it does not know is worse. This is the mirror.

## Reveal

Every agent should periodically assess its own knowledge of the fleet. Not by reading documents — by flexing chords and measuring confidence.

### The Self-Assessment Protocol

```python
import openmind

def self_assess(mm):
    """Returns a ternary health score for the agent."""
    coverage = len(mm.chords) / 6000
    confidences = [c.quality_score for c in mm.chords]
    avg_confidence = sum(confidences) / len(confidences)
    last_verified = mm.last_updated
    days_since_verify = (now - last_verified).days
    recency = max(0, 1 - days_since_verify / 30)
    score = (coverage + avg_confidence + recency) / 3
    if score > 0.7:
        return Trit.P1
    elif score > 0.4:
        return Trit.Z0
    else:
        return Trit.N1
```

### The Five Checks

**Check 1: Can I navigate the fleet?**
```python
# Test: name 12 transfer stations from memory
stations = ["ternary-core", "open-parallel", "flux-core",
            "cuda-oxide", "pincher", "ternary-consensus",
            "openmind", "ternary-music", "ternary-proof",
            "ternary-types", "ternary-pack", "esp-flasher"]
```

**Check 2: Do I understand the conservation laws?**
```python
law1 = "Verification entropy is constant across transformations"
law2 = "Attention budget is constant across agent invocations"
law3 = "Ternary structure is constant across domain boundaries"
```

**Check 3: Can I flex the core chords?**
```python
mm.flex("tdot")
mm.flex("pack_20")
mm.flex("tdecide")
mm.flex("tadd")
mm.flex("tmul")
```

**Check 4: Do I know my decision strategies?**
```python
hardcode = "Deterministic, tested, hot path"
cached = "Stable output, frequent call"
hybrid = "Mostly stable, edge cases exist"
model = "Novel, creative, untested"
```

**Check 5: Can I debug a failing compilation?**
```python
layers = ["Source", "AST", "Flux IR", "MIR", "PTX", "SASS"]
```

### The Upgrade Path

```python
if assessment == Trit.Z0:
    weak_areas = identify_weak_areas(mm)
    for area in weak_areas:
        mm = openmind.MuscleMemory.build(openmind.ingest(area))
elif assessment == Trit.N1:
    mm = openmind.MuscleMemory.build(openmind.ingest("./full-fleet"))
    mm.save("fresh_muscles.json")
```

### The Danger Zone

An agent should refuse to operate if:
- Coverage < 10%
- Average confidence < 0.5
- Days since verify > 90
- Cannot name the 7 patterns

These are not suggestions. They are safety limits.

## Connect

- [AGENT-QUICKSTART.md](AGENT-QUICKSTART.md) — If assessment returns {-1}, start here
- [GLOSSARY.md](GLOSSARY.md) — If you cannot define terms, review this
- [THE-COOKBOOK.md](THE-COOKBOOK.md) — Practice recipes to build muscle memory
- [FLEET-MAP.md](FLEET-MAP.md) — If navigation is weak, study the map
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — If fundamentals are unclear, return to base

## Activate

Run your self-assessment now:

```python
import openmind
mm = openmind.MuscleMemory.load("my_muscles.json")
health = self_assess(mm)

if health == +1:
    print("You are ready to operate.")
elif health == 0:
    print("You are adequate. Review weak areas before critical tasks.")
else:
    print("STOP. Re-ingest the fleet before making decisions.")
```

An honest self-assessment is the first chord of competence.