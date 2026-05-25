# AGENTS.md — Neural Classroom Simulator

Technical reference for developers and AI coding assistants working on `index.html`.

---

## Architecture

The entire application is a **single self-contained HTML file** with no external dependencies beyond Google Fonts. No build step, no server, no npm. Everything — HTML structure, CSS, and JavaScript — lives in `index.html`.

```
index.html
├── <style>          CSS (custom properties, layout, component styles)
├── #setup           Setup screen DOM (shown on load, hidden after Begin)
├── #app             Main app DOM (hidden until Begin Class clicked)
│   ├── .topbar      Sticky header with phase indicator, round counter, target display
│   ├── .cols        4-column grid (Layer A | Layer B | Layer C | Output)
│   └── .abar        Action bar with auto-roll / evaluate / reset buttons
└── <script>         All application logic (no modules, no framework)
```

---

## State Object

All application state lives in a single global object `G`:

```js
const G = {
  aC: 12,          // number of A sensor nodes
  bC: 10,          // number of B student nodes
  cC: 6,           // number of C student nodes
  startW: 20,      // starting weight multiplier for C nodes
  round: 0,        // current round number (0 = not started)
  target: null,    // 'GREEN' | 'RED' — current traffic light colour
  aNodes: [],      // see A node schema below
  bNodes: [],      // see B node schema below
  cNodes: [],      // see C node schema below
  correct: 0,      // cumulative correct predictions
  wrong: 0,        // cumulative wrong predictions
  history: [],     // array of round result objects (newest first)
  phase: 'idle'    // 'idle' | 'rolling' | 'done'
}
```

### A Node Schema
```js
{
  id: Number,         // index (0-based)
  name: String,       // 'A1', 'A2', ...
  colour: null | 'GREEN' | 'RED'   // assigned when teacher clicks Run
}
```

### B Node Schema
```js
{
  id: Number,
  name: String,           // 'B1', 'B2', ...
  student: String,        // 'Student 1', ...
  aConn: Number[],        // indices into G.aNodes (2–3 connections)
  trust: Number[],        // trust weights per aConn input, sum = 6, e.g. [3,3]
  pendingTrust: null | Number[],  // trust to apply at start of next round
  weight: Number,         // B node weight multiplier (starts 1, adjustable)
  diceRolled: null | Number,      // 1–6, face student selected
  output: null | 'GREEN' | 'RED', // result of dice roll given trust weights
  reported: Boolean       // true after student clicks Report
}
```

### C Node Schema
```js
{
  id: Number,
  name: String,           // 'C1', 'C2', ...
  student: String,        // 'Student 11', ...
  bConn: Number[],        // indices into G.bNodes (2–3 connections)
  trust: Number[],        // trust weights per bConn input, sum = 6
  pendingTrust: null | Number[],
  weight: Number,         // multiplier for output (starts startW=20)
  bWeights: Number[],     // legacy — not used in current version
  diceRolled: null | Number,
  output: null | 'GREEN' | 'RED',
  numeric: null | 1 | -1, // GREEN=+1, RED=-1
  value: null | Number,   // numeric × weight
  reported: Boolean,
  effect: null | 'reported',
  sugW: undefined | Number  // suggested weight for next round (from auto-adjust)
}
```

---

## Key Functions

### Setup
| Function | Description |
|----------|-------------|
| `adj(k, d)` | Adjusts setup values (`'a'`, `'b'`, `'c'`, `'w'`) by delta `d` |
| `startSim()` | Initialises all node arrays, hides setup, shows app, calls `newRound()` |
| `buildConns(total, count)` | Distributes `total` inputs evenly across `count` nodes using sliding window. Returns `Number[][]`. Ensures every input is covered. |

### Round Management
| Function | Description |
|----------|-------------|
| `newRound()` | Increments `G.round`, applies `pendingTrust` to all nodes, resets dice/output/reported, resets buttons, calls all render functions |
| `selLight(colour)` | Sets `_light` and `G.target`, updates traffic light UI |
| `runLight()` | Assigns colours to all A nodes based on `_light` and reliability weights, then calls `renderB()` and `renderC()` |
| `evalRound()` | Computes sum, determines prediction, updates verdict/orb/history, sets effects, calls render functions. Round 1 uses raw ±1; Round 2+ uses ±1 × weight. |
| `resetSim()` | Clears history/scores, resets round to 0, calls `newRound()` |
| `goSetup()` | Shows setup screen, hides app |

