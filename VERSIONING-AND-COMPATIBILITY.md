# VERSIONING AND COMPATIBILITY — 303 Crates, One Moving Target

## Hook

> Semantic versioning says: bump major on breaking changes. In a fleet of 303 crates, every change is potentially breaking. The SuperInstance version model doesn't prevent breakage — it detects it before it hurts.

## Reveal

Traditional semantic versioning (semver) assumes a tree: dependencies flow downward, and a major bump in a leaf doesn't affect the root. The ternary fleet is a ring, not a tree. Every crate is reachable from every other crate in three hops. A breaking change anywhere propagates everywhere.

### The Ternary Version Number

SuperInstance uses a 3-part version with ternary semantics:

```
MAJOR.MINOR.PATCH
  ↓     ↓     ↓
  -1     0    +1
```

- **MAJOR bump (-1 signal):** Breaking change. All dependent crates must re-verify.
- **MINOR bump (0 signal):** Compatible addition. New features, old code works.
- **PATCH bump (+1 signal):** Fix. Safer than before, no behavior change.

This isn't cosmetic. The version number IS a ternary signal that the tripartite synchronizer consumes:

```python
mm = openmind.MuscleMemory.load("fleet_muscles.json")
reflex = mm.flex("tdot")

if reflex.chord.version.major > loaded_major:
    # Breaking change detected
    decision = Trit.N1  # Don't execute until re-verified
```

### The Compatibility Matrix

Not all crates depend on all others. But the 12 transfer stations (see FLEET-MAP.md) are universal dependencies. When a transfer station bumps major, the entire fleet must respond.

| Changed Crate | Impact | Required Action |
|---------------|--------|-----------------|
| `ternary-core` | Fleet-wide | All 303 crates re-verify |
| `open-parallel` | All concurrent crates | Re-test async behavior |
| `flux-core` | All compiler crates | Re-validate bytecode |
| `cuda-oxide` | All GPU crates | Re-compile PTX, check witnesses |
| Peripheral crate | 1-2 dependents | Local re-test only |

The induction engine automates this. When `ternary-core` bumps from 0.47.0 to 0.48.0 (major), the engine:
1. Identifies all crates importing `ternary-core`
2. Re-runs their test suites
3. Checks their isomorphism scores
4. Generates a compatibility report

Crates that pass are "green." Crates that fail are "red" and pinned to the old version until fixed.

### Witness-Based Compatibility

Version compatibility isn't just "do the tests pass?" It's "do the proofs still hold?"

When `tdot` in `ternary-core` changes:
- The old witness: `popcount(xnor(a,b)) ≡ Σ(a[i]*b[i])` for v0.47
- The new witness: must prove the same for v0.48
- If the witness is identical, the change is internal (refactoring)
- If the witness differs, every crate using `tdot` must re-verify

This is conservation of verification entropy applied to versioning: a version bump must preserve or increase the total proof strength of the fleet.

### The Pin File

Each deployment specifies exact versions in a `fleet.pin` file:

```json
{
  "ternary-core": "0.47.1",
  "ternary-signals": "0.12.3",
  "open-parallel": "1.8.0",
  "flux-core": "2.1.4",
  "cuda-oxide": "0.91.2",
  "...": "..."
}
```

This is the only file that matters for reproducibility. Source code changes. APIs evolve. But a `fleet.pin` from six months ago still loads the exact same muscle memory, runs the exact same proofs, and produces the exact same results.

### Rolling Updates with Canary

Deploying a new version across the fleet:

```python
# 1. Load current pin
current = openmind.PinFile.load("fleet.pin")

# 2. Propose new version for one crate
proposal = current.clone()
proposal["ternary-core"] = "0.48.0"

# 3. Verify compatibility
report = openmind.verify_compatibility(proposal)
if report.failed_crates:
    print(f"Blocked: {len(report.failed_crates)} crates incompatible")
    for crate in report.failed_crates:
        print(f"  {crate.name}: {crate.failure_reason}")
    raise CompatibilityError()

# 4. Canary: deploy to 5% of nodes
conductor.update_pin(proposal, rollout=0.05)

# 5. Monitor conservation metrics
if conductor.average_entropy() >= current_average_entropy:
    conductor.update_pin(proposal, rollout=1.0)
else:
    conductor.rollback()
```

The canary doesn't just check for crashes. It checks that verification entropy didn't decrease. A version that passes all tests but breaks a witness is still rejected.

### Dependency Cycles

The ring geometry means cycles are normal, not errors:

```
ternary-music → ternary-core → ternary-types → ternary-music
```

This is a cycle. But it's not a problem because:
1. The dependency graph is acyclic at the function level
2. Cycles are resolved by build order: core first, then dependents
3. The induction engine proves that cyclic crate dependencies don't create infinite loops at runtime

Cycles in the crate graph are structural. Cycles in the call graph are bugs.

## Connect

- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — How to version your new crate before submitting
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Tests are the compatibility proofs
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Version bumps must preserve verification entropy
- [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md) — Rolling updates in production
- [FLEET-MAP.md](FLEET-MAP.md) — Transfer stations and the dependency ring

## Activate

Pin your fleet:

```python
import openmind

# Generate pin from current muscle memory
mm = openmind.MuscleMemory.build(openmind.ingest("./fleet"))
pin = mm.generate_pin()
pin.save("fleet.pin")

# Verify a proposed update
new_pin = openmind.PinFile.load("proposed.pin")
report = openmind.verify_compatibility(new_pin, baseline="fleet.pin")
print(f"Green: {len(report.green)} / Red: {len(report.red)}")

# Update with confidence
if report.all_green:
    conductor.update_pin(new_pin)
```

A pinned fleet is a reproducible fleet. A verified update is a safe update. The version number is a signal, not a promise.
