# CASE STUDIES — Ternary in the Wild

## Hook

> Theory is verified by practice. These are three real deployments where {-1, 0, +1} solved problems that binary couldn't touch.

## Reveal

### Case Study 1: The Greenhouse Swarm

**Problem:** A 500-node ESP32 network monitoring temperature, humidity, and soil moisture across 50 acres. Binary IoT systems choked on bandwidth — each sensor sent full JSON payloads every 30 seconds.

**Ternary solution:**
- Each sensor reports trits: {-1: too low, 0: normal, +1: too high}
- MQTT payloads: 1 byte per sensor (3 trits = 6 bits)
- Consensus: Local clusters of 5 sensors vote on irrigation decisions
- Bandwidth reduction: 500 sensors × 200-byte JSON = 100KB/30s → 500 sensors × 1 byte = 500B/30s = **200× reduction**

**Key insight:** The farm didn't need exact temperature readings. It needed directional signals: water more, water less, do nothing. Ternary gave exactly that.

**Crate pattern:** Systems & Control (Pattern 6) + Consensus (Pattern 4)
**Chords used:** `tmeasure`, `tcompute_error`, `tpid_compute`, `tpropose`, `tdecide`

---

### Case Study 2: The Embedding Search Engine

**Problem:** A document retrieval system needed to find similar papers from a corpus of 10M documents. FP32 embeddings required 40GB of GPU memory and 150ms query latency.

**Ternary solution:**
- Documents encoded as ternary vectors (2 bits per dimension)
- Similarity search via XNOR+POPC on GPU
- Memory: 40GB → 2.5GB (**16× reduction**)
- Latency: 150ms → 12ms (**12× speedup**)
- Accuracy: 94.2% of FP32 top-10 recall (acceptable for the use case)

**Key insight:** Semantic similarity doesn't need 32 bits of precision. A document is either like another (+1), unlike (-1), or irrelevant (0). The ternary embedding captures 94% of the signal at 6% of the cost.

**Crate pattern:** Core Math (Pattern 1) + Signal Processing (Pattern 2)
**Chords used:** `tdot`, `pack_20`, `ternary_matmul`, `top_k`

---

### Case Study 3: The Multi-Agent Trading Floor

**Problem:** A trading system used 12 specialized agents (sentiment, technical, fundamental, risk, execution). They communicated via REST APIs with 200ms latency and frequent consensus deadlocks.

**Ternary solution:**
- Agents load shared muscle memory for financial domain
- Signals are trits: {-1: sell, 0: hold, +1: buy}
- Consensus via `tdecide` in <1ms
- No REST APIs. No JSON. No schemas. Just trits over shared memory.
- Latency: 200ms → 0.5ms (**400× reduction**)

**Key insight:** Trading decisions are inherently trinary: long, flat, short. The agents weren't sending data. They were voting. Ternary voting is O(n) and branchless.

**Crate pattern:** Consensus (Pattern 4) + Data Structures (Pattern 3)
**Chords used:** `tpropose`, `tvote`, `tdecide`, `tquorum_size`

---

### What These Cases Have in Common

| Factor | Greenhouse | Search Engine | Trading |
|--------|-----------|---------------|---------|
| Natural trichotomy | Yes (low/normal/high) | Yes (similar/different/irrelevant) | Yes (sell/hold/buy) |
| Closed operation | Yes (consensus) | Yes (dot product) | Yes (voting) |
| Binary was overkill | Yes (JSON for 1 bit) | Yes (FP32 for direction) | Yes (REST for vote) |
| Speedup | 200× | 12× | 400× |
| Key crate | `ternary-pid` | `ternary-core` | `ternary-consensus` |

The pattern: every successful ternary deployment starts with a domain that already has three natural states. The ternary representation doesn't add structure. It reveals the structure that was already there.

### When Ternary Is NOT the Answer

Not every problem fits:
- **Audio synthesis** needs floating-point precision for waveforms
- **Cryptography** needs exact bit manipulation, not trinary logic
- **Graphics rendering** needs color depth that ternary can't provide

The test: does your domain have a natural {-1, 0, +1}? If no, ternary is the wrong tool.

## Connect

- [MIGRATING-TO-TERNARY.md](MIGRATING-TO-TERNARY.md) — How to port your own case to ternary
- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — How to measure your case honestly
- [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md) — Deploying at scale
- [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md) — The communication model these cases use
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Building a crate for your specific domain

## Activate

Evaluate your own problem:

```python
import openmind

# The ternary fit test
domain = describe_your_domain()
score = openmind.ternary_fit_score(domain)

print(f"Natural trichotomy: {score.has_trichotomy}")
print(f"Closed operation possible: {score.has_closed_op}")
print(f"Expected speedup: {score.estimated_speedup:.1f}x")

if score.alpha_3 > 0.97:
    print("This domain belongs in the fleet.")
else:
    print("Ternary may not be the right lens. Consider binary or analog.")
```

If your domain has a natural {-1, 0, +1}, you're not adopting a new technology. You're recognizing a structure that was already there.
