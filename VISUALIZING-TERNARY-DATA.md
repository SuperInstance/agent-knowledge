# VISUALIZING TERNARY DATA — Making {-1, 0, +1} Visible

## Hook

> A trit vector is invisible until you map it to color. {-1} as blue, {0} as white, {+1} as red — suddenly the structure of 303 crates becomes a tapestry you can read at a glance.

## Reveal

Ternary data is compact but abstract. Visualization translates trits into human-readable patterns. The right visualization makes bugs obvious, structure visible, and consensus intuitive.

### The Ternary Color Map

| Trit | Color | Hex | Use |
|------|-------|-----|-----|
| {-1} | Deep Blue | `#2166ac` | Negative, inhibit, cool |
| {0} | White | `#f7f7f7` | Neutral, hold, rest |
| {+1} | Deep Red | `#b2182b` | Positive, excite, heat |

This is a diverging colormap centered at zero. It encodes both magnitude (saturation) and direction (hue).

### Visualizing a Trit Vector

```python
import matplotlib.pyplot as plt
import numpy as np

# A 64-element trit vector
trits = [+1, +1, 0, -1, +1, 0, 0, -1, ...]  # 64 elements

# As a heat strip
plt.figure(figsize=(12, 1))
plt.imshow([trits], cmap='RdBu_r', vmin=-1, vmax=+1, aspect='auto')
plt.xticks([])
plt.yticks([])
plt.title("Trit Vector: 64 Elements")
plt.show()
```

Patterns become visible: runs of red, blocks of white, alternating stripes. A well-formed ternary vector has structure. Random noise looks uniformly gray.

### Visualizing a Ternary Matrix

```python
# A 32×32 ternary weight matrix
W = np.random.choice([-1, 0, +1], size=(32, 32))

plt.figure(figsize=(6, 6))
plt.imshow(W, cmap='RdBu_r', vmin=-1, vmax=+1)
plt.colorbar(ticks=[-1, 0, +1], label='Trit value')
plt.title("Ternary Weight Matrix")
plt.show()
```

What to look for:
- **Sparsity:** White pixels indicate zero weights. High sparsity = efficient computation.
- **Structure:** Diagonal patterns indicate self-connections. Blocks indicate feature groups.
- **Balance:** Equal red and blue suggests the matrix is well-conditioned.

### Visualizing Consensus

```python
# 10 agents voting on 5 proposals
votes = np.array([
    [+1, -1, 0, +1, -1],   # Agent 0
    [+1, +1, 0, +1, -1],   # Agent 1
    [-1, -1, 0, 0, +1],    # Agent 2
    ...
])

plt.figure(figsize=(8, 6))
plt.imshow(votes, cmap='RdBu_r', vmin=-1, vmax=+1)
plt.xlabel("Proposal")
plt.ylabel("Agent")
plt.title("Consensus Matrix")
plt.colorbar(ticks=[-1, 0, +1])
plt.show()
```

Interpretation:
- **Uniform rows:** One agent votes the same on everything (possible bias)
- **Uniform columns:** All agents agree (strong consensus)
- **Scattered:** No consensus (needs more discussion)

### Visualizing the Fleet as a Graph

The 303 crates form a network. Visualize it:

```python
import networkx as nx

# Build graph from dependency data
G = nx.DiGraph()
for crate in fleet.crates:
    for dep in crate.dependencies:
        G.add_edge(crate.name, dep.name)

# Color by pattern
pattern_colors = {
    1: '#e41a1c',  # Core Math: red
    2: '#377eb8',  # Signal: blue
    3: '#4daf4a',  # Data: green
    4: '#984ea3',  # Consensus: purple
    5: '#ff7f00',  # Creative: orange
    6: '#ffff33',  # Systems: yellow
    7: '#a65628',  # Formal: brown
}

pos = nx.spring_layout(G, k=0.3)
colors = [pattern_colors[fleet.get_pattern(node)] for node in G.nodes()]

plt.figure(figsize=(14, 14))
nx.draw(G, pos, node_color=colors, node_size=30, alpha=0.7, arrows=False)
plt.title("Ternary Fleet: 303 Crates by Pattern")
plt.show()
```

The transfer stations appear as large hubs. The ring structure emerges as a circular layout. Peripheral crates cluster at the edges.

### Visualizing Attention Economics

```python
# Context window usage breakdown
labels = ['System', 'History', 'Muscle Memory', 'Improvisation']
sizes = [2000, 5000, 30000, 85000]  # Tokens in 128k window

plt.figure(figsize=(8, 8))
plt.pie(sizes, labels=labels, autopct='%1.1f%%', startangle=90)
plt.title("Context Window Allocation (with Muscle Memory)")
plt.show()
```

Without muscle memory, the pie would show 93% spent on loading source. With muscle memory, 66% is free for improvisation.

### The Ternary Dashboard

A complete monitoring dashboard for a ternary system:

```python
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Top-left: Verification entropy over time
axes[0,0].plot(entropy_history)
axes[0,0].axhline(y=3.0, color='g', linestyle='--', label='Healthy')
axes[0,0].set_title("Verification Entropy")
axes[0,0].legend()

# Top-right: Ternary signal stream
axes[0,1].imshow([signal_stream[-100:]], cmap='RdBu_r', aspect='auto')
axes[0,1].set_title("Last 100 Signals")

# Bottom-left: Consensus heatmap
axes[1,0].imshow(consensus_matrix, cmap='RdBu_r')
axes[1,0].set_title("Agent Consensus")

# Bottom-right: Memory usage
axes[1,1].bar(['FP32', 'Ternary'], [64, 4])
axes[1,1].set_title("Memory Usage (MB)")
axes[1,1].set_ylabel("MB")

plt.tight_layout()
plt.show()
```

This dashboard answers four questions at a glance:
1. Is correctness being preserved? (entropy)
2. Is the system stable? (signal stream)
3. Are agents agreeing? (consensus)
4. Are we saving memory? (usage)

## Connect

- [BENCHMARKING-TERNARY.md](BENCHMARKING-TERNARY.md) — Visualize performance data
- [FLEET-MAP.md](FLEET-MAP.md) — The graph visualization of 303 crates
- [CONSERVATION-LAWS.md](CONSERVATION-LAWS.md) — Visualize entropy and attention metrics
- [CASE-STUDIES.md](CASE-STUDIES.md) — Real-world data to visualize

## Activate

Generate your first ternary visualization:

```python
import openmind
import matplotlib.pyplot as plt

# Load a crate's weight matrix
mm = openmind.MuscleMemory.load("ternary-core_muscles.json")
W = mm.get_weight_matrix("tdot")

# Visualize
plt.imshow(W, cmap='RdBu_r', vmin=-1, vmax=+1)
plt.colorbar(ticks=[-1, 0, +1])
plt.title("tdot Weight Matrix")
plt.savefig("tdot_weights.png")
```

A picture of ternary data is worth 1,024 trits of explanation.
