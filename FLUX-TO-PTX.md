# FLUX TO PTX — The Compilation Pipeline from Intent to Silicon

## Hook

> Every program in this ecosystem passes through five transformations before it touches hardware. The most dangerous one isn't the first or the last — it's the middle, where portable intent becomes machine-specific reality.

## Reveal

The Flux→PTX pipeline has six stages. Each stage is a faithful representation of the same computation, but the transformation between stages is where correctness lives or dies. SuperInstance doesn't treat these as black boxes. It treats them as **verification checkpoints**.

---

### Stage 1: Source → AST (Parsing)

The input is human or agent intent expressed in Rust, Python, or Flux syntax. The parser produces an Abstract Syntax Tree.

**Input:**
```rust
fn similarity(query: &[Trit], docs: &Matrix<Trit>) -> Vec<i32> {
    docs.rows().map(|row| tdot(query, row)).collect()
}
```

**AST (simplified):**
```
FunctionDecl "similarity"
  Param "query" : Slice(Trit)
  Param "docs" : Matrix(Trit)
  Return : Vec(i32)
  Body:
    MethodCall "map"
      Receiver: MethodCall "rows" (Receiver: Ident "docs")
      Closure:
        Param "row"
        Body: Call "tdot" [Ident "query", Ident "row"]
      Collect
```

What happens here: tree-sitter parses the syntax. Type inference resolves `Trit` to the 2-bit packed representation. The AST captures **what** the code means, stripped of syntactic sugar.

**Verification checkpoint:** Parse must succeed without ambiguity. If the AST has unresolved types, the pipeline stops.

---

### Stage 2: AST → Flux IR (Lowering)

The AST is language-specific. Flux IR is universal — a static single-assignment representation of ternary operations that any backend can consume.

**Flux IR:**
```
fn similarity(%query: ptr<trit>, %docs: ptr<matrix<trit>>) -> vec<i32> {
  %n_rows = load.u32 [%docs + 0]           ; matrix metadata
  %n_cols = load.u32 [%docs + 4]
  %result = alloc.vec<i32> %n_rows

  loop.header %i = 0, %n_rows:
    %row_ptr = gep %docs, %i, %n_cols       ; get row pointer
    %a = load.packed.trits %query, %n_cols  ; 16 trits per u32
    %b = load.packed.trits %row_ptr, %n_cols
    %xnor = ternary.xnor %a, %b             ; element-wise match
    %popc = ternary.popcount %xnor          ; count matches
    %adj = sub.i32 %popc, %n_cols           ; center around 0
    store [%result + %i * 4], %adj
  loop.next %i

  ret %result
}
```

What happens here: all high-level constructs lower to explicit memory operations. `map` becomes a loop. `tdot` decomposes into `load → xnor → popcount → adjust`. Types become concrete: `Trit` becomes `u32` with 2-bit encoding.

**Verification checkpoint:** The lowering preserves semantics. `tdot(a, b)` in AST and `xnor→popcount→adjust` in IR compute the same Z₃ dot product. This is proven by the ternary-pack theorem.

---

### Stage 3: Flux IR → MIR (Mid-level IR)

MIR is where optimization happens without losing ternary semantics. This is the critical stage — the last representation that is still hardware-independent.

**Before optimization:**
```
loop.header %i = 0, %n_rows:
  %a = load.packed.trits %query, %n_cols    ; Reload query every iteration
  %b = load.packed.trits %row_ptr, %n_cols
  %xnor = ternary.xnor %a, %b
  %popc = ternary.popcount %xnor
  ...
```

**After optimization:**
```
%query_packed = load.packed.trits %query, %n_cols  ; Hoist outside loop

loop.header %i = 0, %n_rows:
  %b = load.packed.trits %row_ptr, %n_cols
  %xnor = ternary.xnor %query_packed, %b           ; XNOR with hoisted query
  %popc = ternary.popcount %xnor
  %adj = sub.i32 %popc, %n_cols
  store [%result + %i * 4], %adj
loop.next %i
```

**Optimizations applied:**
- **Loop-invariant code motion:** `query` is loaded once, not `n_rows` times
- **Packing fusion:** Consecutive `pack`/`unpack` operations collapse
- **Constant folding:** `sub.i32 %popc, 16` when `n_cols` is known at compile time
- **Dead store elimination:** Unused intermediate trit vectors are dropped

What happens here: the compiler reshapes the computation for efficiency while preserving the invariant that the result is identical to the unoptimized IR. This is where the 16× density advantage materializes.

**Verification checkpoint:** Every optimization must have a witness — a proof that the transform is semantics-preserving. If the compiler can't prove an optimization is safe, it doesn't apply it.

---

### Stage 4: MIR → PTX (GPU Assembly)

PTX is NVIDIA's intermediate assembly. It's hardware-portable across GPU generations but explicit about registers, memory, and warps.

