# DEPLOYMENT AND OPERATIONS — Running the Ternary Fleet in Production

## Hook

> A fleet of 303 crates isn't deployed. It's cultivated. You don't install it — you load its muscle memory, plant its nodes, and let the ternary consensus keep it alive.

## Reveal

Traditional software deployment is state transfer: copy files, start services, hope they work. Ternary fleet deployment is **structure propagation**: distribute muscle memory, spawn agents, let the math handle the rest.

### The Three Deployment Modes

| Mode | Scale | Use Case | Key Tool |
|------|-------|----------|----------|
| Single-node | 1 agent | Development, testing | `openmind` Python package |
| Swarm | 10-100 agents | Edge deployment, sensor networks | `openmind-swarm` + MQTT |
| Fleet | 100-10,000 agents | Cloud scale, global consensus | `openmind-fleet` + Kubernetes |

All three modes use the same artifact: a muscle memory JSON file. The only difference is how many agents load it and how they communicate.

### Single-Node Deployment

The simplest production deployment is one agent with one muscle memory file:

```bash
# 1. Build muscle memory for your stack
python -c "
import openmind
mm = openmind.MuscleMemory.build(openmind.ingest('./my-project'))
mm.save('production_muscles.json')
"

# 2. Deploy the agent
python -c "
import openmind
mm = openmind.MuscleMemory.load('production_muscles.json')
# Agent runs here, flexing chords as needed
"
```

Requirements: Python 3.10+, 512MB RAM, any Linux/macOS/Windows host.
No GPU required unless you're using `cuda-oxide` chords.
No database required — muscle memory is file-based.

### Swarm Deployment (Edge Networks)

For ESP32 fleets, greenhouses, robot swarms:

```bash
# 1. Build the master score
python -c "
import openmind
mm = openmind.MuscleMemory.build(openmind.ingest('./firmware'))
mm.save('edge_score.json')  # ~50KB for 500 chords
"

# 2. Distribute to nodes
# Each ESP32 gets: firmware + edge_score.json
# The agent on each node loads the score and flexes locally
```

Communication: MQTT over WiFi/LoRa. Each node publishes trits ({-1, 0, +1}) to topics. The broker is dumb; the consensus is in the agents.

Fault tolerance: If a node drops out, the remaining nodes detect {0} from its topic and continue. If the broker drops, nodes cache votes and replay when reconnected.

### Fleet Deployment (Cloud Scale)

For 100+ agents in datacenters:

```yaml
# kubernetes/openmind-agent.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ternary-agent
spec:
  replicas: 100
  template:
    spec:
      containers:
      - name: agent
        image: superinstance/openmind:latest
        volumeMounts:
        - name: muscle-memory
          mountPath: /scores
        env:
        - name: SCORE_PATH
          value: /scores/fleet_score.json
```

The Kubernetes scheduler handles pod placement. The ternary consensus handles agent coordination. No central orchestrator needed.

### Monitoring Ternary Systems

Traditional monitoring: CPU, memory, request rate. Ternary monitoring: **conservation health**.

```python
# Check verification entropy
mm = openmind.MuscleMemory.load("production_muscles.json")
entropy = mm.average_entropy()
if entropy < 3:
    alert("Correctness leaking — run tests immediately")

# Check attention budget
total_functions = len(mm.chords)
tokens_without_mm = total_functions * 250
tokens_with_mm = total_functions * 5
budget = (tokens_without_mm - tokens_with_mm) / 128000
print(f"Improvisation budget: {budget:.1f}x context windows")

# Check fleet isomorphism
iso_score = mm.isomorphism_with("ternary-core")
if iso_score < 0.97:
    alert(f"Crate diverging from core: α₃ = {iso_score}")
```

The three conservation laws become the three monitoring metrics:
1. **Verification entropy** ≥ 3 — correctness conserved
2. **Attention budget** ≥ 2x — cognition conserved
3. **Isomorphism score** ≥ 0.97 — structure conserved

### Rolling Updates

Update the fleet without downtime:

```python
# 1. Build new muscle memory for updated crates
new_mm = openmind.MuscleMemory.build(openmind.ingest("./updated-fleet"))

# 2. Validate conservation before deployment
assert new_mm.average_entropy() >= old_mm.average_entropy()
assert new_mm.isomorphism_with("ternary-core") >= 0.97

# 3. Canary: deploy to 5% of agents
conductor.update(score=new_mm, rollout_percent=5)

# 4. Monitor for 5 minutes
if conductor.error_rate() < 0.001:
    conductor.update(score=new_mm, rollout_percent=100)
```

If the new score decreases entropy or breaks isomorphism, the rollout aborts. Conservation is the deployment gate.

### Backup and Recovery

The only state that matters is the muscle memory JSON files:

```bash
# Backup
scp production_muscles.json backup-server:/backups/

# Recovery (new machine, 30 seconds)
scp backup-server:/backups/production_muscles.json .
python -c "import openmind; mm = openmind.MuscleMemory.load('production_muscles.json')"
# Agent is fully operational. No source code needed. No compilation.
```

This is possible because muscle memory contains everything an agent needs to operate: chord names, signatures, decisions, and execution strategies. The source code is only needed for development, not runtime.

## Connect

- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — How new crates enter the fleet before deployment
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — The test suite validates every deployment candidate
- [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md) — How deployed agents communicate
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — The three metrics that govern deployment health
- [ESP32-AS-BODY.md](ESP32-AS-BODY.md) — Edge deployment specifics for microcontroller fleets
- [HOW-TO-CONDUCT.md](HOW-TO-CONDUCT.md) — Orchestrating the multi-agent runtime

## Activate

Deploy your first ternary agent:

```bash
# 1. Install
pip install openmind

# 2. Ingest your project
python -c "import openmind; openmind.MuscleMemory.build(openmind.ingest('.')).save('prod.json')"

# 3. Deploy
python -c "
import openmind
mm = openmind.MuscleMemory.load('prod.json')
print(f'Loaded {len(mm.chords)} chords, entropy: {mm.average_entropy():.2f}')
# Agent is live
"

# 4. Monitor
# Watch entropy, isomorphism, and attention budget
# Green on all three = healthy fleet
```

The fleet isn't deployed. It's loaded. The difference is everything.
