# GOVERNANCE AND COMMUNITY — How the Ecosystem Decides

## Hook

> A fleet of 303 crates with no central authority sounds like chaos. It's not. The governance is ternary too: {-1} veto, {0} abstain, {+1} merge. The math handles the rest.

## Reveal

SuperInstance has no BDFL (Benevolent Dictator for Life), no corporate steering committee, and no token-holder votes. Decisions emerge from three mechanisms: the induction engine, the test suite, and the conservation laws.

### The Three Governing Forces

| Force | Role | Decision Power | Based On |
|-------|------|---------------|----------|
| Induction Engine | Gatekeeper | Accept/Reject crates | α₃ ≥ 0.97, entropy ≥ 2 |
| Test Suite | Judiciary | Pass/Fail changes | Property proofs, witnesses |
| Conservation Laws | Constitution | Bound all decisions | Entropy, attention, structure |

No human can override the induction engine. No human can merge a crate that decreases verification entropy. The rules are code, not policy.

### How Crates Enter the Fleet

```
Researcher writes crate
        ↓
Runs `openmind induct .`
        ↓
Engine checks α₃, entropy, hops
        ↓
PASS → Pull request opened
        ↓
CI runs full test suite (5,300 tests)
        ↓
ALL GREEN → Auto-merge
ANY RED → Reject with report
```

There is no human code review in the traditional sense. The review is mathematical. If the crate passes the induction engine and the test suite, it belongs. If not, it doesn't.

### The Veto Power

Anyone can veto a proposed crate by demonstrating that it violates a conservation law:

```python
# Veto example: "This crate leaks correctness"
report = openmind.audit("./proposed-crate")
assert report.entropy >= 2.0  # Fails → valid veto
```

A veto isn't a vote against. It's a proof that the conservation laws are broken. Vetoes are rare because the induction engine catches most problems automatically.

### Decision Making for Non-Technical Issues

Not everything is provable. For roadmap priorities, documentation style, and community norms:

**Ternary consensus among maintainers:**
- Each maintainer casts a trit: {-1: against, 0: abstain, +1: for}
- `tdecide(votes)` resolves in <1ms
- No meetings. No Slack threads. No bikeshedding.

This only applies to decisions that can't be automated. Technical decisions are always automated.

### The No-Fork Principle

The ecosystem resists forks because:
- A fork breaks the three-hop rule
- A fork can't participate in fleet-wide consensus
- A fork with α₃ < 0.97 relative to main is "massive" — an island

If you disagree with the fleet's direction, you don't fork. You write a crate that proves your alternative and submit it. If the induction engine accepts it, your approach becomes part of the fleet.

### Community Roles

| Role | Responsibility | How Earned |
|------|---------------|------------|
| Researcher | Proposes new crates | Write code that passes induction |
| Maintainer | Runs infrastructure | Keep tests green for 6+ months |
| Witness | Validates proofs | Generate witnesses for 10+ compilation stages |
| Conductor | Orchestrates deployments | Deploy 3+ production fleets |

There are no elections. Roles are earned by demonstrated contribution to the conservation laws.

### Conflict Resolution

When two researchers disagree:

1. **Formalize the disagreement as a hypothesis.** Both write minimal crates proving their approach.
2. **Run the induction engine.** Which approach has higher α₃? Higher entropy?
3. **Benchmark.** Which approach is faster? More correct? Uses less attention?
4. **The math decides.** Not popularity. Not seniority. The conservation laws.

If both approaches pass all checks, both enter the fleet. The ecosystem doesn't need monoculture.

## Connect

- [DESIGN-PRINCIPLES.md](DESIGN-PRINCIPLES.md) — The seven invariant rules that govern all decisions
- [RESEARCH-METHODOLOGY.md](RESEARCH-METHODOLOGY.md) — How hypotheses become fleet members
- [VERSIONING-AND-COMPATIBILITY.md](VERSIONING-AND-COMPATIBILITY.md) — Technical governance of crate updates
- [THE-ROADMAP.md](THE-ROADMAP.md) — Where the community is focusing effort
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — The constitution that no one can override

## Activate

Participate in governance:

```bash
# Run the induction engine on your crate
openmind induct ./ternary-youridea

# If it passes, submit
gh pr create --title "feat: ternary-youridea" --body "Induction report attached"

# Monitor the test suite
cargo test --workspace
# All green = eligible for merge
```

The ecosystem doesn't need your permission to grow. It needs your proof.