**PTX output (from cuda-oxide):**
```ptx
.version 7.8
.target sm_80
.address_size 64

.entry similarity(
  .param .u64 query,
  .param .u64 docs,
  .param .u64 output,
  .param .u32 n_rows,
  .param .u32 n_cols
)
{
  .reg .pred %p<2>;
  .reg .b32 %r<16>;
  .reg .b64 %rd<8>;

  // Load parameters
  ld.param.u64 %rd0, [query];
  ld.param.u64 %rd1, [docs];
  ld.param.u64 %rd2, [output];
  ld.param.u32 %r0, [n_rows];
  ld.param.u32 %r1, [n_cols];

  // Compute row stride: stride = (n_cols + 15) / 16 * 4 bytes
  add.u32 %r2, %r1, 15;
  shr.u32 %r2, %r2, 4;
  shl.u32 %r2, %r2, 2;

  // Load query (hoisted, loaded once)
  ld.global.b32 %r3, [%rd0];

  // Loop: one iteration per row
  mov.u32 %r4, 0;           // i = 0

LOOP:
  setp.ge.u32 %p0, %r4, %r0;
  @%p0 bra END;

  // Compute row pointer: docs + i * stride
  mul.wide.u32 %rd3, %r4, %r2;
  add.u64 %rd4, %rd1, %rd3;

  // Load document row
  ld.global.b32 %r5, [%rd4];

  // THE TERNARY OPERATION: 2 instructions
  xnor.b32 %r6, %r3, %r5;   // Match bits
  popc.b32 %r7, %r6;        // Count matches

  // Adjust: result = popcount - n_cols (center around 0)
  sub.u32 %r8, %r7, %r1;

  // Store result
  mul.wide.u32 %rd5, %r4, 4;
  add.u64 %rd6, %rd2, %rd5;
  st.global.u32 [%rd6], %r8;

  // Increment and branch
  add.u32 %r4, %r4, 1;
  bra LOOP;

END:
  ret;
}
```

What happens here: register allocation maps MIR temporaries to physical GPU registers. Memory operations get explicit addressing modes. The loop becomes a branch with a predicate register. Most importantly: the ternary dot product is **exactly** `xnor` + `popc` — no approximation, no emulation.

**Verification checkpoint:** The PTX must be symbolically equivalent to the MIR. cuda-oxide generates a **witness** for each instruction mapping:
```
MIR: %popc = ternary.popcount %xnor
PTX: popc.b32 %r7, %r6
Witness: popcount(xnor(a, b)) ≡ Σ(a[i] * b[i]) for packed trits
  Proof: ternary-pack theorem (see TERNARY-NUMBERS.md)
```

---

### Stage 5: PTX → SASS (Machine Code)

PTX is portable. SASS is not. The NVIDIA driver (or `ptxas`) compiles PTX to SASS — the actual machine code the GPU executes.

**SASS (disassembled, sm_80):**
```sass
/* xnor.b32 %r6, %r3, %r5 */
0x5c0037800c0de020  LOP3.LUT  R6, R3, R5, RZ, 0xc0, !PT

/* popc.b32 %r7, %r6 */
0x5c003980000e0600  POPC      R7, R6

/* sub.u32 %r8, %r7, %r1 */
0x5c100800001e0704  IADD3     R8, R7, -R1, RZ
```

What happens here: `XNOR` becomes `LOP3.LUT` (lookup-table logic op). `POPC` stays `POPC`. `SUB` becomes `IADD3` with a negated operand. Register numbers map to physical register files. The compiler schedules instructions to hide memory latency.

**Verification checkpoint:** This stage is outside SuperInstance's control, but it's safe because:
- PTX is a strict semantic subset of SASS
- NVIDIA's assembler is formally verified
- The witness from Stage 4 still applies (PTX semantics = SASS semantics)

---

### Stage 6: SASS → Execution (Silicon)

The GPU scheduler dispatches warps. Registers are read. ALUs fire. Memory buses transfer data. This is the only stage where anything actually **happens**. Everything before was description.

In a single clock cycle on an A100:
- 64 threads in a warp execute the `LOP3.LUT` simultaneously
- Each thread XNORs two 32-bit registers (16 trits each)
- Next cycle: 64 `POPC` instructions execute
- Result: 64 dot products of 16-element ternary vectors in 2 cycles

That's 1,024 ternary multiply-accumulate operations in 2 clock cycles. The equivalent FP32 operation would require 64 threads × 16 muls + 15 adds = 1,984 operations. The ratio holds: ternary is ~31× fewer instructions at the silicon level.

## Connect

- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — Why XNOR+POPC is the native language of GPU hardware
- [FIVE-LAYER-ARCHITECTURE.md](FIVE-LAYER-ARCHITECTURE.md) — Where this pipeline sits in the full stack (Layers 2-5)
- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The mathematical proof that XNOR+POPC equals Z₃ dot product
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — How verification entropy is preserved across all six stages
- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — Pattern 6 (Systems) crates often bridge the compilation gap

## Activate

Trace a compilation through all six stages:

```bash
# 1. Write Flux source
cat > similarity.flux << 'FLUX'
fn similarity(query: &[Trit], docs: &Matrix<Trit>) -> Vec<i32> {
  docs.rows().map(|row| tdot(query, row)).collect()
}
FLUX

# 2. Source → AST → Flux IR
cuda-oxide parse similarity.flux --emit ir > similarity.ir

# 3. Flux IR → MIR (optimized)
cuda-oxide optimize similarity.ir --emit mir > similarity.mir

# 4. MIR → PTX
cuda-oxide codegen similarity.mir --target sm_80 > similarity.ptx

# 5. PTX → SASS
ptxas -arch sm_80 similarity.ptx -o similarity.cubin
cuobjdump -sass similarity.cubin > similarity.sass

# 6. Execute
nvcc -ptx similarity.ptx -o similarity.o
# Link and run via cudaclaw
```

To verify conservation at each stage:
```python
import openmind

# Compile with full trace
result = openmind.compile("similarity.flux", trace=True)

for stage in result.trace:
    print(f"{stage.name}: {stage.witness}")
    print(f"  Verification entropy: {stage.entropy}")
    print(f"  Instructions: {stage.instruction_count}")
```

Every stage should show entropy ≥ 4 and a valid witness. If any stage shows entropy dropping or a missing witness, the pipeline has a leak — and the compilation aborts.

The agent's intent becomes silicon through six transformations. But with witnesses at every boundary, it's still the same intent.
