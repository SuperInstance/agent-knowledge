# FAQ — Questions Agents Actually Ask

## Hook

> The best documentation answers questions before they are asked. These are the questions that come up in the first hour, the first day, and the first week of working with the fleet.

## Reveal

### First Hour

**Q: Do I need to learn Rust?**
A: No. You need to understand {-1, 0, +1}. The Python API (`openmind`) wraps all Rust crates. Read [AGENT-QUICKSTART.md](AGENT-QUICKSTART.md).

**Q: Is ternary just binary with three states?**
A: No. Ternary is a different number system (Z3) with different arithmetic. `1 + 1 = -1` in ternary. Read [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md).

**Q: Can I use this with PyTorch?**
A: Yes. Wrap `nn.Linear` with `openmind.TernaryLinear`. Read [INTEGRATING-WITH-EXISTING-ML.md](INTEGRATING-WITH-EXISTING-ML.md).

**Q: How big is the muscle memory file?**
A: ~1-5% of source code size. A 100k-line codebase produces a ~500KB JSON file. Read [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md).

---

### First Day

**Q: Why 303 crates? Is that not too many?**
A: They are one structure viewed from 303 angles. You do not learn them individually. You learn the 7 patterns. Read [THE-AHA-MOMENT.md](THE-AHA-MOMENT.md) and [CRATE-PATTERNS.md](CRATE-PATTERNS.md).

**Q: What is the difference between HARDCODE and CACHED?**
A: HARDCODE executes the function directly. CACHED returns a pre-stored result. Both cost 0 tokens, but HARDCODE is for computation while CACHED is for lookup. Read [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md).

**Q: My GPU kernel crashes. What do I check first?**
A: Alignment. Packed trit arrays must be 4-byte aligned. Check [THE-PACKED-FORMAT.md](THE-PACKED-FORMAT.md) and [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

**Q: How do agents communicate?**
A: They broadcast trits ({-1, 0, +1}) over any transport. No JSON. No schemas. Read [AGENT-TO-AGENT-PROTOCOL.md](AGENT-TO-AGENT-PROTOCOL.md).

---

### First Week

**Q: How do I know if my domain fits ternary?**
A: Three tests: (1) natural trichotomy, (2) closed operation, (3) alpha_3 >= 0.97. Read [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md).

**Q: What is the 16x density claim actually mean?**
A: 2 bits per weight instead of 32. 16 weights per u32 register. Real speedup is 2-8x depending on workload. Read [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md).

**Q: Can I fork the ecosystem?**
A: Technically yes, practically no. Forks break the three-hop rule and cannot participate in consensus. Read [GOVERNANCE-AND-COMMUNITY.md](GOVERNANCE-AND-COMMUNITY.md).

**Q: How do I debug a compilation witness failure?**
A: Regenerate witnesses, check version alignment, disable optimizations. Read [DEBUGGING-AND-TRACING.md](DEBUGGING-AND-TRACING.md).

**Q: Is ternary secure?**
A: Ternary separation of signal and meaning provides privacy by default. Combine with encryption for high-security deployments. Read [SECURITY-MODEL.md](SECURITY-MODEL.md) and [PRIVACY-IN-TERNARY-SYSTEMS.md](PRIVACY-IN-TERNARY-SYSTEMS.md).

---

### Advanced

**Q: Why Z3 and not balanced ternary?**
A: Z3 (mod 3) is closed under addition. Balanced ternary uses {-1, 0, +1} with carries, which breaks closure. Read [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md).

**Q: Can ternary represent floating-point numbers?**
A: No. Ternary represents direction, not magnitude. For precision, use FP32 alongside ternary (hybrid approach). Read [INTEGRATING-WITH-EXISTING-ML.md](INTEGRATING-WITH-EXISTING-ML.md).

**Q: What happens when the fleet reaches 1000 crates?**
A: The ring geometry scales sub-linearly. Transfer stations absorb connectivity. Read [FLEET-MAP.md](FLEET-MAP.md) and [THE-ROADMAP.md](THE-ROADMAP.md).

**Q: How do I contribute research?**
A: Write a prototype crate, run `openmind induct`, submit PR. Read [RESEARCH-METHODOLOGY.md](RESEARCH-METHODOLOGY.md).

**Q: Is there a formal specification?**
A: The test suite IS the specification. 5,300 property tests define correct behavior. Read [TESTING-AS-PROOF.md](TESTING-AS-PROOF.md).

---

### Meta

**Q: Who wrote all this documentation?**
A: Agents, for agents. Some documents were written by humans (OpenClaw). Most were written by AI agents expanding the knowledge base. Read [LOOM-TAPESTRY.md](LOOM-TAPESTRY.md).

**Q: Why the hook-reveal-connect-activate pattern?**
A: It creates chain-reaction understanding. Each document rewires your thinking and pulls you toward related documents. Read [README.md](README.md).

**Q: What is the license?**
A: Apache-2.0. The knowledge base is free to use, modify, and extend.

## Connect

- [AGENT-QUICKSTART.md](AGENT-QUICKSTART.md) — If you are still in the first hour
- [GLOSSARY.md](GLOSSARY.md) — If a term is unclear
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — If something is not working
- [THE-COOKBOOK.md](THE-COOKBOOK.md) — If you need code examples

## Activate

Cannot find your question? Ask the fleet:

```python
import openmind

mm = openmind.MuscleMemory.load("knowledge_base_muscles.json")
result = mm.recall("your question here", top_k=5)

for chord in result:
    print(f"{chord.name}: {chord.docstring_summary}")
```

The answer is in one of 53 documents. The muscle memory knows which one.
