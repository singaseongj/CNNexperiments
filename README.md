# Neural Classroom Simulator

A browser-based classroom activity where students physically **become neurons** inside a working neural network. Designed for 16–17 students, the simulator teaches how AI learns to identify a traffic light colour (RED or GREEN) through layered signal processing, trust weights, and backpropagation — all done by hand with real dice.

Inspired by [Stage One Education's Human Neural Network](https://stageoneeducation.com/human-neural-network-45.html).

---

## How It Works

The network has three layers. Students are assigned to Layer B or Layer C. Layer A is automated (no students needed).

```
[Layer A — Sensors] → [Layer B — 1st Hidden] → [Layer C — 2nd Hidden] → Output
  (12 auto-nodes)        (10 students)              (6 students)          (teacher)
```

### Layer A — Sensor Nodes (automated)
- Teacher selects RED or GREEN on the traffic light, then clicks **Run**
- Each A node is automatically assigned RED or GREEN based on its reliability weight
- **A1** is always correct (always matches the light)
- **A2** is always wrong (always opposite the light)
- **A3–A12** have fixed reliability weights (w=3–5), giving 50–83% chance of matching the light
- Students discover which nodes are reliable over multiple rounds

### Layer B — 1st Hidden Layer (students)
Each B student:
1. Sees the outputs of their 2–3 connected A nodes on screen
2. Checks their **trust weights** (total = 6) — e.g. trust 4+2 means faces 1–4 follow A node 1, faces 5–6 follow A node 2
3. Rolls a physical die and **clicks the matching face** on screen
4. The simulator maps that face to GREEN or RED based on their trust weights
5. Clicks **📢 Report to Teacher**
6. After evaluation, adjusts trust weights for next round using +/− buttons (range 0–6, total always 6)

### Layer C — 2nd Hidden Layer (students)
Each C student:
1. Sees the outputs of their 2–3 connected B nodes on screen
2. Same trust weight system as Layer B (follows B inputs instead of A)
3. Rolls die, clicks face → outputs GREEN (+1) or RED (−1)
4. Output is multiplied by their **weight multiplier** (starts at 20, adjustable ±5)
5. Clicks **📢 Report to Teacher**

### Output (teacher)
- The simulator sums all C node values: `Σ (output × weight)`
- **Σ ≥ 0 → 🟢 GREEN** · **Σ < 0 → 🔴 RED**
- Round 1 uses raw +1/−1 (no weight multiplier)
- From Round 2 onwards: weighted sum

---

## Running a Session

### Setup
1. Open `index.html` in any modern browser
2. Set the number of A sensors, B students, C students, and starting C weight
3. Click **Begin Class**

### Each Round
| Step | Who | Action |
|------|-----|--------|
| 1 | Teacher | Select traffic light colour → click **▶ Run** |
| 2 | B students | Roll die → click face on screen → Report |
| 3 | C students | Roll die → click face on screen → Report |
| 4 | Teacher | Click **Evaluate →** in output panel |
| 5 | All | Discuss result. Adjust trust weights and C weights for next round |

### Action Bar Buttons
| Button | What it does | One-click? |
|--------|-------------|------------|
| Auto-Roll B | Randomly rolls all unset B nodes | ✓ per round |
| Auto-Roll C | Randomly rolls all unset C nodes | ✓ per round |
| Auto-Report & Adjust Trust | Reports all unreported nodes + saves trust adjustments for next round | ✓ per round |
| New Round ↺ | Resets all dice, applies pending trust, starts next round | — |
| Reset ⟳ | Clears history, restarts from Round 1 (keeps student counts) | — |

### Output Panel (right column)
- **C Values** — live scores from each C node
- **Total Σ** — running sum, colour-coded green/red
- **Evaluate →** — reveals the network's prediction
- **Auto-Adjust Weights** — applies ±5 to each C node weight based on whether it matched the light (saves for next round, available from Round 2)
- **C Node Weights** — bar chart showing each C node's current weight with ±5 buttons

---

## Trust Weight System

Trust weights are the core learning mechanic. Each B and C student splits 6 "trust points" between their inputs:

- **Round 1**: All nodes start at 3+3 (50/50)
- **After each round**: Student adjusts using +/− buttons. e.g. shift to 4+2 → that input gets 4 die faces, the other gets 2
- **Range**: 0–6 (6:0 means fully trust one input and ignore the other)
- **Pending trust**: Adjustments are saved as "next round" values (shown with → arrow). Applied automatically when New Round starts
- **Auto-adjust**: The Auto-Report button calculates which input matched the light and shifts trust +1 toward it

---

## Weight System (Layer C)

C node weights scale the final signal sent to the output:

- **Default**: 20 for all C nodes
- **Round 1**: Raw ±1 used (weight not applied yet)
- **Round 2+**: Output × weight contributes to the sum
- **Manual adjust**: ±5 buttons on each C card (visible after evaluation) and in the output panel
- **Auto-adjust**: Auto-Adjust Weights button applies +5 if C node matched the light, −5 if not
- **Negative weights**: Allowed — a negative weight inverts that node's signal

---

## Connections (Default 12A / 10B / 6C)

Connections are distributed so every node in each layer is watched by at least one node in the next layer:

**B nodes watch A nodes** (sliding window, all 12 A nodes covered):
B1→A1,A2 · B2→A2,A3 · B3→A3,A4 · B4→A5,A6 · B5→A6,A7 · B6→A7,A8 · B7→A8,A9 · B8→A9,A10 · B9→A11,A12 · B10→A12,A1

**C nodes watch B nodes** (all 10 B nodes covered):
C1→B1,B2 · C2→B3,B4 · C3→B4,B5 · C4→B6,B7 · C5→B8,B9 · C6→B9,B10

---

## Customising

All counts are set in the setup screen before class begins:

| Setting | Default | Notes |
|---------|---------|-------|
| Layer A sensors | 12 | Auto-assigned, no students |
| Layer B students | 10 | Becomes 1st hidden layer |
| Layer C students | 6 | Becomes 2nd hidden layer |
| C start weight | 20 | Applied from Round 2 |

The network recalculates all connections automatically when counts change.

---

## Educational Concepts Taught

| Concept | How it appears in the activity |
|---------|-------------------------------|
| Neural network layers | Three physical groups with distinct roles |
| Weighted input | Trust weights split dice faces across inputs |
| Signal propagation | A → B → C → Output flow each round |
| Activation | GREEN/RED determined by die face mapping |
| Backpropagation | Weight + trust adjustments after each round |
| Reliable vs noisy sensors | A node weights hidden from students — discovered over rounds |
| Convergence | Network accuracy improves as weights and trust stabilise |

---

## Files

```
index.html    Single self-contained HTML file — no server, no dependencies
README.md             This file
AGENTS.md             Technical reference for developers and AI assistants
```

Open `index.html` directly in any modern browser. No installation required.
