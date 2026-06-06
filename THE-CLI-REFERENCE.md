# THE CLI REFERENCE — Command-Line Tools for Agent Operations

## Hook

> The mouse is for exploration. The command line is for operation. Every capability in the SuperInstance ecosystem is exposed through a CLI that agents can script.

## Reveal

The `openmind` CLI is the agent's shell. It ingests, flexes, conducts, and debugs — all without writing Python.

### Installation

```bash
pip install openmind
openmind --version  # Should print version and commit hash
```

### Core Commands

#### `openmind ingest`

Turn any codebase into chord shapes.

```bash
# Basic ingestion
openmind ingest ./my-project

# With options
openmind ingest ./my-project \
  --languages rust,python \
  --exclude tests/,target/ \
  --output my_project_muscles.json

# Remote repo
openmind ingest https://github.com/SuperInstance/ternary-core \
  --output ternary-core_muscles.json
```

Output: A JSON file of chord shapes. Size is typically 1-5% of the source code.

#### `openmind flex`

Execute a capability from muscle memory.

```bash
# Load and flex
openmind flex my_project_muscles.json "spi_write" \
  --param data="0x01,0x02"

# With decision override
openmind flex my_project_muscles.json "novel_function" \
  --strategy model  # Force LLM generation

# Batch flex
openmind flex my_project_muscles.json "process_batch" \
  --param input_file="data.csv" \
  --output results.json
```

#### `openmind conduct`

Orchestrate multiple agents.

```bash
# Start a conductor with a score
openmind conduct --score greenhouse_score.json --tempo 100

# Add agents
openmind conduct --add-agent sensor-node-1 --section sensors
openmind conduct --add-agent actuator-node-1 --section actuators

# Start the performance
openmind conduct --start

# Monitor
openmind conduct --status
```

#### `openmind compile`

Compile Flux source to PTX.

```bash
# Basic compilation
openmind compile my_kernel.flux --output kernel.ptx

# With optimization
openmind compile my_kernel.flux --opt-level 3 --target sm_80

# With full trace
openmind compile my_kernel.flux --trace --emit-all
```

#### `openmind verify`

Check conservation across compilation layers.

```bash
# Verify a compiled kernel
openmind verify kernel.ptx --against my_kernel.flux

# Verify fleet compatibility
openmind verify --pin fleet.pin --check-all
```

#### `openmind benchmark`

Measure performance.

```bash
# Benchmark a ternary operation
openmind benchmark ternary_matmul --size 4096 --device cuda:0

# Compare against FP32
openmind benchmark matmul --size 4096 --baseline fp32

# Generate report
openmind benchmark --report json --output benchmark.json
```

#### `openmind doctor`

Diagnose system health.

```bash
openmind doctor
# Output:
# [PASS] ternary-core: v0.47.1
# [PASS] cuda-oxide: compatible driver
# [PASS] muscle memory: 6,000 chords, entropy 4.2
# [WARN] cache hit rate: 45% (expected > 80%)
# [FAIL] agent-node-3: unreachable
```

### Environment Variables

| Variable | Default | Effect |
|----------|---------|--------|
| `OPENMIND_CACHE_DIR` | `~/.openmind/cache` | Nail file location |
| `OPENMIND_LOG_LEVEL` | `INFO` | Verbosity: DEBUG, INFO, WARN, ERROR |
| `CUDA_OXIDE_OPT_LEVEL` | `3` | Compiler optimization: 0-3 |
| `OPENMIND_GPU_DEVICE` | `0` | Default CUDA device |
| `OPENMIND_MQTT_BROKER` | `localhost:1883` | Multi-agent message broker |

### Shell Completion

```bash
# Bash
openmind --generate-completion bash > /etc/bash_completion.d/openmind

# Zsh
openmind --generate-completion zsh > /usr/share/zsh/site-functions/_openmind

# Fish
openmind --generate-completion fish > ~/.config/fish/completions/openmind.fish
```

### Scripting with the CLI

The CLI is designed for pipes and scripts:

```bash
# Ingest, build, and flex in one pipeline
openmind ingest . | openmind build | openmind flex "main"

# Batch process all repos
for repo in */; do
  openmind ingest "$repo" --output "${repo%/}.json"
done

# Verify entire fleet
find . -name "*.pin" -exec openmind verify --pin {} \;
```

Every CLI command returns structured output (JSON with `--json` flag) for agent parsing.

## Connect

- [HOW-TO-FLEX.md](HOW-TO-FLEX.md) — Python API for flex (the CLI wraps this)
- [HOW-TO-CONDUCT.md](HOW-TO-CONDUCT.md) — Multi-agent orchestration via Python
- [DEBUGGING-AND-TRACING.md](DEBUGGING-AND-TRACING.md) — Use `openmind compile --trace` for debugging
- [DEPLOYMENT-AND-OPERATIONS.md](DEPLOYMENT-AND-OPERATIONS.md) — CLI for production deployment

## Activate

Set up your agent shell:

```bash
# 1. Install
pip install openmind

# 2. Ingest a project
openmind ingest ./my-project --output muscles.json

# 3. Explore
openmind flex muscles.json --list-all  # Show all chords
openmind flex muscles.json "main"       # Execute main

# 4. Monitor
openmind doctor
```

The CLI is the fastest way for an agent to interact with the fleet. Three commands: ingest, flex, doctor. Everything else is optional.
