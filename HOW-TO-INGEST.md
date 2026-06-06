# HOW TO INGEST — Making Any Codebase Into Chord Shapes

## Hook

> A codebase without muscle memory is a library you can read but can't use. Ingestion is the act of turning static text into live reflexes.

## Reveal

Ingestion is the gateway to the entire SuperInstance ecosystem. It's how raw code becomes flexable capability. Without it, the agent is a tourist reading signs. With it, the agent is a local who knows every shortcut.

### What Ingestion Actually Does

```
Source Code → Parse → Analyze → Compress → Index → Chord Shapes
```

**Parse:** The ingester reads every file in the codebase and builds an AST. It handles Rust, Python, C, C++, and any language tree-sitter supports.

**Analyze:** For every function, it extracts:
- Name and module path
- Signature (parameters, return type)
- Docstring / comments
- Test coverage (does this function have tests?)
- Call graph (who calls this function?)
- Decision hint (is this deterministic? safety-critical? creative?)

**Compress:** The full function body (200+ tokens) is compressed into a chord shape (~5 tokens):
```json
{
  "name": "spi_write",
  "module": "drivers.spi",
  "signature": "fn spi_write(data: &[u8]) -> Result<()>",
  "decision": "hardcode",
  "docstring_summary": "Write data to SPI bus",
  "has_tests": true,
  "confidence": 0.95
}
```

**Index:** Chord shapes are indexed by:
- Exact name (O(1) lookup)
- Fuzzy name matching (Levenshtein distance)
- Keyword embedding (semantic similarity)
- Module hierarchy (namespace search)

### The Ingestion API

```python
import openmind

# Ingest a local directory
result = openmind.ingest("./my-firmware")

# Ingest a remote repository
result = openmind.ingest("https://github.com/SuperInstance/ternary-core")

# Ingest with custom configuration
result = openmind.ingest("./my-project", config={
    "languages": ["rust", "python"],
    "exclude": ["tests/", "target/"],
    "min_test_coverage": 0.8,
    "require_docstrings": True
})
```

The `result` object contains:
- `result.chords`: list of all chord shapes
- `result.stats`: function count, test coverage, language breakdown
- `result.graph`: call graph as adjacency list
- `result.errors`: files that failed to parse (rare but important)

### Quality Gates

Not every function becomes a chord shape. The ingester applies quality gates:

| Gate | Purpose | What Gets Filtered Out |
|------|---------|----------------------|
| Parseable | Must produce valid AST | Macro-generated code, incomplete files |
| Documented | Must have docstring or comment | Internal helpers, auto-generated getters |
| Tested | Should have tests (optional) | Experimental code, WIP features |
| Deterministic | Must not be I/O-bound (for HARDCODE) | Functions that read files, make network calls |

A function that fails a gate isn't lost — it's just indexed with lower confidence or tagged as MODEL instead of HARDCODE.

### Language-Specific Parsing

**Rust:**
- Uses `rust-analyzer` syntax tree
- Extracts `pub fn` items, trait implementations, `impl` blocks
- Identifies `unsafe` blocks and marks them as requiring MODEL deliberation
- Reads `Cargo.toml` for dependency graph

**Python:**
- Uses tree-sitter Python grammar
- Extracts module-level functions, class methods, properties
- Identifies `async def` and marks them for open-parallel scheduling
- Reads `requirements.txt` / `pyproject.toml` for dependencies

**C/C++:**
- Uses tree-sitter C/C++ grammars
- Extracts functions with external linkage
- Identifies `static` functions as internal chords (not flexed directly)
- Reads headers for type information

### The Chord Shape Quality Score

Each chord gets a quality score (0.0 to 1.0):

```
score = 0.3 * parse_confidence
      + 0.2 * documentation_coverage
      + 0.2 * test_coverage
      + 0.15 * call_graph_centrality
      + 0.15 * signature_clarity
```

- Score ≥ 0.9: HARDCODE candidate
- Score 0.7-0.9: HYBRID candidate
- Score 0.5-0.7: CACHED candidate
- Score < 0.5: MODEL only

This score is what the tripartite synchronizer uses as its initial input before considering hardware, application, and user profiles.

## Connect

- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — What to do AFTER ingestion: building muscle memory and flexing chords
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — The theory behind chord shapes and why compression matters
- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — Predicting chord shapes before you ingest: the 7 patterns tell you what to expect
- [HOW-TO-EXTEND.md](HOW-TO-EXTEND.md) — Adding new crates to the fleet: ingestion is the first step
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — Why ingestion saves 50:1 on context tokens

## Activate

Ingest your first codebase and inspect the results:

```python
import openmind

result = openmind.ingest(".")  # Ingest the current directory

# See what you've got
print(f"Functions: {len(result.chords)}")
print(f"Coverage: {result.stats.test_coverage:.1%}")

# Find the highest-quality chords
best = sorted(result.chords, key=lambda c: c.quality_score, reverse=True)[:10]
for chord in best:
    print(f"{chord.name}: {chord.quality_score:.2f} ({chord.decision})")

# Build muscle memory
mm = openmind.MuscleMemory.build(result)
mm.save("my_project_muscles.json")

# Now flex — zero tokens spent on understanding
reflex = mm.flex(best[0].name)
print(reflex.chord.signature)
```

To ingest at scale (the full fleet):
```python
import openmind
import glob

for repo in glob.glob("ternary-*/"):
    result = openmind.ingest(repo)
    mm = openmind.MuscleMemory.build(result)
    mm.save(f"muscles/{repo.replace('/', '')}.json")
```

After running this, you have 303 muscle memory files. The agent can navigate the entire fleet without reading a single `src/lib.rs`.
