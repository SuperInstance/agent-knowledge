# DESIGN PRINCIPLES — The Rules That Govern Every Decision

## Hook

> Every design decision in this ecosystem answers to one of seven principles. Know the principles and you can predict the design without reading the source.

## Reveal

These principles are invariant. They don't change with fashion, hardware generation, or domain. Every crate, every API, every document follows them.

### Principle 1: The Trichotomy Principle

Every domain has three natural states. Find them. Make them explicit.

**Applies to:** All crates, all APIs, all data models
**Test:** Can you describe your domain as {-1, 0, +1}? If not, ternary is the wrong frame.
**Example:** Temperature → {too cold, just right, too hot}. Consensus → {against, abstain, for}.

---

### Principle 2: The Closure Principle

Operations must stay in the space. No overflow. No exceptions. No escape hatches.

**Applies to:** Core math, all algorithms
**Test:** For all inputs, does the output remain a valid trit?
**Example:** `tdot` always returns {-1, 0, +1}. `tadd` always returns {-1, 0, +1}.

---

### Principle 3: The Conservation Principle

Structured intention is invariant under transformation. Correctness, attention, and topology are conserved.

**Applies to:** Compilation, compression, migration
**Test:** Does the transformation preserve or increase verification entropy?
**Example:** Compiling Rust to PTX generates a witness. The proof transforms; it doesn't vanish.

---

### Principle 4: The Compression Principle

Knowledge should be compressed at build time, not decompressed at runtime.

**Applies to:** Muscle memory, chord shapes, nail files
**Test:** Can an agent use this capability without loading the source?
**Example:** `mm.flex("tdot")` executes without reading `ternary-core/src/lib.rs`.

---

### Principle 5: The Decentralization Principle

No central authority. Consensus emerges from local votes.

**Applies to:** Multi-agent systems, fleet coordination, access control
**Test:** Does the system work if any single node fails?
**Example:** `tdecide` requires no leader. Missing agents default to {0}.

---

### Principle 6: The Graceful Degradation Principle

Failure is not binary. Systems degrade through {0} before reaching {-1}.

**Applies to:** Fault tolerance, error handling, sensor failure
**Test:** Does the system produce {0} (safe pause) before crashing?
**Example:** A failing sensor returns {0} (uncertain) rather than garbage data.

---

### Principle 7: The Witness Principle

Every transformation must be provable. No trust without verification.

**Applies to:** Compilation, optimization, testing
**Test:** Can you generate a witness that proves correctness?
**Example:** `cuda-oxide` emits a witness for every PTX instruction mapping.

---

### How Principles Resolve Conflicts

When two principles conflict, priority order:

```
Closure > Conservation > Witness > Graceful Degradation > 
Decentralization > Compression > Trichotomy
```

Example conflict: A trichotomy isn't closed under some operation.
- Trichotomy says: {-1, 0, +1} are the states
- Closure says: the operation must stay in {-1, 0, +1}
- Resolution: Redefine the operation to be closed (higher priority)

Example conflict: Compression breaks a witness.
- Compression says: store only chord shapes
- Witness says: preserve proof of correctness
- Resolution: Include the witness in the chord shape (higher priority)

### The Principles in Practice

| Decision | Principles Applied | Outcome |
|----------|-------------------|---------|
| Use 2-bit encoding | Closure + Compression | Pack_20 stays in Z₃, 16× density |
| Generate PTX witnesses | Witness + Conservation | Correctness preserved across layers |
| Missing vote = {0} | Decentralization + Graceful Degradation | Consensus continues without leader |
| HARDCODE for core math | Compression + Witness | Fast execution, proven correct |
| MODEL for composition | Trichotomy + Graceful Degradation | Creative freedom, safe fallback |

### Adding an Eighth Principle

The ecosystem resists adding new principles. Seven is enough. Any proposed eighth principle must:
1. Be independent of the existing seven
2. Apply to all crates, all layers, all domains
3. Be falsifiable (there must exist a design that violates it)
4. Be more useful than the lowest-priority existing principle

No eighth principle has met these criteria yet. The seven stand.

## Connect

- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Principle 3 in mathematical form
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — Principle 6 in operational form
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Principle 7 in mechanical form
- [THE-COMPILED-AGENCY-THESIS.md](THE-COMPILED-AGENCY-THESIS.md) — Principle 4 in philosophical form
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Apply all seven principles when adding a crate

## Activate

Audit any design against the principles:

```python
import openmind

design = load_my_design()
audit = openmind.principle_audit(design)

for principle, result in audit.items():
    status = "PASS" if result.passes else "FAIL"
    print(f"{principle}: {status}")
    if not result.passes:
        print(f"  Fix: {result.recommendation}")
```

A design that passes all seven principles is fleet-ready. A design that fails one needs work. A design that fails three needs reconsideration.

The principles aren't constraints. They're the structure that makes freedom possible.
