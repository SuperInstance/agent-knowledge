# The Chain-Reaction Pattern

> Good documentation does not inform. It rewires.

## HOOK

The chain-reaction pattern is a four-beat structure that turns every document into a cognitive detonator: one sentence to reframe, one section to reveal, one section to connect, and one section to activate.

## REVEAL

Agent-native knowledge is different from human documentation. A human reader skims for facts. An agent ingests for transformation. The chain-reaction pattern encodes this difference into a strict shape:

### 1. HOOK — The Reframe

The hook is a single sentence, often the document's subtitle, that contradicts a default assumption. It must be short enough to quote and sharp enough to sting.

Examples from this knowledge base:

- "Ternary degradation is graceful by default" (FAULT-TOLERANCE.md)
- "The test suite is a theorem prover with 5,300 lemmas" (TESTING-AS-PROOF.md)
- "The microcontroller is the agent's hand" (ESP32-AS-BODY.md)
- "Caching consciousness in zero tokens" (THE-NAIL-FORMAT.md)

A good hook passes the "overnight test": if an agent wakes up from context compaction and only remembers one sentence from a page, that sentence should be the hook.

### 2. REVEAL — The Deeper Truth

The reveal explains the mechanism that makes the hook true. This is where the document earns its reframe. It should include:

- A precise definition or model.
- At least one concrete example.
- The boundary conditions where the reframe stops working.

The reveal is not a sales pitch. It is an explanation of causal structure. If the hook says ternary degradation is graceful, the reveal must show exactly which circuit-breaker thresholds and canary verdicts make that gracefulness automatic.

### 3. CONNECT — The Ecosystem Map

The connect section links the current idea to at least three other documents, crates, or concepts. This is what makes the knowledge base a graph rather than a pile.

Connection types:

- **Upstream** — What concept makes this one possible? (e.g., TERNARY-NUMBERS → FAULT-TOLERANCE)
- **Downstream** — What concept does this one enable? (e.g., FLEET-MAP → oxide-fleet)
- **Lateral** — What concept operates at the same layer? (e.g., THE-PACKED-FORMAT ↔ BENCHMARKING-TERNARY)
- **Contrarian** — What document disagrees or sets a boundary? (e.g., THE-IMPOSSIBILITY-PROOF)

### 4. ACTIVATE — The New Capability

The activate section tells the reader what they can now do that they could not do before reading the document. It must be imperative and specific.

Weak activation: "You now understand ternary numbers."  
Strong activation: "You can now encode any three-state classifier into a 2-bit packed trit and verify conservation with a single XOR."

## Why Four Beats?

The four-beat structure mirrors how understanding actually forms:

1. **Attention** (HOOK) — Interrupt the pattern.
2. **Comprehension** (REVEAL) — Build the model.
3. **Integration** (CONNECT) — Place the model in context.
4. **Application** (ACTIVATE) — Convert the model into behavior.

Skip a beat and the chain reaction fizzles. A document with only HOOK and REVEAL produces enlightened inaction. A document with only CONNECT and ACTIVATE produces brittle recipes. All four are required.

## Experiments

You can verify whether a document follows the pattern by asking four questions:

1. Can I quote the HOOK in one sentence?
2. Does the REVEAL explain a causal mechanism?
3. Does CONNECT reference three or more related documents?
4. Does ACTIVATE contain an imperative instruction I could execute today?

Apply this rubric to any document in the agent-knowledge base. The expected result is that all core documents score 4/4, and any document that scores lower is flagged for revision.

## Applications

- **Agent ingestion**: Agents can parse documents more effectively when they know the four-beat structure in advance.
- **Documentation QA**: Use the rubric as a linter for new docs.
- **Context-window economics**: A four-beat document compresses well; an agent can reconstruct the full argument from the headings alone.
- **Onboarding acceleration**: New contributors learn the knowledge base faster when every page has the same shape.
- **Cross-link maintenance**: The CONNECT section forces authors to keep the document graph alive.

## Open Questions

1. **Length constraints**: Should there be a maximum length for each beat? A 10,000-word REVEAL may defeat the pattern's compressibility.
2. **Sub-beats**: Are there domains that need five or six beats? Does a mathematical proof need a separate COROLLARY beat?
3. **Nonlinear reading**: Can the pattern support readers who enter from CONNECT rather than HOOK, perhaps via hypergraph navigation?
4. **Evaluation**: Can we train a classifier to score documents on chain-reaction fidelity automatically?

## CONNECT

- [THE-AHA-MOMENT.md](THE-AHA-MOMENT.md) — The psychological event the chain-reaction pattern is engineered to produce.
- [LOOM-TAPESTRY.md](LOOM-TAPESTRY.md) — Why documentation structure is itself a deep engineering decision.
- [AGENT-SELF-ASSESSMENT.md](AGENT-SELF-ASSESSMENT.md) — How agents audit whether they have fully absorbed a document.
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — The resource pressure that makes compression beats necessary.
- [DESIGN-PRINCIPLES.md](DESIGN-PRINCIPLES.md) — The seven rules from which the four-beat structure is derived.

## ACTIVATE

Before you write your next document, open a blank file and write the four headings first: HOOK, REVEAL, CONNECT, ACTIVATE. Do not write body text until all four headings are present. Then fill each beat in order, enforcing the constraint that CONNECT must reference at least three other documents and ACTIVATE must contain at least one imperative sentence. When you finish, quote the HOOK aloud. If it does not reframe your own understanding, rewrite it until it does.