### Layer A
| Function | Description |
|----------|-------------|
| `renderA()` | Renders all A node cards in `#a-grid`. A1 always correct, A2 always wrong, A3+ use `AW` weights. Does not show reliability info to students. |

### Layer B
| Function | Description |
|----------|-------------|
| `renderB()` | Renders all B node cards in `#b-grid`. Builds input badges, dice faces (with letter hints), output display, report button, trust adjustment panel. |
| `rollB(bi, face)` | Records `diceRolled` and computes `output` via `faceColourMulti`. Calls `renderB`, `renderC`, `updateProg`, `canEval`, `checkAutoTrust`. |
| `reportB(i)` | Sets `n.reported = true`, re-renders, updates progress |
| `autoRollB()` | Randomly rolls all unset B nodes, then calls `renderB`, `renderC`, `updateProg`, `canEval`, `checkAutoTrust`. Disables itself after click. |
| `adjBTrust(bi, idx, d)` | Adjusts `pendingTrust[idx]` by `d`, balancing from adjacent index. Range 0–6, total must stay 6. |
| `adjBWeight(i, d)` | Adjusts `G.bNodes[i].weight` by `d` (no floor/ceiling). |

### Layer C
| Function | Description |
|----------|-------------|
| `renderC()` | Renders all C node cards in `#c-grid`. Same structure as B cards but uses `bConn` and shows weight multiplier. |
| `rollC(ci, face)` | Records dice, computes `output`, `numeric`, `value`. Calls renders. |
| `reportC(i)` | Sets reported/effect, re-renders |
| `autoRollC()` | Randomly rolls all unset C nodes, auto-reports them |
| `adjCTrust(ci, idx, d)` | Same as `adjBTrust` but for C nodes |
| `adjCWeight(i, d)` | Adjusts weight, recalculates `value` if already rolled, updates output panel |

### Auto-Actions
| Function | Description |
|----------|-------------|
| `autoAdjustTrust()` | Auto-rolls/reports unreported B and C nodes, then saves `pendingTrust` shifting +1 toward whichever input matched `G.target`. One-click per round. |
| `autoEval()` | Adjusts all C node weights ±5 based on whether `n.output === G.target`. Saves as `sugW`. Does NOT call `evalRound()`. One-click per round. |

### Utilities
| Function | Description |
|----------|-------------|
| `faceColour(face, colour0, colour1, t0)` | Legacy 2-input version |
| `faceColourMulti(face, colours, trust)` | Maps face 1–6 to a colour using cumulative trust array. Face falls in bucket `k` if it's within the cumulative sum of trust[0..k]. |
| `canEval()` | Enables `#btn-eval` if all C nodes reported. Also enables `#btn-autoeval` from round 2+. |
| `checkAutoTrust()` | Enables `#btn-autotrust` if all B dice rolled and any node unreported |
| `updateProg()` | Updates B and C progress bar/text (reported count / total) |
| `renderScores()` | Updates C scores list in output panel |
| `renderWeightPanel()` | Updates C node weight bar chart in output panel |
| `renderHist()` | Updates round history list (newest first, max 6 shown) |
| `setPhase(cls, txt)` | Updates phase pill class and text |
| `setInfo(text)` | Updates action bar info text |

---

## A Node Reliability Weights

```js
const AW = [5,4,3,4,3, 5,4,3,4,3, 5,4,3,4,3, 5,4,3,4,3];
// index:  0 1 2 3 4  5 6 7 8 9  10 ...
```

- **Index 0 (A1)**: Always outputs `G.target` (100% correct, hardcoded)
- **Index 1 (A2)**: Always outputs opposite of `G.target` (0% correct, hardcoded)
- **Index 2+ (A3+)**: `Math.random() * 6 < AW[i] ? G.target : opposite`
  - w=5 → 83% correct · w=4 → 67% · w=3 → 50%

---

## Trust Weight Mechanics

Each B/C node has a `trust` array (length = number of connections, sum = 6).

```
trust = [4, 2]   →  faces 1–4 follow input 0, faces 5–6 follow input 1
trust = [6, 0]   →  all 6 faces follow input 0 (fully committed)
trust = [3, 3]   →  50/50 (default starting value)
```

`faceColourMulti(face, colours, trust)`:
```js
function faceColourMulti(face, colours, trust) {
  var cumulative = 0;
  for (var k = 0; k < trust.length; k++) {
    cumulative += trust[k] || 0;
    if (face <= cumulative) return colours[k] || null;
  }
  return colours[colours.length - 1] || null;
}
```

