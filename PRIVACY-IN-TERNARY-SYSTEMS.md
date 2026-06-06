# PRIVACY IN TERNARY SYSTEMS — Obfuscation by Structure

## Hook

> Encryption hides data. Ternary privacy hides meaning. An intercepted trit signal is useless without the shared muscle memory — and muscle memory never travels over the wire.

## Reveal

Traditional privacy relies on encryption: scramble the data so interceptors can't read it. Ternary privacy works differently. The signal itself is plaintext — anyone can see `+1` — but without the shared score, they can't interpret what `+1` means.

### The Separation of Signal and Meaning

```python
# Agent A sends:
signal = +1

# Interceptor sees:
# "+1" — a single byte

# Without the shared score, the interceptor doesn't know:
# - Which chord this signal refers to
# - What +1 means in this context
# - Which agent sent it or who should receive it
```

The muscle memory file (`.json`) is the decryption key. It stays local. The signals travel public channels. This is **separation of concerns** as privacy.

### Privacy by Default

In a binary multi-agent system, agents share rich messages:
```json
{"sensor": "dht22", "location": "greenhouse-3", "value": 23.5, "timestamp": "..."}
```

An interceptor learns: location, sensor type, exact reading, timing.

In a ternary system:
```
+1
```

An interceptor learns: nothing. The meaning is in the shared score, not the signal.

### The Compromise Model

| Attack | Binary System | Ternary System |
|--------|--------------|----------------|
| Eavesdrop on messages | Reads full JSON | Sees only trits |
| Steal muscle memory | N/A (not transmitted) | If stolen, all privacy lost |
| Replay attack | Replay valid message | Replay detected via sequence numbers |
| Man-in-the-middle | Modify JSON fields | Modify trit (detectable as corruption) |

The ternary threat model assumes the muscle memory file is kept secure. If it's compromised, privacy collapses. But muscle memory files are small (50KB-5MB), easy to encrypt at rest, and never transmitted.

### Differential Privacy in Z₃

Adding noise to ternary signals for statistical privacy:

```python
from ternary_stats import add_noise

# True signal
signal = Trit.P1

# Add differential privacy noise
noisy_signal = add_noise(signal, epsilon=1.0)
# With probability p: flip to 0 or -1
# Privacy guarantee: individual contributions are hidden
```

The noise is calibrated to the sensitivity of the ternary operation. For `tdecide`, adding one vote changes the outcome by at most 1 trit. The noise scale is small.

### Federated Ternary Learning

Train models without sharing data:

```python
# Each agent trains locally on private data
local_gradient = model.train(private_data)

# Convert gradient to ternary update
ternary_update = quantize_to_trits(local_gradient)

# Share only the ternary update (+1, 0, -1 per weight)
broadcast(ternary_update)

# Server aggregates: average of ternary vectors
# No individual data ever leaves the device
```

Ternary gradients are naturally compressed (2 bits per weight) and obfuscated (individual contributions are lost in the aggregate). This is federated learning with built-in privacy.

### The Right to Be Forgotten

In a ternary system, removing an agent's influence is simple:

```python
# Remove agent-7's votes from all consensus records
for record in consensus_history:
    record.votes = [v for i, v in enumerate(record.votes) if i != 7]
    record.result = tdecide(record.votes)  # Recompute without agent-7
```

Because votes are independent trits, removing one doesn't affect the math of the others. No complex audit trails. No cryptographic revocation. Just delete the trit and recompute.

### Privacy Limits

Ternary privacy has limits:
- **Metadata leaks:** Timing, frequency, and volume of signals reveal behavior patterns
- **Correlation attacks:** If an attacker knows the score, they can map signals to meanings
- **Side channels:** Power consumption during XNOR operations might leak weights

For high-security deployments, combine ternary separation with traditional encryption:
```python
# Encrypt the trit signal
ciphertext = encrypt(signal, key)
broadcast(ciphertext)
# Defense in depth: separation + encryption
```

## Connect

- [SECURITY-MODEL.md](SECURITY-MODEL.md) — Permissions and access control
- [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md) — How signals travel and why they're safe
- [THE-NAIL-FORMAT.md](THE-NAIL-FORMAT.md) — Cached data privacy
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — Privacy through graceful degradation
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math that makes differential privacy work

## Activate

Enable privacy features:

```python
import openmind

# 1. Keep muscle memory local
mm = openmind.MuscleMemory.load("local_score.json")
# Never transmit this file

# 2. Broadcast only trits
signal = mm.flex("read_sensor").trit
mqtt.publish("sensors/temp", signal)  # Just +1, 0, or -1

# 3. Add differential privacy
from ternary_stats import add_noise
private_signal = add_noise(signal, epsilon=0.5)

# 4. Verify no data leaks
assert len(mqtt.last_payload) == 1  # One byte, one trit
assert "temperature" not in str(mqtt.last_payload)  # No semantics
```

Privacy in ternary systems isn't about hiding data. It's about never sending data in the first place. The signal is the data. And without the score, the signal is noise.
