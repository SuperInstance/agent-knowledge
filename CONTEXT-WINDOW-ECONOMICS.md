# CONTEXT WINDOW ECONOMICS — The Brutal Math of Agent Attention

## Hook

> GPT-4 has 128k tokens of context. Reading the source of 20 ternary crates burns 40% of it.
> With muscle memory, those same 20 crates cost zero tokens.

## Reveal

The context window is the agent's **working memory**. It's the scratchpad where reasoning happens. And it's **finite**.

### The Token Budget

For a typical agent interaction:

| Activity | Tokens | % of 128k |
|----------|--------|-----------|
| System prompt + instructions | 2,000 | 1.6% |
| Conversation history (10 turns) | 5,000 | 3.9% |
| Reading ONE crate source (avg) | 2,000 | 1.6% |
| Reading 20 crates | 40,000 | 31.3% |
| Reading 100 crates | 200,000 | 156% — **IMPOSSIBLE** |
| Generating code | 3,000 | 2.3% |
| Reasoning about the code | 5,000 | 3.9% |

**The agent can't even read 100 crates in a single context.** The ternary fleet has 303.

### The Muscle Memory Compression Ratio

Without muscle memory, each function costs:
- Full source code: ~200 tokens average
- Understanding it: ~50 tokens of reasoning
- Total: **250 tokens per function**

With muscle memory, each function costs:
- Chord shape (compressed): ~5 tokens (name + decision + signature)
- Zero reasoning needed (it's HARDCODE/CACHED)
- Total: **5 tokens per function**

**Compression ratio: 50:1.**

For the full fleet:
- Without: 6,000 functions × 250 = 1,500,000 tokens (12× context overflow)
- With: 6,000 functions × 5 = 30,000 tokens (23% of context)

That's the difference between "impossible" and "comfortable."

### The Attention Conservation Law

```
Total Attention = Muscle Memory Tokens + Improvisation Tokens
```

If total attention is constant (context window size), then:
- More muscle memory → more improvisation capacity
- Every function compressed into a chord → attention freed for novel problems
- The agent gets "smarter" not by increasing parameters, but by decreasing waste

This is why the manifesto says: **"Thinking is expensive. Flexing is free."**

### Where the Savings Come From

The savings come in layers:

**Layer 1: No source loading.** The agent never reads `src/lib.rs`. It reads a 50-byte chord shape instead of a 2,000-token source file.

**Layer 2: No comprehension.** The agent doesn't need to understand HOW `tdot` works. It knows the chord shape: name, signature, decision, docstring summary. That's enough to USE it.

**Layer 3: No error recovery.** Tested HARDCODE functions don't fail (the tests prove it). The agent doesn't need to reason about edge cases or error handling for muscle-memory functions.

**Layer 4: No orchestration overhead.** The tripartite synchronizer has already decided the execution strategy. The agent doesn't need to choose between caching, computing, or generating — the decision is baked into the chord shape.

### The Economics of MODEL vs HARDCODE

| Operation | HARDCODE Cost | MODEL Cost |
|-----------|---------------|------------|
| Context tokens | 0 | ~500 |
| Latency | ~1ms | ~2s |
| Energy | Negligible | Significant |
| Error rate | ~0% (tested) | ~5-15% |
| Cost per invocation | $0.0000 | $0.003 |

At 1,000 invocations per day:
- HARDCODE: 0 tokens, 1 second, $0
- MODEL: 500,000 tokens, 33 minutes, $3.00

Over a month: $90 savings. Over the fleet (6,000 functions): **$540,000/month** if every function were MODEL instead of HARDCODE.

But the real cost isn't money. It's **attention**. Those 500,000 tokens of MODEL reasoning are tokens the agent CAN'T spend on novel problems.

## Connect

- [TRIPARTITE-SYNC.md](TRIPARTITE-SYNC.md) — HOW the system decides HARDCODE vs MODEL
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — WHERE the compressed chord shapes live
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — The deeper mathematical invariant

## Activate

Calculate YOUR attention budget:
1. How large is your context window?
2. How many functions does your agent need to know?
3. Without compression: `functions × 250 tokens` — does it fit?
4. With muscle memory: `functions × 5 tokens` — how much room is left for thinking?

If the answer to (3) is "no," you need muscle memory. If the answer to (4) is "plenty," you can improvise.