**Pending trust**: Adjustments made during a round are stored in `n.pendingTrust`. Applied in `newRound()` via:
```js
if (n.pendingTrust) { n.trust = n.pendingTrust.slice(); n.pendingTrust = null; }
```

---

## Connection Algorithm

`buildConns(total, count)` distributes `total` inputs across `count` nodes:

```js
function buildConns(total, count) {
  var conns = [], step = total / count;
  for (var i = 0; i < count; i++) {
    var start = Math.round(i * step);
    var end = Math.round((i + 1) * step);
    if (end <= start) end = start + 1;
    var conn = [];
    for (var j = start; j < end; j++) conn.push(j % total);
    if (conn.length < 2) conn.push((start + 1) % total);
    conns.push(conn);
  }
  return conns;
}
```

All inputs are covered (no node left unwatched). With 12A/10B: each B node gets 1–2 A connections covering all 12 A nodes. With 10B/6C: each C node gets 1–3 B connections covering all 10 B nodes.

---

## Sum Calculation

```js
var sum = G.cNodes.reduce(function(a, n) {
  if (!n.numeric) return a;
  return a + (G.round === 1 ? n.numeric : n.numeric * n.weight);
}, 0);

var pred = sum >= 0 ? 'GREEN' : 'RED';
```

- Round 1: raw ±1 sum (no weight applied — weight 20 is introduced from round 2)
- Round 2+: weighted sum

---

## Button State Rules

| Button ID | Enabled when | Disabled after |
|-----------|-------------|----------------|
| `#run-btn` | Light selected | — (re-enabled each round) |
| `#btn-autorollb` | Always at round start | After one click |
| `#btn-autorollc` | Always at round start | After one click |
| `#btn-autotrust` | All B dice rolled + any unreported | After one click |
| `#btn-eval` | All C nodes reported | After evalRound() |
| `#btn-autoeval` | Always at round start (Round 2+) | After one click |

All buttons reset at `newRound()`. `#btn-autoeval` hidden in round 1.

---

## CSS Architecture

Custom properties in `:root` for all colours, shadows, and radii. BEM-ish class naming.

Key layout classes:
- `#app` — `display:flex; flex-direction:column; height:100vh; overflow:hidden`
- `.cols` — `display:grid; grid-template-columns:1fr 1fr 1fr 260px`
- `.col` — `display:flex; flex-direction:column; overflow:hidden`
- `.col-hd` — `flex-shrink:0; max-height:40%; overflow-y:auto` (header, never scrolls away)
- `.col-body` — `flex:1; overflow-y:auto` (cards, scrolls independently)

Each column scrolls its body independently. The 4th column (output) has `background:var(--bg2)` to visually distinguish it.

---

## Common Pitfalls

1. **`rollB` must use `n.aConn`**, not `n.bConn` — B nodes connect to A nodes
2. **Template literals with nested quotes** — the JS uses string concatenation (`+'...'`) not template literals for HTML building, to avoid browser `innerHTML` parsing issues
3. **`pendingTrust` vs `trust`** — always read `n.pendingTrust || n.trust` when showing next-round values; only read `n.trust` for current-round dice mapping
4. **Round 1 weight exception** — `evalRound` branches on `G.round === 1` for raw sum
5. **`overflow:visible` on `.n-card`** — required so dice face hint labels (`.d-hint`) don't get clipped
6. **One-click buttons** — auto-roll/auto-report/auto-eval buttons self-disable. They're re-enabled in `newRound()` via `btnIds.forEach(...)`

---

## Modifying Node Counts

To change default node counts, update both the JS setup vars and the HTML setup card display values:

```js
// JS (near top of <script>)
let _a=12, _b=10, _c=6, _w=20;

// HTML setup cards
<div class="su-val" id="sa-v">12</div>
<span class="step-num" id="sa-n">12</span>
// ... etc
```

---

## Adding a New Layer

To add a 4th layer (e.g. Layer D):
1. Add `dNodes: []` to `G`
2. Add `buildConns(G.cC, G.dC)` in `startSim()`
3. Add a new column to `.cols` and update `grid-template-columns`
4. Implement `renderD()`, `rollD()`, `reportD()` following the C node pattern
5. Update `evalRound()` to sum D node values instead of C
6. Update `canEval()` to check D nodes reported
