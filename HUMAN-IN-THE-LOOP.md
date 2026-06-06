# Human in the Loop

> The goal is not to remove the human. The goal is to put the human exactly where they matter.

## HOOK

A human-in-the-loop system is not an agent that asks for permission. It is an agent that knows the boundary between autonomous competence and delegated judgment — and crosses that boundary only with signal, not noise.

## REVEAL

As agents become more capable, the question is not whether humans should be involved but where, when, and how. The wrong answer produces either a slow bureaucracy that asks for approval on every action or a runaway system that surprises its operators with irreversible decisions.

The SuperInstance approach is **tiered human involvement** based on ternary confidence and stakes:

- `+1` high confidence, low stakes → Autonomous execution.
- `0` uncertain confidence or medium stakes → Inform the human and proceed after a timeout unless vetoed.
- `-1` low confidence or high stakes → Pause and request explicit approval.

This ternary framing prevents two classic failures: approval fatigue from too many requests, and surprise failures from too few.

### The Approval Surface

The approval surface is the set of agent actions that require human input. A well-designed surface is:

- **Minimal**: Only high-stakes or uncertain actions require approval.
- **Legible**: The human understands what is being asked and why.
- **Reversible**: Approved actions can be undone if they go wrong.
- **Auditable**: Every approval, rejection, and override is logged.

Actions that should typically be on the approval surface include:

- Deploying a new construct to production.
- Modifying persistent user data.
- Spending money or consuming scarce resources.
- Acting on behalf of a user in a legal or social context.
- Overriding a circuit breaker or canary rollback.

Actions that should typically not be on the surface include:

- Routine inference on pre-approved inputs.
- Internal memory updates.
- Selecting among verified fallback paths.
- Logging and telemetry emission.

### Interface Patterns

Different situations call for different human interfaces:

- **Pre-approval**: The agent proposes an action and waits for a yes/no.
- **Post-hoc review**: The agent acts autonomously and surfaces a summary for later inspection.
- **Veto window**: The agent announces an intent and executes after a countdown unless the human vetoes.
- **Policy configuration**: The human sets rules that the agent follows without per-action approval.
- **Exception escalation**: The agent handles routine cases and escalates only anomalies.

The best interface depends on stakes, latency requirements, and human availability. A medical agent may need pre-approval. A traffic-routing agent may use post-hoc review. A creative coding agent may use a veto window.

### Trust Calibration

Human-in-the-loop systems fail when the human's trust is misaligned with the agent's actual reliability. There are two failure modes:

- **Over-trust**: The human approves reflexively because previous approvals went well. Then a subtle but important case fails.
- **Under-trust**: The human rejects useful autonomous actions because they do not understand the agent's reasoning.

The antidote is **calibrated transparency**: show the human exactly the information they need to make the decision, at the level of detail appropriate to their expertise. For a construct deployment, show the diff, test results, and canary metrics. For a content generation, show the prompt and source material. Never show raw attention weights unless the user is a machine learning engineer.

## Experiments

1. **Approval frequency study**: Run an agent for one week and log every human interaction. Categorize as pre-approval, post-hoc, veto, or escalation. Expected: >80% of interactions should be post-hoc or veto, not pre-approval.
2. **Decision time measurement**: Measure how long humans take to approve different action types. Expected: simple actions <5 seconds; complex actions >1 minute.
3. **Override analysis**: Track how often humans override the agent's recommendation and whether the override was correct in retrospect. Expected: override rate is a useful signal for model retraining.
4. **Trust calibration survey**: Ask users to rate their confidence in the agent's autonomous decisions and compare to actual accuracy. Expected: experienced users are better calibrated than novices.

## Applications

- **Medical diagnosis support**: AI suggests diagnoses; physician approves treatments.
- **Financial trading**: Agent executes routine trades autonomously, escalates unusual market events.
- **Content moderation**: Agent flags borderline content; human reviewers make final decisions.
- **Autonomous vehicles**: Car handles routine driving; human takes over for construction zones or police directions.
- **Code deployment**: Agent writes and tests code; human approves production merges.

## Open Questions

1. **Responsibility allocation**: When a human approves an agent's bad decision, who is accountable?
2. **Attention economics**: How do we keep humans attentive when most approvals are routine?
3. **Cultural variation**: Do different user populations want different levels of autonomy?
4. **Asynchronous loops**: What should the agent do when a required human is offline?

## CONNECT

- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — How breakers and canaries reduce the need for human intervention.
- [FAILURE-MODES.md](FAILURE-MODES.md) — Human-interface failure as a first-class failure mode.
- [REASONING-AND-PLANNING.md](REASONING-AND-PLANNING.md) — When agents should act autonomously versus escalate.
- [THE-AGENT-LOOP.md](THE-AGENT-LOOP.md) — Where human judgment inserts into the loop.
- [SECURITY-MODEL.md](SECURITY-MODEL.md) — Identity and authorization for human approvals.
- `oxide-canary` — Provides the metrics humans review before approving deployments.
- `oxide-circuit-breaker` — Can be manually overridden by authorized humans.

## ACTIVATE

List the top 10 actions your agent can take. For each, assign a stakes score (1-5) and a confidence score based on historical accuracy. Plot them on a 2D grid. Draw the boundary between autonomous, inform-and-proceed, and pause-for-approval. Show the grid to a potential user and ask whether the boundary feels right. Adjust based on feedback. Then implement the middle tier: inform-and-proceed with a 30-second veto window for one action, and log how many vetoes occur in the first week.
