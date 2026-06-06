# HOW TO EXTEND — Adding Your Own Crates to the Fleet

## Hook

> Every new ternary crate is a new chord shape for the agent's repertoire.
> And every chord shape is attention freed for higher-level thinking.

## The Pattern

Adding a crate to the ecosystem is a 4-step process:

### Step 1: Pick Your Domain

What problem are you solving? Every ternary crate applies {-1, 0, +1} to a specific domain.

Ask yourself:
- What am I classifying? (→ ternary signal)
- What am I deciding? (→ ternary vote)
- What am I controlling? (→ ternary PID)
- What am I searching? (→ ternary graph)

If your answer involves yes/no/maybe, up/down/hold, positive/negative/neutral — it's ternary.

### Step 2: Choose a Pattern

From [CRATE-PATTERNS.md](CRATE-PATTERNS.md), pick the closest match:

| Your domain | Pattern | Example crates |
|-------------|---------|---------------|
| Arithmetic, types, encoding | Core Math | ternary-core, ternary-pack |
| Filtering, transforming, analyzing | Signal Processing | ternary-signals, ternary-warp |
| Storing, routing, scheduling | Data Structures | ternary-heap, ternary-cache |
| Voting, agreeing, deciding | Consensus | ternary-consensus, ternary-paxos |
| Composing, generating, classifying | Creative | ternary-music, ternary-counterpoint |
| Controlling, monitoring, regulating | Systems Control | ternary-thermostat, ternary-pid |
| Proving, verifying, authenticating | Formal | ternary-proof, ternary-zkp |

### Step 3: Build It

```bash
# Create the repo
mkdir ternary-YOURNAME && cd ternary-YOURNAME
cargo init --lib
```

Follow the standard structure:
```rust
// src/lib.rs

/// Your module's purpose (one line).
///
/// Longer description of what this crate does and why it matters.

/// Trit type (import from ternary-types if available, or define locally)
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum Trit {
    MinusOne = -1,
    Zero = 0,
    PlusOne = 1,
}

/// Your core function. Every crate has ONE main abstraction.
pub fn your_function(input: &[Trit]) -> Trit {
    // Implementation using ternary arithmetic
    // (a + b + 3) % 3 is WRONG — use explicit match arms!
    todo!("Implement your ternary logic")
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_basic() {
        // Test with {-1, 0, +1} inputs
        // Verify outputs are in {-1, 0, +1}
    }
}
```

### Step 4: Document It

Every crate gets a research-grade README (3-8KB) with:
1. **Background**: What domain are you in? Why ternary for this?
2. **Architecture**: What's the main abstraction? How does it work?
3. **Test Results**: Run `cargo test` and include the output
4. **Use Cases**: 3-5 real-world applications
5. **Open Questions**: What would you research next?
6. **Oxide Stack Connection**: How does this connect to the five-layer architecture?

### Step 5: Ingest It

```bash
# Add to the fleet
openmind ingest ./ternary-YOURNAME

# Build muscle memory
openmind save ./ternary-YOURNAME YOURNAME_muscles.json

# Verify
openmind stats YOURNAME_muscles.json
```

Your crate is now a set of chord shapes that any agent can flex.

## The Extension Points

The ecosystem is designed to grow at these points:

### New Crates (Any Domain)
- Follow the pattern above
- Name it `ternary-<domain>` for Rust, `ternary-<domain>-python` for Python, `ternary-<domain>-c` for C
- Push to `SuperInstance/ternary-<domain>`

### New Hardware Targets
- ESP32: via openmind-esp32-bridge
- FPGA: ternary logic gates compile directly to LUTs
- RISC-V: custom ternary ISA extensions
- Browser: via ternary-wasm (already exists)

### New Agent Types
- Python agent: `import openmind` (already exists)
- Rust agent: use openmind-conductor (being built now)
- Browser agent: via OpenClaw + Jupyter magic
- Embedded agent: runs ON the ESP32 itself

### New Transport Layers
- Serial (already in esp32-bridge)
- MQTT (already in esp32-bridge)
- WebSocket (already in esp32-bridge)
- BLE: Bluetooth Low Energy for mobile agents
- LoRa: Long-range for field deployments
- CAN bus: Automotive/industrial

## Connect

- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — The 7 templates every crate follows
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — Why {-1,0,+1} for your domain
- [MUSCLE-MEMORY.md](MUSCLE-MEMORY.md) — How your crate becomes chord shapes

## Activate

Start here:
1. `cargo init --lib ternary-mydomain`
2. Write 3 tests first
3. Implement the core function
4. Run `cargo test`
5. Write the README
6. `openmind ingest ./ternary-mydomain`
7. Push to SuperInstance

You just added chord shapes to an agent's repertoire. Every function you wrote is attention the agent won't have to spend next time.
