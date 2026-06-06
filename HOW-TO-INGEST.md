# HOW TO INGEST — Making Any Codebase Into Chord Shapes

## Hook

> Any codebase — Rust, Python, C, JavaScript — becomes muscle memory in 3 seconds.

## The Ingestion Pipeline

```
Source Code → File Discovery → Language Detection → AST Parsing → Extraction → Indexing
```

### Step 1: File Discovery

```python
result = openmind.ingest("./my-repo")
```

The ingester walks the directory tree, skipping:
- `.git/`, `target/`, `node_modules/`, `__pycache__/`
- Build artifacts, configuration files, assets
- Only processes source files (.py, .rs, .c, .h, .cpp, .js, .ts)

### Step 2: Language Detection

File extension → parser selection:
- `.py` → Python AST (built-in `ast` module, always available)
- `.rs` → tree-sitter-rust (optional, requires `[tree-sitter]` extra)
- `.c/.h` → tree-sitter-c
- `.cpp/.hpp` → tree-sitter-cpp
- `.js` → tree-sitter-javascript
- `.ts` → tree-sitter-typescript

**Fallback**: If tree-sitter isn't installed, only Python files are parsed. The system still works — just with fewer languages.

### Step 3: AST Parsing

For each source file, the parser extracts:

**Functions:**
```python
FunctionInfo(
    name="process_data",
    module="pipeline.processor",
    file_path="pipeline/processor.py",
    line_start=42,
    line_end=67,
    signature="def process_data(data: list[dict], threshold: float = 0.5) -> list[dict]",
    docstring="Process raw sensor data through the cleaning pipeline.",
    arg_names=["data", "threshold"],
    arg_types={"data": "list[dict]", "threshold": "float"},
    return_type="list[dict]",
    calls=["clean", "filter", "normalize", "validate"],
    called_by=[],  # Filled in later
    has_tests=False,  # Filled in later
)
```

**Classes:**
```python
ClassInfo(
    name="DataProcessor",
    module="pipeline.processor",
    methods=["__init__", "process", "validate", "export"],
    bases=["BaseProcessor"],
    docstring="Main data processing class with validation pipeline.",
)
```

### Step 4: Call Graph Resolution

After parsing all files, the ingester:
1. Builds a global call graph: `caller → [callees]`
2. Resolves reverse references: `callee → [callers]` (who calls this?)
3. This enables "degree" calculation — how connected each function is

High-connectivity functions (called by 5+) get HARDCODE decisions.
Low-connectivity functions (called by 0-1) get MODEL/HYBRID.

### Step 5: Test Detection

Heuristic: a file is a test file if:
- Name starts with `test_` or ends with `_test.py`
- Name starts with `test_` or ends with `.rs` in a `tests/` directory
- Contains `assert`, `#[test]`, `def test_`

Functions whose names appear in test files (via regex search) get `has_tests=True`.

Tested functions get higher confidence scores in their Reflexes.

### Step 6: Muscle Memory Build

```python
mm = openmind.MuscleMemory.build(result)
```

For each extracted function:
1. Extract intent keywords (name parts, arg names, docstring words, called functions)
2. Run tripartite synchronizer with default profiles
3. Hash the source code for change detection
4. Compress into a Chord shape (name, module, signature, decision, keywords)

## What You Get

```python
mm.stats()
# {
#   'total_chords': 58,
#   'muscle_memory': 42,     # HARDCODE + CACHED
#   'needs_thinking': 16,    # MODEL + HYBRID
#   'tested': 23,
#   'untested': 35,
#   'decision_breakdown': {'hardcode': 30, 'cached': 12, 'hybrid': 10, 'model': 6}
# }
```

## Ingestion Time

| Repo Size | Files | Parse Time | Build Time | Total |
|-----------|-------|-----------|-----------|-------|
| Small (1-10 files) | 5 | 0.1s | 0.05s | 0.15s |
| Medium (10-100) | 50 | 0.5s | 0.3s | 0.8s |
| Large (100-1000) | 300 | 3s | 1.5s | 4.5s |
| Fleet (300+ crates) | 1000+ | 10s | 5s | 15s |

The entire 303-crate ternary fleet ingests in ~15 seconds. After that, it's saved to JSON and loads instantly.

## Connect

- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — What to do with the ingested data
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — Why the fleet ingests so cleanly
- [CONTEXT-WINDOW-ECONOMICS.md](CONTEXT-WINDOW-ECONOMICS.md) — The token savings from ingestion

## Activate

```bash
# CLI
openmind ingest ./any-repo

# Python
import openmind
result = openmind.ingest("./any-repo")
mm = openmind.MuscleMemory.build(result)
mm.save("repo_muscles.json")
```
