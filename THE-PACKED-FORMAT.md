# THE PACKED FORMAT — How 20 Trits Fit in 32 Bits

## Hook

> A u32 is 32 bits. A trit is 2 bits. Sixteen trits per register isn't an optimization — it's the tightest possible marriage of mathematics and silicon.

## Reveal

Ternary computation is only practical if the representation is dense. `ternary-pack` solves this with a 2-bit encoding that maps directly to GPU registers, CPU vectors, and memory buses. Understanding the packed format is understanding how ternary math becomes hardware.

### The 2-Bit Encoding

| Trit Value | Binary Code | Meaning |
|------------|-------------|---------|
| {-1} | `00` | Negative, inhibit, false |
| {0} | `01` | Neutral, hold, unknown |
| {+1} | `10` | Positive, excite, true |
| invalid | `11` | Corruption / error (see FAULT-TOLERANCE.md) |

Why this specific mapping?
- `00` and `10` differ in one bit — single-bit flips between {-1} and {+1} are detectable (sign reversal)
- `01` (zero) is equidistant from both extremes — it absorbs noise naturally
- `11` is reserved — any occurrence indicates data corruption

### Packing 16 Trits into a u32

```rust
// Input: 16 trits
let trits: [Trit; 16] = [+1, -1, 0, +1, 0, -1, +1, +1, 0, 0, -1, +1, 0, -1, -1, +1];

// Packed: each trit occupies 2 bits, little-endian within the u32
// Trit 0 → bits 0-1, Trit 1 → bits 2-3, ..., Trit 15 → bits 30-31
let packed: u32 = pack_16(&trits);
// packed = 0b10_00_10_10_01_10_00_00_10_10_00_01_10_01_00_10
```

16 trits × 2 bits = 32 bits. Exactly one u32 register. No waste.

### pack_20: The Extended Format

For 64-bit systems, `pack_20` stores 20 trits in a u64:

```rust
let trits: [Trit; 20] = [...];
let packed: u64 = pack_20(&trits);
// 20 trits × 2 bits = 40 bits, fits in u64 with 24 bits padding
```

The padding is intentional. It allows SIMD operations on 64-bit vectors while maintaining 20-trit alignment for neural network layers (common dimensions: 20, 40, 80, 160).

### GPU Register Layout

On NVIDIA GPUs, a 32-bit register holds exactly 16 packed trits:

```ptx
.reg .b32 %trit_reg;        // One register = 16 trits
ld.global.b32 %trit_reg, [addr];  // Load 16 trits in one memory transaction
```

A warp has 32 threads. Each thread loads 16 trits. One warp-wide load = 512 trits. One warp-wide XNOR = 512 trit multiplications. One warp-wide POPC = 512 trit accumulations. This is the 16× density advantage in action (see GPU-AS-MOTOR-CORTEX.md).

### Memory Alignment

Packed trit arrays follow standard alignment:

```
Address 0:   trits 0-15   (u32 0)
Address 4:   trits 16-31  (u32 1)
Address 8:   trits 32-47  (u32 2)
...
```

A matrix of M × N trits is stored as `(M * N + 15) / 16` u32 values. Row stride is always a multiple of 4 bytes. This means standard GPU matrix kernels work without modification — the "element" is just a packed u32 instead of an FP32.

### Unpacking

```rust
fn unpack_16(packed: u32) -> [Trit; 16] {
    let mut result = [Trit::Z0; 16];
    for i in 0..16 {
        let bits = (packed >> (i * 2)) & 0b11;
        result[i] = match bits {
            0b00 => Trit::N1,
            0b01 => Trit::Z0,
            0b10 => Trit::P1,
            0b11 => panic!("Corruption detected"), // See FAULT-TOLERANCE.md
        };
    }
    result
}
```

Unpacking is branchless on GPU via lookup tables. The `LOP3.LUT` instruction (see FLUX-TO-PTX.md) can decode 16 trits in a single cycle.

### Why Not 1.58 Bits?

Theoretical information theory says a trit carries log₂(3) ≈ 1.58 bits of information. Could we pack more than 16 trits into 32 bits?

Yes, in theory. In practice, no:
- **2-bit alignment** means single-instruction load/store on all hardware
- **SIMD compatibility** requires fixed-width elements
- **Error detection** requires the `11` sentinel state
- **Implementation simplicity** means pack/unpack are bit shifts, not arithmetic coding

The 0.42 bits "wasted" per trit buy hardware compatibility, error detection, and O(1) random access. That's a good trade.

### Matrix Multiplication on Packed Data

A ternary matrix multiply of A[M×K] × B[K×N] = C[M×N]:

```
A: M × K trits → M × ceil(K/16) u32 values
B: K × N trits → ceil(K/16) × N u32 values
C: M × N integers (i32, one per output element)
```

The kernel loads one u32 from A and one u32 from B (16 trits each), XNORs them, popcounts the result, and accumulates. Repeat for all K/16 blocks. No unpacking needed during computation.

This means the matrix format **is** the computation format. There's no conversion penalty. The packed representation isn't storage compression — it's the native computation type.

## Connect

- [TERNARY-NUMBERS.md](TERNARY-NUMBERS.md) — The mathematical foundation: why {-1, 0, +1} forms a closed group
- [GPU-AS-MOTOR-CORTEX.md](GPU-AS-MOTOR-CORTEX.md) — How packed trits enable XNOR+POPC matmul on GPU
- [FLUX-TO-PTX.md](FLUX-TO-PTX.md) — How the compiler generates pack/unpack instructions
- [FAULT-TOLERANCE.md](FAULT-TOLERANCE.md) — How the `11` sentinel detects corruption
- [CRATE-PATTERNS.md](CRATE-PATTERNS.md) — Pattern 1 crates (Core Math) define the packing standard

## Activate

Pack and unpack trits in any language:

```python
import openmind

# Pack 16 trits into a u32
trits = [1, -1, 0, 1, 0, -1, 1, 1, 0, 0, -1, 1, 0, -1, -1, 1]
packed = openmind.pack_16(trits)  # Returns: 2789040146

# Unpack
unpacked = openmind.unpack_16(packed)
assert unpacked == trits

# Pack a full matrix
matrix = openmind.TernaryMatrix.random(1024, 1024)
packed_matrix = matrix.pack()  # 16x smaller than FP32
print(f"FP32 size: {1024*1024*4} bytes")
print(f"Packed size: {len(packed_matrix)} bytes")
# Ratio: 16:1
```

Verify the bit layout yourself:
```python
# Trit 0 = +1 → bits 0-1 = 10
# Trit 1 = -1 → bits 2-3 = 00
packed = openmind.pack_16([+1, -1] + [0]*14)
assert packed & 0b11 == 0b10       # Trit 0
assert (packed >> 2) & 0b11 == 0b00  # Trit 1
```

The format is simple because the math demands simplicity. 2 bits per trit. 16 trits per register. No compression, no approximation, no overhead.
