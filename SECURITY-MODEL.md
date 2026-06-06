# SECURITY MODEL — Ternary Permissions and Threat Topology

## Hook

> A binary permission system asks: "allowed or denied?" A ternary system asks: "allowed, denied, or deferred?" That third state is where security lives — in the pause before commitment.

## Reveal

Security in SuperInstance isn't an access control list. It's a **ternary permission topology** where every access decision is a vote, every identity is a signal, and every resource has a consensus threshold.

### The Ternary Permission Model

| Permission | Value | Meaning | Action |
|------------|-------|---------|--------|
| Grant | +1 | Explicitly allowed | Execute |
| Deny | -1 | Explicitly forbidden | Block |
| Abstain | 0 | No opinion / insufficient data | Defer to higher authority |

Binary systems collapse grant and abstain into "allowed." This is dangerous: a missing permission rule becomes implicit allow. In ternary, a missing rule is explicit abstain, which triggers escalation.

### Identity as Ternary Signal

An identity isn't a username or a key. It's a **ternary capability vector**:

```json
{
  "identity": "agent-sensor-node-7",
  "capabilities": {
    "read_temperature": +1,
    "set_fan_speed": -1,
    "firmware_update": 0
  }
}
```

- `+1`: This agent CAN perform this action
- `-1`: This agent CANNOT perform this action
- `0`: This agent's permission is undetermined — requires validation

The capability vector is signed by a trusted authority (`ternary-proof` crate) and verified before every flex.

### Access Control as Consensus

When an agent requests access to a resource, the system doesn't check one ACL. It checks a **quorum of validators**:

```python
def check_access(identity, resource, action):
    votes = []
    
    # Vote 1: Does the identity have the capability?
    votes.append(identity.capability(action))
    
    # Vote 2: Is the resource in a safe state?
    votes.append(resource.safety_status())
    
    # Vote 3: Is the action within policy?
    votes.append(policy.check(action, context))
    
    # Consensus: all +1 = grant, any -1 = deny, any 0 = defer
    return tdecide(votes)
```

This is `ternary-semaphore` (see FLEET-MAP.md). Access control is consensus. No single point of failure. No superuser account that can bypass everything.

### The Threat Model

Ternary security defends against three classes of attacker:

| Attacker | Goal | Ternary Defense |
|----------|------|-----------------|
| Eavesdropper | Read ternary signals | Signals are meaningless without shared muscle memory |
| Forger | Inject false votes | Every vote is signed; `tverify` checks signatures in Z₃ |
| Saboteur | Crash the consensus | Missing votes default to {0}; system pauses, doesn't break |

**Eavesdropper:** Intercepting a ternary signal (e.g., `+1` on MQTT) reveals nothing. Without the shared score, the attacker doesn't know what `+1` means or which chord it refers to. The muscle memory file is the decryption key.

**Forger:** Forging a vote requires forging a Z₃ signature. `ternary-zkp` (see FLEET-MAP.md) provides zero-knowledge proof kernels: an agent can prove it has a capability without revealing the capability itself.

**Saboteur:** Crashing N agents reduces the quorum size but doesn't halt the system. If the consensus requires 5 votes and 2 agents crash, the remaining 3 can still reach consensus. The system degrades gracefully (see FAULT-TOLERANCE.md).

### The Deferred State as Security

{0} isn't just "no opinion." It's a **security control**. When any validator returns {0}, the action is deferred:

```python
if decision == Trit.Z0:
    # Escalate to human review
    notify_admin(request)
    # Or escalate to stronger proof
    require_additional_proof(request)
    # Or simply wait
    queue_for_retry(request)
```

This means the system defaults to safe in ambiguity. A new agent with no history? {0} → deferred. A resource in an unknown state? {0} → deferred. An action never seen before? {0} → deferred.

Binary systems default to allow or deny. Ternary systems default to "think harder."

### Key Rotation in Z₃

Cryptographic keys in SuperInstance use ternary representations:

```rust
// A ternary key is a vector of trits
let key: Vec<Trit> = generate_ternary_key(256);  // 256 trits = 512 bits

// Signing: tdot(message_hash, key) mod 3
let signature = tdot(&message_hash, &key);
// signature ∈ {-1, 0, +1}
```

This isn't production cryptography. It's a demonstration that ternary structures can support security primitives. For real deployments, `ternary-zkp` wraps industry-standard primitives (SHA-3, ed25519) with ternary interfaces.

## Connect

- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — Z₃ arithmetic underpins ternary signatures
- [FLEET-MAP.md](FLEET-MAP.md) — Pattern 7 crates: `ternary-proof`, `ternary-zkp`, `ternary-semaphore`
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — Missing votes default to {0}; system stays safe
- [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md) — Shared muscle memory is the encryption key
- [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md) — Security invariants are proven by property tests

## Activate

Implement ternary access control:

```python
import openmind

# Define a capability vector
identity = openmind.Identity("agent-7")
identity.grant("read_temperature")
identity.deny("firmware_update")
# "write_config" is not granted → defaults to abstain (0)

# Check access
resource = openmind.Resource("thermostat")
decision = resource.check_access(identity, "read_temperature")
# Returns: +1 (granted)

decision = resource.check_access(identity, "firmware_update")
# Returns: -1 (denied)

decision = resource.check_access(identity, "write_config")
# Returns: 0 (deferred → requires admin approval)
```

Audit your permission model:
```python
# List all abstain permissions — these are your security gaps
for identity in fleet.identities:
    for cap, vote in identity.capabilities.items():
        if vote == 0:
            print(f"GAP: {identity.name} has no rule for {cap}")
```

A secure ternary system has no implicit permissions. Every capability is explicit: +1, -1, or intentionally left at 0.
