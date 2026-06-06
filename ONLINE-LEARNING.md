# Online Learning

> An agent that stops learning when deployed is already obsolete.

## HOOK

Online learning is not periodic retraining. It is the continuous adjustment of an agent's behavior from the stream of its own experience, without stopping the world to collect a new dataset.

## REVEAL

Traditional machine learning operates in two phases: train on a static dataset, then deploy a frozen model. This batch paradigm works for stable distributions but fails when the world changes, when users personalize, and when the agent must discover capabilities that were not in the training data.

Online learning unifies training and deployment into a single loop:

```
perceive → act → observe outcome → update → repeat
```

The update must be:

- **Fast**: Experience should improve behavior before the experience becomes irrelevant.
- **Safe**: The agent must not catastrophically forget previous knowledge.
- **Testable**: The update should be provably beneficial or at least bounded in harm.
- **Efficient**: Updates should run on the same hardware that runs inference.

### The Update Spectrum

Online updates vary in depth:

1. **Context adaptation**: Adjust attention weights, few-shot examples, or prompt framing. Latency: milliseconds. No persistent change.
2. **Memory insertion**: Add new facts, constructs, or muscle memories to the registry. Latency: seconds. Persistent but local.
3. **Local fine-tuning**: Run a small number of gradient steps on recent experience. Latency: minutes. Changes model weights.
4. **Construct generation**: Compile a new skill from experience and validate it in the sandbox. Latency: minutes to hours. Changes agent capabilities.

SuperInstance agents use all four, selecting the depth based on confidence, stakes, and compute budget.

### Catastrophic Forgetting

The central challenge in online learning is catastrophic forgetting: learning task B degrades performance on task A. The ecosystem addresses this through:

- **Modular constructs**: New knowledge is added as separate constructs rather than overwriting shared weights.
- **Elastic weight consolidation**: Important parameters are protected by a penalty term derived from the Fisher information matrix.
- **Replay buffers**: A small reservoir of past experiences is mixed with new experience during updates.
- **Ternary regularization**: Ternary weights have fewer degrees of freedom than real weights, which naturally constrains the update magnitude.

### Exploration vs. Exploitation

Online learning requires exploration: sometimes the agent must try unfamiliar actions to discover better policies. The ternary framework maps exploration to a `0` confidence state:

- `+1` confidence → exploit known best action.
- `0` confidence → explore actions with uncertain outcomes.
- `-1` confidence → avoid actions known to be harmful.

This mapping lets the agent use the same circuit-breaker and canary infrastructure for safe exploration. Uncertain actions are deployed as canaries; harmful actions are tripped open.

### Verification

Not all online updates are beneficial. The ecosystem requires that updates be verified before they become permanent:

- **Self-test**: The agent runs the updated policy on held-out tasks.
- **Sandbox validation**: New constructs are compiled and tested in `oxide-sandbox`.
- **Canary rollout**: Updated skills are exposed to 5% of traffic before full deployment.
- **Rollback**: If metrics degrade, the agent reverts to the previous construct version.

## Experiments

1. **Few-shot adaptation**: Show an agent 10 examples of a new classification task and measure accuracy before and after context adaptation. Expected: >70% accuracy with minimal latency cost.
2. **Forgetting curve**: Train an agent on task A, then stream task B for 1000 steps. Measure task A accuracy at intervals. Expected: modular constructs retain >90% of task A performance; naive fine-tuning drops below 50%.
3. **Exploration efficiency**: Compare epsilon-greedy exploration with ternary-confidence exploration on a bandit task. Expected: ternary exploration converges faster by avoiding known-bad actions.
4. **Construct generation from failure**: When an agent fails at a task, generate a new construct from the successful recovery and deploy it. Measure reduction in future failure rate. Expected: 30-60% reduction for repeated task types.

## Applications

- **Personalized assistants**: Learn user preferences from interaction without explicit training sessions.
- **Adaptive robotics**: Robots that improve manipulation policies from every grasp attempt.
- **Self-healing systems**: Agents that learn to route around newly discovered failure modes.
- **Continuous fraud detection**: Update detection models as adversaries evolve their strategies.
- **Scientific discovery**: Agents that refine experimental policies based on observed outcomes.

## Open Questions

1. **Update frequency**: How often should an agent update? Too frequent causes instability; too infrequent causes stagnation.
2. **Credit assignment**: In long-horizon tasks, how does the agent know which past action caused a distant outcome?
3. **Adversarial updates**: Can a malicious user craft experiences that corrupt the agent's learned behavior?
4. **Legal and ethical boundaries**: What must an agent forget, and who decides?

## CONNECT

- [THE-AGENT-LOOP.md](THE-AGENT-LOOP.md) — The feedback loop that drives online learning.
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How learned behaviors are cached as constructs.
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Why every online update must be testable.
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — How breakers and canaries make exploration safe.
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — What knowledge must be preserved during updates.
- `oxide-sandbox` — Validates constructs generated from online experience.
- `oxide-canary` — Safely rolls out learned updates.

## ACTIVATE

Implement one online learning rule in an agent you control. Choose the simplest rule: when the agent makes a mistake, store the corrected example in a replay buffer, and every hour run 10 gradient steps on a batch mixed from the buffer and recent experience. Run the agent for one day. Measure whether accuracy on recent tasks improves and whether accuracy on older tasks degrades. If degradation occurs, add a simple regularization term that penalizes changes to parameters with high historical importance. Log the trade-off curve.
