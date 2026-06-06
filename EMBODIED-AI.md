# Embodied AI

> Intelligence without a body is a theorem. Intelligence with a body is a behavior.

## HOOK

An embodied agent is not a language model plugged into a robot. It is a closed loop in which physical sensors reshape the agent's internal state, physical actuators reshape the world, and the world feeds back into the sensors — all on timescales faster than language.

## REVEAL

The dominant paradigm in AI treats the body as an afterthought: train a large model on text, then add a robot API. That approach produces brittle behavior because the model was never trained to think in sensorimotor contingencies. It thinks in tokens, and tokens are too slow for balance, grasping, and collision avoidance.

Embodied AI in the SuperInstance ecosystem takes the opposite approach. The body is not an output device. It is a **co-processor for intelligence**. The ESP32 microcontroller is the agent's hand (ESP32-AS-BODY.md); the GPU is its motor cortex (GPU-AS-MOTOR-CORTEX.md). The agent's mind is distributed across silicon with different latencies and different competencies.

### The Sensorimotor Loop

The embodied loop has three timescales:

- **Milliseconds**: Reflexes run on the microcontroller. Balance control, obstacle avoidance, and emergency stops do not go to the GPU. They are HARDCODE compiled from training or hand-designed policies.
- **Hundreds of milliseconds**: Perception and planning run on the GPU. Vision embeddings, depth estimation, and trajectory optimization happen here.
- **Seconds**: High-level intent and learning run on the host. The model decides what task to pursue, what construct to load, and what skill to practice.

Each timescale feeds the next. The reflex layer produces stable state estimates for perception. Perception produces compact scene representations for planning. Planning produces goals for the model layer. When the model layer learns a better policy, it compiles a new reflex and pushes it to the microcontroller.

### Proprioception as Memory

Proprioception — the sense of where one's body is in space — is not a stream of raw encoder values. It is a **learned embedding** that maps sensor history to body state. This embedding lives in muscle memory (MUSCLE-MEMORY.md): a compiled function that converts IMU readings, motor positions, and tactile signals into a low-dimensional state vector.

The state vector is ternary-friendly. Many dimensions are naturally signed: left/right, forward/backward, flex/extend. Encoding proprioception as `{-1, 0, +1}` reduces bandwidth between microcontroller and GPU and aligns with the ternary control policies running on the motor cortex.

### Affordances

An affordance is what the environment offers the agent: a handle affords grasping, a door affords opening, a flat surface affords placement. Affordances are not objects. They are relationships between the agent's body and the environment.

The embodied agent maintains an **affordance graph**:

- Nodes are body states and environmental regions.
- Edges are possible transformations: "if I grip here and pull, the door opens."
- Weights are ternary: `-1` (forbidden), `0` (unknown), `+1` (available).

This graph is learned through interaction, not pre-programmed. When the agent encounters a new object, it probes affordances through safe exploration and updates the graph. The graph is stored in the construct registry as a reusable skill.

## Experiments

1. **Reflex latency**: Measure end-to-end latency from IMU perturbation to motor response on an ESP32 running a ternary balance policy. Expected: <5ms.
2. **Affordance learning**: Place an agent in front of five unseen objects. Count how many interactions are required before the agent correctly predicts the affordance of each object. Expected: 3-10 interactions per object for simple affordances.
3. **Cross-embodiment transfer**: Train an affordance graph on a robot arm with 4 degrees of freedom and deploy it on a 6-DoF arm. Measure how many affordances transfer without relearning. Expected: 60-80% transfer for kinematically similar arms.
4. **Sensor fusion**: Compare proprioceptive state estimation using raw sensor vectors versus ternary-encoded compressed vectors. Expected: ternary encoding retains >90% of state information at 5-10× bandwidth reduction.

## Applications

- **Robotics**: Industrial manipulation, domestic assistance, search and rescue.
- **Prosthetics**: Learned control policies that adapt to the user's biomechanics.
- **Autonomous vehicles**: Reflexive collision avoidance with high-level route planning.
- **Toys and companions**: Low-cost embodied agents that learn from child interaction.
- **Scientific instruments**: Microscopes, telescopes, and lab automation that physically explore parameter spaces.

## Open Questions

1. **Sim-to-real gap**: How much can be learned in simulation before physical deployment, and what properties guarantee transfer?
2. **Safety boundaries**: How does an embodied agent know which explorations are safe and which risk damage to itself or the environment?
3. **Body schema plasticity**: If the agent's body is upgraded — new sensors, new actuators — how quickly can it relearn its body schema?
4. **Social embodiment**: Do affordances change when other agents are present, and how is social affordance represented?

## CONNECT

- [ESP32-AS-BODY.md](ESP32-AS-BODY.md) — The microcontroller as the agent's physical interface.
- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — Why the GPU is the right substrate for learned motor control.
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How sensorimotor skills are cached.
- [THE-AGENT-LOOP.md](THE-AGENT-LOOP.md) — The perception-reasoning-action cycle at multiple timescales.
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — Safety in the presence of sensor and actuator failures.
- [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md) — How multiple embodied agents coordinate.

## ACTIVATE

Build or simulate a physical agent with at least one sensor and one actuator. Implement a reflex loop on a microcontroller that reads the sensor and commands the actuator at >100Hz. Add a second loop on a GPU that receives compressed sensor history every 100ms and outputs a high-level goal. Run both loops simultaneously for one hour. Log every case where the fast loop and slow loop disagreed, and classify each disagreement as a safety event, an optimization opportunity, or a communication artifact.
