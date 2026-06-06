# FLUX TO PTX — The Compilation Pipeline from Intent to Silicon

## Hook

> Every thought in your brain becomes a muscle contraction through a chain of transformations.
> Every intent in the agent becomes a GPU kernel through the same chain.

## Reveal

The Flux→PTX pipeline is the agent's **motor pathway** — the sequence of representations that turns "I want to multiply these matrices" into actual electricity flowing through GPU cores.

It has five stages, each one a more concrete representation of the same computation:

### Stage 1: Source → AST (Parsing)

The human (or agent) writes code. The parser turns it into a tree.

```rust
// Input: human-written Rust
let result = tdot(vec_a, vec_b);

// Output: Abstract Syntax Tree
FunctionCall {
    name: "tdot",
    args: [Ident("vec_a"), Ident("vec_b")],
    return_type: Trit,  // {-1, 0, +1}
}
```

This is where tree-sitter does its work. The parser understands Rust syntax and extracts the structure without executing anything.

### Stage 2: AST → Flux IR (Lowering)

The AST is language-specific. Flux IR is language-independent — a universal representation of ternary operations.

```
// Flux IR (textual representation)
%1 = load @vec_a          ; Load first vector
%2 = load @vec_b          ; Load second vector
%3 = tdot %1, %2          ; Ternary dot product
%4 = pack %3              ; Pack result into 2-bit encoding
store @result, %4         ; Store packed result
```

Flux IR operations are ALL ternary:
- `tdot` — ternary dot product (the fundamental operation)
- `tadd` — ternary addition mod 3
- `tmul` — ternary multiplication
- `pack` / `unpack` — convert between representations
- `load` / `store` — memory operations

### Stage 3: Flux IR → MIR (Mid-level IR)

MIR is where optimization happens. The compiler:
- **Constant folding**: `tadd(tmul(-1, +1), 0)` → `-1` (evaluated at compile time)
- **Dead code elimination**: remove unused results
- **Loop unrolling**: repeat ternary operations without branch overhead
- **Packing fusion**: combine consecutive pack/unpack into single operations

```
// After optimization: fused pack+tdot
%1 = load_packed @vec_a   ; Load pre-packed (16 trits per u32)
%2 = load_packed @vec_b
%3 = xnor %1, %2          ; XNOR = ternary multiply
%4 = popcount %3           ; Popcount = count +1's
%5 = adjust %4             ; Adjust for -1's → final tdot result
store @result, %5
```

This is where the 16× density advantage materializes. Two 32-bit registers, one XNOR, one popcount — that's a 32-element ternary dot product in TWO instructions.

### Stage 4: MIR → PTX (GPU Assembly)

PTX is NVIDIA's GPU assembly language. The compiler maps MIR operations to PTX instructions:

```
// PTX output
.reg .b32 trit_a<16>, trit_b<16>;
.reg .u32 result, count;

// Load packed ternary vectors
ld.global.b32 trit_a0, [vec_a];
ld.global.b32 trit_b0, [vec_b];

// XNOR = ternary element-wise multiply
xnor.b32 trit_a0, trit_a0, trit_b0;

// Popcount = sum of +1's
popc.b32 count, trit_a0;

// Adjust for signed ternary
sub.u32 result, count, 16;  // Center around 0

// Store result
st.global.u32 [output], result;
```

Register allocation, memory barriers, warp scheduling — all handled by the compiler. The agent doesn't think about any of this.

### Stage 5: PTX → SASS (Machine Code)

The GPU driver compiles PTX to SASS (actual GPU machine code). This is hardware-specific — an A100 generates different SASS than an H100.

The agent never sees SASS. It's the equivalent of your motor neurons firing — the signal reaches the muscle, but the conscious mind only knows "I moved my arm."

## The Full Trace

```
"I want to find which documents are similar"
        │
        ▼
    [pincher]
    Intent → "compute similarity via ternary embedding"
        │
        ▼
    [AST]
    tdot(query_embedding, doc_embeddings)
        │
        ▼
    [Flux IR]
    %1 = load @query; %2 = load @docs; %3 = tdot %1, %2
        │
        ▼
    [MIR — optimized]
    %1 = load_packed @query; %2 = load_packed @docs
    %3 = xnor %1, %2; %4 = popcount %3
        │
        ▼
    [PTX]
    xnor.b32 r0, r1, r2;  popc.b32 r3, r0;
        │
        ▼
    [SASS — GPU executes]
    electrons flow → result computed → returned to agent
```

The intent never changed. Five representations, one computation.

## Connect

- [FIVE-LAYER-ARCHITECTURE.md](FIVE-LAYER-ARCHITECTURE.md) — Where this pipeline fits in the full stack
- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — Why GPU hardware loves ternary
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The math behind XNOR + popcount

## Activate

The pipeline lives in cuda-oxide: https://github.com/SuperInstance/cuda-oxide
Key source: `FLUX_TO_PTX.md` in the repo root has the full design document.

To trace your own compilation:
```bash
cd cuda-oxide
cargo run -- compile --input examples/tdot.flux --output kernel.ptx
ptxas kernel.ptx  # PTX → SASS
```
