---
name: dsa-dryrun-visualizer
description: Build a single self-contained HTML file that interactively dry-runs ANY DSA problem step-by-step with Hinglish (Hindi+English) teacher-voice explanations. PRIMARY TRIGGER is the slash command `/dsa-dryrun-visualizer`. The visualizer shows EVERY code line's EVERY execution with Hinglish logs explaining WHY each step happens AND a real-life analogy. Produces a clean 4-section HTML (problem | code | visualization | log preview) with step controls.
version: 2.5
last_updated: 2026-06-13
---

# DSA Dry-Run HTML Visualizer — The Definitive Skill

Single self-contained HTML file that dry-runs ANY DSA algorithm step-by-step with Hinglish explanations + real-life analogies.

---

## 📑 Table of Contents

1. [When to trigger](#-when-to-trigger)
2. [🔴 CRITICAL RULES #1–#10](#-critical-rules)
3. [File naming convention](#-file-naming-convention)
4. [Output format](#-output-format)
5. [Workflow](#-workflow)
6. [Layout architecture](#-layout-architecture)
7. [SECTION 1 — Problem Card](#-section-1--problem-card)
8. [SECTION 2 — Python Code](#-section-2--python-code-panel)
9. [SECTION 3 — Visualization (2/3 width)](#-section-3--visualization-panel-23-width)
10. [LOG PREVIEW — Floating Popup](#-log-preview--floating-popup-bottom-right)
11. [Enhanced Log Card Schema v2 (richLog)](#-enhanced-log-card-schema-v2-richlog)
12. [Footer Controls](#-footer-controls)
13. [JS patterns & buildSteps](#-js-patterns)
14. [CSS design system](#-css-design-system)
15. [Action types & colors](#-action-types--colors)
16. [Status pills](#-status-pills)
17. [Hinglish writing style](#-hinglish-writing-style)
18. [Pre-delivery master checklist](#-pre-delivery-master-checklist)
19. [Emoji & Symbol Fix Guide](#-emoji--symbol-fix-guide)
20. [Reference implementations](#-reference-implementations)

---

## 🎯 When to trigger

> **🔥 PRIMARY TRIGGER: `/dsa-dryrun-visualizer`**

**Slash command variants:** `/dsa-visualizer`, `/dsa-dryrun`, `/dryrun`, `/dsa`

**Other triggers:** "dry run", "step-by-step visualizer", "visualize this algorithm", "trace through", "interactive walkthrough", user pastes code + asks "how does this work step by step"

- [ ] When slash command appears → invoke immediately, no clarifying questions
- [ ] Only acceptable question: missing problem details (which problem? which test cases?)

---

## 🔴🔴🔴 CRITICAL RULES

### 🔴 RULE #1 — Slash command = immediate build

- [ ] `/dsa-dryrun-visualizer` → start building immediately
- [ ] Do NOT ask "do you want a visualizer?" — the command IS the answer

### 🔴 RULE #2 — SHOW EVERY CODE LINE'S EVERY EXECUTION

> **THE most important rule.** A dry-run visualizer must show EVERY time the Python interpreter touches a line.

| Construct | Steps required |
|---|---|
| `for x in range(a, b):` | One step per iteration PLUS one EXIT step when range exhausted. Empty range → SKIP step |
| `while condition:` | One step per check — TRUE entry AND FALSE exit. Even first-check failure |
| `if condition:` | Condition-check step (TRUE/FALSE verdict). Body ONLY if TRUE |
| `elif` / `else:` | Each gets its own step |
| `break` / `continue` | Own dedicated step BEFORE loop exits/jumps |
| `return` | Always shown as its own step |
| Recursion call | TWO steps: call site + function entry |
| Recursion return | Step showing return back to parent |
| Base case | Own step with "BASE CASE!" badge |
| Assignment / mutation | Step showing variable change |

**Anti-patterns — NEVER do:**

- [ ] ❌ Skipping condition-FALSE steps
- [ ] ❌ Merging loop-exit into previous step
- [ ] ❌ Combining multiple code lines into one step (L2-4 is WRONG — each line = own step)
- [ ] ❌ Log/why/analogy text in pure English — ALL THREE MUST be Hinglish teacher-voice
- [ ] ❌ Generic emoji like 🔵 — match emoji to action semantics

### 🔴 RULE #3 — Syntax highlighter uses control-char tokenizer

```js
function renderCode() {
  var p = document.getElementById('codePanel');
  p.innerHTML = codeLines.map(function(raw, i) {
    var h = raw;
    h = h.replace(/("[^"]*"|'[^']*')/g, '\x01STR\x02$1\x03');
    h = h.replace(/\b(def|for|in|if|return|break|continue|and|else|elif|True|False|None|not|or|while|lambda)\b/g, '\x01KW\x02$1\x03');
    h = h.replace(/\b(len|range|int|str|list|append|pop|sorted|min|max|sum|abs|enumerate|zip|print|set|add)\b/g, '\x01FN\x02$1\x03');
    h = h.replace(/(?<![.\w"\x01\x02\x03])(-?\d+)\b/g, '\x01NUM\x02$1\x03');
    h = h.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
    h = h.replace(/\x01([A-Z]+)\x02/g, function(_, c) { return '<span class="' + c.toLowerCase() + '">'; }).replace(/\x03/g, '</span>');
    return '<div class="code-line" data-line="' + i + '"><span class="ln">' + (i+1) + '</span><span class="code-content">' + h + '</span></div>';
  }).join('');
}
```

- [ ] Tokenize with `\x01`, `\x02`, `\x03` BEFORE HTML escape
- [ ] Strings tokenized FIRST (both `"..."` AND `'...'`)
- [ ] Span conversion AFTER HTML escape
- [ ] Provide mini Python tokenizer `hlCode(raw)` for log card code snippets (uses `cs/ck/cf/cn/cc` classes for dark theme)

### 🔴 RULE #4 — Tree node spacing minimums

| Param | Inline (≥) | Fullscreen (≥) |
|---|---|---|
| Node radius `R` | **6** | **11** |
| Horizontal gap `gapX` | **26** | **48** |
| Vertical gap `gapY` | **26** | **44** |

### 🔴 RULE #5 — Render EMPTY data structures IMMEDIATELY

When code creates a data structure (`dp = [[0]*n]`, `graph = [[] for _]`, `stack = []`), the visualization MUST show it in empty/initial state in the SAME step.

- [ ] ❌ renderViz returns placeholder for init steps (only L0 func-def may)
- [ ] ❌ Data structures only appear when first data is written
- [ ] ✅ renderViz ALWAYS draws all data structures from snapshot state

### 🔴 RULE #6 — NEVER use HTML entities — use actual Unicode chars

- [ ] `▶` not `&#x25B6;` — direct Unicode only
- [ ] Applies to ALL: buttons, badges, kbd hints, JS strings, headings

### 🔴 RULE #7 — system-ui font fallback on symbol elements

- [ ] `.btn`, `.log-badge`, `.action-badge`, `.section-label`, `kbd`, `.log-status-pill`, `.log-section-title-icon` → include `system-ui, "Segoe UI Emoji"` in font-family
- [ ] Never use `▴`/`▾` (use `▲`/`▼`), never use `⟲` (use `↺`)


### 🆕 🔴 RULE #8 — Every log card MUST follow the 6-section schema

Every step's log popup MUST contain these sections (where applicable):

1. **Header** — badge + line ref + status pill
2. **📜 Code Executing** — actual highlighted code snippet
3. **📥 Input Values** — KV pills of variables this line READS
4. **📤 Output / State After Line** — KV pills of variables this line WROTE (with `changed:true` flag)
5. **⚖️ Verdict / Decision** — for if/for/while only — boolean evaluation result
6. **💡 Why This Happens** — Hinglish technical explanation
7. **🎭 Real-Life Analogy** — Hinglish relatable metaphor

See [Section 11](#-enhanced-log-card-schema-v2-richlog) for full schema.

### 🆕 🔴 RULE #9 — File name header at top of every response

Every response delivering a visualizer MUST start with:

```
## 📁 File name: lc<NNN>-<kebab-case-name>.html
```

Followed by a single fenced HTML code block. See [File naming convention](#-file-naming-convention).

### 🆕🔴🔴🔴 RULE #10 — ZERO-MISS LOGGING (SABSE IMPORTANT RULE)

> ⚠️ **THIS IS THE #1 MOST IMPORTANT RULE IN THE ENTIRE SKILL.** Every single line of code, every single execution — success ya fail ya skip ya loop ya kuch bhi ho — MUST have a complete, detailed log. **EK BHI LINE MISS NAHI HONI CHAHIYE.**

#### ✅ Master Checkboxes — EVERY log entry MUST have ALL of these:

- [ ] **📜 ACTUAL CODE** — Jo line execute ho rahi hai uska actual Python code with **concrete values substituted** (e.g. `for i in range(2, 5): i=3`, NOT just `for loop iteration`)
- [ ] **📥 INPUT VALUES** — Line execute hone se PEHLE ki variable states (e.g. `index=2, n=5, num="12345"`)
- [ ] **📤 OUTPUT / RESULT** — Line execute hone ke BAAD kya change hua (e.g. `curr_str="34"`, `stack.size=3`, `visited.add((1,2))`)
- [ ] **⚖️ VERDICT** — Condition TRUE hua ya FALSE? Loop continue ya exit? Match mila ya nahi? (e.g. `if 6 == 6 → True ✓`, `if 3 > 5 → False ✗`)
- [ ] **💡 WHY (Hinglish)** — KYUN yeh hua? Technical reason in Hinglish teacher voice (e.g. `"Value target ke barabar hai, toh yeh valid expression hai!"`)
- [ ] **🎭 ANALOGY (Hinglish)** — Real-life analogy in Hinglish (e.g. `"Jaise exam mein passing marks aa gaye — result board pe naam lag gaya!"`)
- [ ] **🚦 STATUS PILL** — `success` / `fail` / `skip` / `loop` / `prune` / `info` — har step ka status MUST be visible

#### ✅ EVERY execution type MUST have a log — NO EXCEPTIONS:

- [ ] ✅ **Line SUCCESS hone pe** — code + values + kya output aaya + Hinglish "Mil gaya!" explanation
- [ ] ❌ **Line FAIL hone pe** — code + values + kyun fail hua + Hinglish "Yeh kaam nahi aaya kyunki..." explanation
- [ ] ⏭ **Line SKIP hone pe** — code + values + kyun skip hua + Hinglish "Yeh branch lene ki zaroorat nahi..." explanation
- [ ] 🔁 **Loop ITERATION pe** — code + current i value + range + Hinglish "Loop ka X-th chakkar..."
- [ ] 🔁 **Loop EXIT pe** — code + final i value + kyun exit hua + Hinglish "Loop khatam — saare options try ho gaye"
- [ ] 🔎 **IF condition TRUE pe** — code + both sides of condition + `→ True` + Hinglish "Condition match hua!"
- [ ] 🔎 **IF condition FALSE pe** — code + both sides of condition + `→ False` + Hinglish "Condition fail — else ya skip"
- [ ] 📞 **Function CALL pe** — code + all arguments with values + Hinglish "Naya function call..."
- [ ] 🏆 **Function RETURN pe** — code + return value + Hinglish "Kaam ho gaya, wapas aa rahe hain"
- [ ] ✂️ **BREAK / PRUNE pe** — code + kyun break hua + Hinglish "Yahan se aage jaana bekaar — katna padega"
- [ ] ⚙️ **Variable ASSIGNMENT pe** — code + purani value → nayi value + Hinglish "Update ho gaya"
- [ ] 🎉 **RESULT FOUND pe** — code + found value + Hinglish "🎉 Mil gaya bhai! Board pe likho!"
- [ ] 🏁 **BASE CASE pe** — code + condition met + Hinglish "Recursion ka base — ab wapas jaao"

#### ✅ Log format — ACTUAL CODE WITH VALUES (not vague descriptions):

```
❌ WRONG (vague):
   log: "checking loop condition"
   log: "base case check"
   log: "variable updated"

✅ CORRECT (code + values + verdict):
   log: "L9 — for i in range(2, 5): i=3"
   log: "L5 — if index==3 == n==3 → True (base case HIT!)"
   log: "L12 — curr_str = num[1:3] = \"23\""
   log: "L6 — if value==15 == target==6 → False (match nahi hua)"
   log: "L10 — if 2 > 0 and num[0]==='0' → True (leading zero! PRUNE!)"
   log: "L18 — backtrack(3, \"1+2*3\", 1-2+2*3=7, 2*3=6) [* branch]"
```

#### ✅ Hinglish quality — MUST be teacher-voice, NOT robotic:

```
❌ WRONG (robotic English):
   why: "The condition evaluated to true."
   why: "Variable has been updated."
   analogy: "This is like a comparison."

✅ CORRECT (Hinglish teacher):
   why: "Dekho, value=6 aur target=6 — dono barabar hain! Matlab yeh expression sahi hai."
   why: "Leading zero detect hua — '05' type number allowed nahi. Toh loop break karna padega."
   analogy: "Jaise exam paper mein answer match ho gaya — tick laga do aur aage badho!"
   analogy: "Bank cheque pe '0123' likhna galat hai — leading zero reject! Cheque wapas."
```

#### ✅ ZERO-MISS verification checklist (before delivery):

- [ ] Open the generated HTML → step through ALL steps → verify **every code line** has a log
- [ ] Check: does `for` loop show **EACH iteration** AND the **exit step**?
- [ ] Check: does `if` show the **condition check** even when it's **FALSE**?
- [ ] Check: does `return` show its **own separate step**?
- [ ] Check: does every log show the **actual Python code** with **real values**?
- [ ] Check: does every log have **Hinglish why** (not English)?
- [ ] Check: does every log have **Hinglish analogy** (not English)?
- [ ] Check: is there a **status pill** on every log card?
- [ ] Check: are **input KV pills** shown (what the line reads)?
- [ ] Check: are **output KV pills** shown (what the line changes)?
- [ ] Total steps count ≈ expected (e.g. 3-digit backtracking ≈ 100-200 steps, not 20)

---

## 🆕 📁 File naming convention

Every response delivering a visualizer must follow this header format:

```
## 📁 File name: lc<NUMBER>-<kebab-case-name>.html
```

### Naming patterns

| Pattern | Example |
|---|---|
| `lc<number>-<kebab-case-name>.html` | `lc282-expression-add-operators.html` |
| `lc<number>-<short>.html` | `lc1568-min-days-disconnect-island.html` |
| `lc<number>.html` (fallback) | `lc282.html` |

### Naming rules

- [ ] All lowercase
- [ ] Hyphens between words (kebab-case)
- [ ] No spaces, no underscores
- [ ] Max ~50 chars including extension
- [ ] Use the official LeetCode problem slug when possible

---

## 🆕 📤 Output format

Every response delivering a visualizer must follow this order:

1. ✅ Static note (Sidekick code-generation rule)
2. ✅ File name header (`## 📁 File name: lcXXX-name.html`)
3. ✅ Single fenced HTML code block (complete, self-contained)
4. ✅ Brief recap of what's covered (algorithm steps, viz components, test cases)
5. ✅ Follow-up offer (variant ideas, enhancements, etc.)

### Critical rules

- [ ] ⚠️ NO external dependencies except Google Fonts
- [ ] ⚠️ Single `<html>` file — opens directly in browser, no build step
- [ ] ⚠️ Always include `` as first line
- [ ] ⚠️ Always prepend the static note before any code response

---

## 🚀 Workflow

- [ ] 1. Gather problem details — name, LC number, Python solution, 3–5 test cases
- [ ] 2. Verify Python solution works on all test cases (mental trace or bash)
- [ ] 3. Generate HTML following every checklist
- [ ] 4. Verify buildSteps covers every code line execution
- [ ] 5. Verify every step produces a `richLog` with 6 sections (where applicable)
- [ ] 6. Deliver response with file name header + code block + recap

---

## 🏗️ Layout Architecture

**Problem Card (top, collapsible) + 2-column main (code 1/3 | viz 2/3) + Split Footer. Log Preview = floating popup (right corner). Light theme only (Claude colors).**

```
┌─────────────────────────────────────────────────────────────┐
│ [GT LEARN]  #282 Expression Add Operators  HARD  Backtrack │
│             num=[123] target=[6]              [exp:[...]]   │
│                                                    [▲ Hide] │
├──────────────────────────────────┬──────────────────────────┤
│ Problem statement (left)         │ Test cases (right)       │
│ - detailed explanation           │ T1 "123" → 6 (basic)     │
│ - core concepts & base case      │ T2 "232" → 8 (precedence)│
│ (NO code logic here)             │ [▶ Run]                  │
├──────────────────┬───────────────┴──────────────────────────┤
│ 📜 Python Code   │ 📊 Visualization (2/3 width)             │
│ (1/3 width)      │                                          │
│                  │ [vars strip: index=2, expr=12+3, val=15] │
│ 1  def func():   │                                          │
│ 2    res=[]      │  Tile row: [1][2][3]                     │
│ 3    n=len(num)  │                                          │
│ [active line]    │  Stack | Result chips                    │
│                  │                          ┌────────────┐  │
│                  │                          │📋 Log v2   │  │
│                  │                          │6-section   │  │
│                  │                          └────────────┘  │
├──────────────────┼──────────────────────────────────────────┤
│ 3/42 [FOR]       │  ←→ [↺][◀][▶ Next] | [📋ON][−]100%[+] [⛶]│
└──────────────────┴──────────────────────────────────────────┘
```

### Layout CSS (light theme only)

```css
.app-shell { display:flex; flex-direction:column; height:100vh; overflow:hidden; background:#FAFAF8 }
.problem-card { flex-shrink:0; border-bottom:1px solid rgba(0,0,0,.11) }
.main-row { flex:1; display:flex; overflow:hidden; min-height:0 }
.col { display:flex; flex-direction:column; overflow:hidden; min-width:0 }
.col-code { width:33%; min-width:140px; max-width:45vw }
.col-viz { flex:1; min-width:0 }
.footer-bar { display:flex; flex-shrink:0; border-top:1px solid rgba(0,0,0,.11); background:#FFFFFF }
.footer-left { width:33%; min-width:140px; max-width:45vw; display:flex; align-items:center; gap:8px; padding:7px 12px; border-right:1px solid rgba(0,0,0,.06) }
.footer-right { flex:1; display:flex; align-items:center; gap:8px; padding:7px 20px }
```

**Resize handle (1 only):** `resizeH1` between code and viz. Footer-left width syncs with code column via `syncFooterWidth()`.


---

## 🔹 SECTION 1 — Problem Card

Top section. Collapsible with ▲/▼ toggle. Has **GT LEARN** home button at top-left.

### Header row pattern

```html
<header class="prob-header">
  <div class="prob-header-left">
    <button class="gt-home-btn" type="button" onclick="location.reload()">GT LEARN</button>
    <span class="prob-num">#282</span>
    <span class="prob-name">🎨 Expression Add Operators</span>
    <span class="prob-diff--hard">HARD</span>
    <span class="prob-tag">Backtracking</span>
  </div>
  <div class="prob-header-right">
    <span class="top-label">num=</span>
    <input id="inputNum" class="input" value="123">
    <span class="top-label">target=</span>
    <input id="inputTarget" class="input" value="6">
    <span id="expectedBadge" class="top-expected">exp: ["1+2+3","1*2*3"]</span>
    <button id="probToggleBtn" class="prob-toggle-btn" type="button" onclick="toggleProblem()">▲ Hide</button>
  </div>
</header>
```

### Body — Two columns

**Left column (problem explanation — NO code logic):**

- [ ] 📝 Problem Statement — 3–5 sentences, use `<code>` for params, `<strong>` for constraints
- [ ] 🧠 Solving Approach — 2–3 sentences explaining strategy
- [ ] 🎯 Core Concepts & Base Case — `.concept-block` cards (Base Case / Transition / Pruning)
- [ ] Complexity chips — Time + Space Big-O

**Right column (test cases + run):**

- [ ] 3–5 test case chips — full input + descriptive `.pt-tag` + expected
- [ ] **🔴 MANDATORY: Each chip MUST display the ACTUAL INPUT VALUES** (real numbers, real strings, real arrays) — never use abstract labels like `"mixed"`, `"all valid"`, `"basic"` as the input display. Use a two-row chip: top row has tag + label + expected; bottom row shows the actual values.
- [ ] For list/array inputs: show each item as a separate `<code>` pill inside a `.tc-vals` row
- [ ] Live output — algorithm result after running
- [ ] ▶ Run button — calls `resetWithInput()`

---

## 🔹 SECTION 2 — Python Code Panel

Left column (1/3 width). Syntax-highlighted with active line tracking.

- [ ] Default `width:33%; min-width:140px; max-width:45vw`
- [ ] `fitCodeWidth()` auto-fits, capped at `Math.min(offsetWidth, window.innerWidth * 0.45)`
- [ ] Guard with `codeWidthUserSet` flag
- [ ] Renders from `codeLines = [...]` array
- [ ] Each line: `<div class="code-line" data-line="N">` with `<span class="ln">` + `<span class="code-content">`
- [ ] RULE #3 tokenizer (`.kw`, `.fn`, `.num`, `.str`)
- [ ] `.active` class on currently executing line + `scrollIntoView({block:'nearest'})`
- [ ] `overflow-y:auto`, line-height `1.9`

---

## 🔹 SECTION 3 — Visualization Panel (2/3 width)

> 🔴 **9 CRITICAL VISUALIZATION RULES (V1–V9)**

| Rule | Summary |
|---|---|
| **V1** | NO duplicate data display (vars strip is single source of truth) |
| **V2** | Graphical diagrams with area/shape coverage (not just colored boxes) |
| **V3** | Diagrams SMALL by default (`max-height:180px`), zoom to enlarge |
| **V4** | Show ALL data structures from code (mandatory, even when empty) |
| **V5** | Best emoji/signs per problem domain (water=🌊, coins=🪙, tree=🌳) |
| **V6** | Multiple viz-cards side by side (`.viz-row` flex layout) |
| **V7** | Stack/Queue/Set as proper visual containers (column-reverse, chips, empty states) |
| **V8** | Footer-left syncs with code column width (init + drag + window resize) |
| **V9** | Grid/Matrix problems MUST have SVG flow-arrow overlay (traversal path **or** DP recurrence/dependency arrows for current cell) |

### Visual design — state color system

| State | Border | Background | Animation | Emoji |
|---|---|---|---|---|
| **Current/Active** | `--accent-indigo` | `--accent-indigo-bg` | `pulse-node` | 📍 |
| **Done/Confirmed** | `--accent-green` | `--accent-green-bg` | none | ✅ |
| **Visiting/In-progress** | `--accent-amber` | `--accent-amber-bg` | subtle glow | 🟡 |
| **Failed/Pruned** | `--accent-red` | `--accent-red-bg` | `shake` | ❌ |
| **Unvisited/Future** | `--border-color` | `--bg-secondary` | dim opacity .5 | ⬜ |
| **Win/Solution path** | gold | gold-bg | `win-pulse` | ⭐ |

### Required CSS animations

```css
@keyframes pulse-node { 0%,100% {box-shadow:0 0 4px var(--accent-indigo)} 50% {box-shadow:0 0 12px var(--accent-indigo)} }
@keyframes shake { 0%,100% {transform:translateX(0)} 25% {transform:translateX(-3px)} 75% {transform:translateX(3px)} }
@keyframes win-pulse { 0%,100% {box-shadow:0 0 3px rgba(234,179,8,.5)} 50% {box-shadow:0 0 8px rgba(234,179,8,.9)} }
```

### Variables strip (single source of truth — V1)

- [ ] `.vars-strip` — `display:flex; gap:6px; flex-wrap:wrap; margin-bottom:8px`
- [ ] `.var-chip` — pills with name + value
- [ ] Contextual classes: `.vc-active` (indigo), `.vc-safe` (green), `.vc-flip` (red)
- [ ] **No separate tracker/summary div** — vars strip IS the tracker

### viz-row layout (V6)

```html
<div class="viz-row">
  <div class="viz-card"><div class="viz-title">🧱 Board</div>...</div>
  <div class="viz-card"><div class="viz-title">🗂️ Stack</div>...</div>
  <div class="viz-card"><div class="viz-title">🗺️ Visited</div>...</div>
</div>
```

```css
.viz-row { display:flex; gap:12px; align-items:flex-start; flex-wrap:wrap }
.viz-card { background:var(--bg-card); border:1px solid var(--border-color); border-radius:var(--radius-md); padding:10px }
```

### Stack/Queue/Set patterns (V7)

```css
.stack-col { display:flex; flex-direction:column-reverse; gap:4px; min-height:124px; padding:8px; background:var(--bg-secondary); border-radius:var(--radius-md); border:1px dashed var(--border-color) }
.chips { display:flex; gap:4px; flex-wrap:wrap }
.chip { font:10px var(--font-mono); padding:3px 6px; border-radius:99px; background:var(--bg-secondary); border:1px solid var(--border-color) }
.empty-state { font-size:11px; color:var(--text-tertiary); font-style:italic }
```

### 🆕 Grid/Matrix Flow Arrows — SVG Overlay (V9)

> **For ALL grid/matrix traversal problems** (diagonal traverse, DFS/BFS on grid, spiral, etc.) — the grid MUST have animated SVG arrows overlaid inside it showing the traversal path step by step.

#### What it does

- Draws animated dashed arrows **between consecutive visited cells** as the algorithm steps forward
- Uses **3 arrow colors** to distinguish move types:
  - 🔴 **Red dashed** — normal traversal within the current phase/diagonal
  - 🟣 **Purple dashed** — jump between phases (e.g., diagonal switch, BFS level boundary)
  - 🟡 **Amber solid** — the *next planned move* (lookahead to `currCell`)
- Arrow **shortens from both ends** so it doesn't overlap the cell border
- All arrows carry `<marker>` arrowheads matching their color

#### Required CSS animations

```css
@keyframes flow-dash { to { stroke-dashoffset: -18 } }
@keyframes flow-pulse { 0%,100% { opacity: .85 } 50% { opacity: 1 } }
```

#### CSS classes

```css
/* SVG sits absolutely over the matrix grid; PAD must match matrix-wrap padding */
.flow-svg { position:absolute; top:14px; left:14px; pointer-events:none; z-index:6; overflow:visible }

/* Normal traversal arrow — animated dashed red */
.flow-line {
  stroke:#dc2626; stroke-width:2.5; stroke-dasharray:5,4; fill:none;
  animation: flow-dash 1s linear infinite, flow-pulse 1.4s ease-in-out infinite;
  filter: drop-shadow(0 0 2px rgba(220,38,38,.5));
}
/* Next-move arrow — thicker amber, faster dash */
.flow-line--current {
  stroke:#f59e0b; stroke-width:3; stroke-dasharray:4,3;
  animation: flow-dash .6s linear infinite;
  filter: drop-shadow(0 0 3px rgba(245,158,11,.7));
}
/* Phase-jump arrow — purple, static (no animation) */
.flow-line--diag-jump { stroke:#8b5cf6; stroke-width:2.2; stroke-dasharray:2,3; opacity:.85 }
```

#### Matrix cell CSS (required companion)

```css
/* Dark-background matrix wrap so arrows are visible */
.matrix-wrap { display:inline-block; padding:14px; background:#1e293b; border-radius:8px; border:2px solid #334155; position:relative }
.matrix-grid { display:grid; gap:3px; border-radius:3px; position:relative }
.mat-cell { width:50px; height:50px; display:flex; flex-direction:column; align-items:center; justify-content:center;
  font-family:var(--font-mono); font-size:18px; font-weight:700; border-radius:6px;
  background:#fff; color:#1e293b; border:2px solid transparent; position:relative; transition:all .25s }
.mat-cell-coord { font-size:7px; color:var(--text-tertiary); font-weight:400; position:absolute; top:2px; left:3px; opacity:.6 }
.mat-cell--curr   { background:#FEF3C7!important; border-color:#B45309!important; animation:pulse-node 1.1s infinite; transform:scale(1.06); z-index:2 }
.mat-cell--visited{ background:#DCFCE7; color:#14532D; opacity:.9 }
.mat-cell--diag   { background:#EEEDFE!important; border-color:#534AB7!important; animation:diagonal-glow 1.4s infinite }
```

#### JS pattern — SVG arrow rendering inside `renderViz`

```js
// Cell size constants (must match CSS — CELL=50, GAP=3, PAD=14)
var CELL = 50, GAP = 3, PAD = 14;
var totalW = s.n * CELL + (s.n - 1) * GAP;
var totalH = s.m * CELL + (s.m - 1) * GAP;

// Center of a cell in grid-local coords
function cx(col) { return col * (CELL + GAP) + CELL / 2; }
function cy(row) { return row * (CELL + GAP) + CELL / 2; }

// Only draw when there are visited cells to connect
if (s.visitedSet.length >= 2 || (s.visitedSet.length >= 1 && s.hl.currCell)) {
  h += '<svg class="flow-svg" width="' + totalW + '" height="' + totalH + '" viewBox="0 0 ' + totalW + ' ' + totalH + '">';
  // Arrowhead marker defs (one per color)
  h += '<defs>';
  h += '<marker id="ah-red"    viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#dc2626"/></marker>';
  h += '<marker id="ah-amber"  viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#f59e0b"/></marker>';
  h += '<marker id="ah-purple" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#8b5cf6"/></marker>';
  h += '</defs>';

  // Draw arrows between consecutive visited cells
  var visits = s.visitedSet;
  for (var pi = 0; pi < visits.length - 1; pi++) {
    var a = visits[pi], b = visits[pi + 1];
    var x1 = cx(a[1]), y1 = cy(a[0]);
    var x2 = cx(b[1]), y2 = cy(b[0]);

    // Detect phase jump: when r+c sum changes (diagonal switch, BFS level, etc.)
    var isPhaseJump = (a[0] + a[1]) !== (b[0] + b[1]);
    var cls    = isPhaseJump ? 'flow-line flow-line--diag-jump' : 'flow-line';
    var marker = isPhaseJump ? 'ah-purple' : 'ah-red';

    // Shorten line so arrowhead clears the cell border (shrink=18px from each end)
    var dx = x2 - x1, dy = y2 - y1, len = Math.sqrt(dx * dx + dy * dy);
    var shrink = 18, ux = dx / len, uy = dy / len;
    var sx1 = x1 + ux * shrink, sy1 = y1 + uy * shrink;
    var sx2 = x2 - ux * shrink, sy2 = y2 - uy * shrink;
    h += '<line class="' + cls + '" x1="' + sx1 + '" y1="' + sy1 + '" x2="' + sx2 + '" y2="' + sy2 + '" marker-end="url(#' + marker + ')"/>';
  }

  // Draw amber "next move" arrow from last visited → currCell (if different)
  if (s.hl.currCell && visits.length > 0) {
    var last = visits[visits.length - 1], cur = s.hl.currCell;
    if (last[0] !== cur[0] || last[1] !== cur[1]) {
      var x1c = cx(last[1]), y1c = cy(last[0]);
      var x2c = cx(cur[1]),  y2c = cy(cur[0]);
      var dxc = x2c - x1c, dyc = y2c - y1c, lenc = Math.sqrt(dxc * dxc + dyc * dyc);
      if (lenc > 0) {
        var uxc = dxc / lenc, uyc = dyc / lenc, shrinkC = 18;
        var sx1c = x1c + uxc * shrinkC, sy1c = y1c + uyc * shrinkC;
        var sx2c = x2c - uxc * shrinkC, sy2c = y2c - uyc * shrinkC;
        h += '<line class="flow-line flow-line--current" x1="' + sx1c + '" y1="' + sy1c + '" x2="' + sx2c + '" y2="' + sy2c + '" marker-end="url(#ah-amber)"/>';
      }
    }
  }
  h += '</svg>';
}
```

#### State tracking required in `buildSteps`

```js
// Maintain a visited-in-order array in the step snapshot
var visitedSet = [];   // each entry: [row, col]  — pushed when result.append() fires

// In snap():
visitedSet: visitedSet.map(function(p) { return p.slice(); })

// When a cell is appended to result, also push to visitedSet:
visitedSet.push([r, c]);
```

#### Legend below the grid (required)

```html
<div class="legend">
  <span><i style="background:#fff"></i>Unvisited</span>
  <span><i style="background:#EEEDFE;border-color:#534AB7"></i>Current phase cells</span>
  <span><i style="background:#FEF3C7;border-color:#B45309"></i>Active cell</span>
  <span><i style="background:#DCFCE7;border-color:#15803D"></i>Visited</span>
  <span><i class="legend-flow"></i>Flow (within phase)</span>
  <span><i class="legend-flow" style="border-top-color:#8b5cf6"></i>Phase jump</span>
  <span><i class="legend-flow-curr"></i>Next move</span>
</div>
```

```css
.legend { display:flex; gap:10px; flex-wrap:wrap; margin-top:8px }
.legend span { display:inline-flex; align-items:center; gap:5px; font-size:10px; color:var(--text-secondary) }
.legend i { display:inline-block; width:14px; height:14px; border-radius:3px; border:1px solid var(--border-color) }
.legend i.legend-flow      { width:22px; height:0; border:0; border-top:2.5px dashed #dc2626 }
.legend i.legend-flow-curr { width:22px; height:0; border:0; border-top:3px   dashed #f59e0b }
```

#### V9 checklist

- [ ] `.matrix-wrap` has `position:relative` so SVG can be `position:absolute` inside it
- [ ] `.flow-svg` sits at `top:PAD; left:PAD` (matching the matrix-wrap padding — usually `14px`)
- [ ] SVG `width`/`height`/`viewBox` match total grid pixel dimensions (no padding)
- [ ] Cell center formula: `col*(CELL+GAP)+CELL/2` and `row*(CELL+GAP)+CELL/2`
- [ ] Lines shortened by `shrink=18px` from each end so arrowhead clears cell border
- [ ] Phase-jump detection via sum `r+c` change (adapt condition per problem: BFS level, diagonal index, etc.)
- [ ] Amber "next move" arrow only drawn when `currCell ≠ last visited`
- [ ] `visitedSet` in snapshot is a **deep copy** (`.map(p => p.slice())`)
- [ ] Legend rendered below the grid (not inside the SVG)
- [ ] `flow-dash` and `flow-pulse` keyframes declared in `<style>`
- [ ] All 3 arrowhead `<marker>` defs inside `<defs>` at top of SVG


---

## 🔹 DP Table Recurrence / Dependency Arrows (new pattern — lc72 reference)

For **DP problems on grids/tables** (edit distance, LCS, 2D knapsack, matrix chain, etc.) the visualization should overlay **recurrence arrows** that make the "why this value?" decision visible for the current cell.

### Core idea
- When a cell becomes "current" (`hiI`, `hiJ`), draw arrows **from its 3 (or N) possible predecessors** according to the recurrence.
- **Small & compact arrows** (marker 5–6 units) so they never overpower the numbers in the cells.
- **Dashed yellow/orange** (subtle) = the candidate sources (all legal previous cells).
- **Solid green + soft halo** (prominent) = the actual chosen / min predecessor.
- Always computed **live from `snap.dp`** + current cell coordinates → works perfectly even when viewing the fully filled table on the final return step.

### Small arrow + visibility best practices (lc72)
- Use very small, clean markers:
  ```html
  <!-- subtle chevron for candidates -->
  <marker id="ah-cand" viewBox="0 0 8 8" refX="7" refY="4" markerWidth="5" markerHeight="5">
    <path d="M1,1 L7,4 L1,7" fill="none" stroke="#f59e0b" stroke-width="2" stroke-linecap="round"/>
  </marker>
  <!-- compact filled + outline for chosen -->
  <marker id="ah-chosen" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6">
    <path d="M1,1.5 L8,5 L1,8.5 L2.5,5 Z" fill="#22c55e" stroke="#166534" stroke-width="1.2"/>
  </marker>
  ```
- Line weights kept deliberately small (candidates ~2–2.5px dashed, chosen ~4px solid).
- For the chosen arrow, prepend a **soft halo** (slightly thicker, low-opacity same path, no marker) — gives excellent visual pop and emphasis.
- **Critical visibility rules**:
  - `s += candLines + chosenHalo + chosenLine;` (candidates first, chosen last in SVG DOM).
  - `.flow-svg { z-index: 100; }` (much higher than any `.current` z-index:5 or `.win`).
  - `shrink = 12` (tighter than traversal arrows) so the small head sits cleanly near the target cell.
- Dark high-contrast container (`#0f172a`) + subtle inner frame.
- Always provide an integrated caption right under the grid (highly recommended):
  > Small arrows: dashed = 3 candidate predecessors, green + halo = chosen min cost source

### When to apply
Use this pattern for any 2D DP whose recurrence has a small fixed number of sources (up/left/diag, above/left, etc.). It is one of the most powerful ways to make the DP recurrence "click" for learners.

---

## 🔹 LOG PREVIEW — Floating Popup (bottom-right corner)

Floating card at right corner of viz area. **NO toolbar inside popup** — controls in footer.

```html
<div class="log-popup" id="logPopup">
  <div class="log-popup-body" id="logPopupBody">
    <!-- richLog() output rendered here -->
  </div>
</div>
```

```css
.log-popup {
  --log-scale: 1;
  position: fixed; bottom: 52px; right: 14px;
  width: 480px;   /* base; visual size via transform */
  max-width: 92vw; max-height: 75vh;
  border-radius: var(--radius-lg);
  box-shadow: 0 10px 32px rgba(0,0,0,.14);
  border: 1px solid var(--border-color);
  background: var(--bg-card);
  overflow: hidden;
  display: flex; flex-direction: column;
  z-index: 100;
  transition: opacity .15s, transform .15s;
  transform: scale(var(--log-scale));
  transform-origin: bottom right;
}
.log-popup--hidden { opacity: 0; pointer-events: none; }
.log-popup-body { flex: 1; overflow: auto; padding: 0; width: 100%; }
.log-popup-body > .log-card { border-radius: 0; border: none; border-left: 0 }
```

- [ ] Width **480px** default (wider than v1's 420px to fit 6 sections); visual size (width + height + all content) controlled via `transform: scale(var(--log-scale))` on `.log-popup` with `transform-origin: bottom right`
- [ ] Max-height **75vh** (taller than v1's 65vh)
- [ ] NO padding on body (sections have their own)
- [ ] Zoom via `--log-scale` CSS variable (60%–180%, 20% steps). Buttons in footer use `.btn-zoom` class for − / + / ↺, `.log-zoom-label` for the % span. Full visual scaling of box + fonts + "and so on" via the transform (internal elements can use `calc(XXpx * var(--log-scale))` for fonts if needed). Call `updateLogZoom()` inside `showLogPreview()`.
- [ ] Fullscreen (`⛶`) button uses `.btn btn-fullscreen` class and targets `.col-viz` for `requestFullscreen` / `exitFullscreen`

---

## 🆕 🔹 Enhanced Log Card Schema v2 (richLog)

> **THE single most important enhancement.** Every step's log popup MUST be built via `richLog(opts)` helper.

### richLog(opts) API

```js
function richLog(opts) {
  // opts: {
  //   line: <0-indexed code line>,
  //   action: <action-type>,
  //   code: "<actual code being executed>",
  //   status: "success" | "fail" | "skip" | "loop" | "prune" | "info",
  //   inputs: [{k, v}, ...],            // variables this line READS
  //   outputs: [{k, v, changed}, ...],  // variables this line WROTE
  //   verdict: "x (5) > y (3)",         // for if/for/while only
  //   verdictTrue: true | false,
  //   why: "<Hinglish technical explanation>",
  //   analogy: "<Hinglish real-world metaphor>"
  // }
  // Returns: HTML string
}
```

### Visual layout of card

```
┌─────────────────────────────────────────┐
│ [BADGE] [L5] ............ [STATUS PILL] │  ← header (colored left border)
├─────────────────────────────────────────┤
│ 📜 Code Executing                       │
│ ┌─────────────────────────────────────┐ │
│ │ if value == target:                 │ │  ← syntax-highlighted dark code
│ └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│ 📥 Input Values                         │
│ [value=6] [target=6]                    │  ← blue-tinted KV pills
├─────────────────────────────────────────┤
│ 📤 Output / State After Line            │
│ [res=["1+2+3"]] [res.size=1]            │  ← green-tinted (or amber if changed)
├─────────────────────────────────────────┤
│ ⚖️ Verdict / Decision                   │
│ ✓ value (6) == target (6)               │  ← green if true, red if false
├─────────────────────────────────────────┤
│ 💡 Why This Happens                     │
│ Base case HIT! Saari digits consume...  │
├─────────────────────────────────────────┤
│ 🎭 Real-Life Analogy                    │
│ Jaise cricket match mein last ball...   │  ← amber/cream callout
└─────────────────────────────────────────┘
```

### Required sections per step type

| Scenario | Status | Required sections |
|---|---|---|
| Variable assignment | `success` | code, inputs, outputs, why, analogy |
| If condition (true) | `success` | code, inputs, verdict (true), why, analogy |
| If condition (false) | `skip` | code, inputs, verdict (false), why, analogy |
| For/While iteration | `loop` | code, inputs, outputs (i changed), why, analogy |
| Function call | `info` | code, inputs (args), outputs (frame opened), why, analogy |
| Function return | `info`/`success` | code, inputs, outputs, why, analogy |
| Break / prune | `prune` | code, inputs, outputs (loop terminated), why, analogy |
| Found result | `success` | code, inputs, outputs (changed), why, analogy |
| Failure / no match | `fail` | code, inputs, verdict, why, analogy |
| Loop natural exit | `info` | code, inputs (final i), outputs, why, analogy |

### Log card CSS (key classes)

```css
.log-card { border-radius:var(--radius-md); overflow:hidden; border:1px solid var(--border-color); border-left-width:4px; font-size:var(--fs-xs); background:var(--bg-card) }

.log-card-header { display:flex; align-items:center; gap:6px; padding:8px 10px 6px; background:var(--bg-secondary); border-bottom:1px solid var(--border-light) }
.log-line-ref { font:700 10px var(--font-mono); color:var(--accent-indigo); background:var(--accent-indigo-bg); padding:1px 6px; border-radius:4px; border:1px solid var(--accent-indigo) }
.log-status-pill { font-family:system-ui,"Segoe UI Emoji",sans-serif; font-size:9px; font-weight:700; padding:2px 7px; border-radius:99px; text-transform:uppercase; margin-left:auto }
.log-status--success { background:var(--accent-green-bg); color:var(--accent-green-text); border:1px solid var(--accent-green) }
.log-status--fail { background:var(--accent-red-bg); color:var(--accent-red-text); border:1px solid var(--accent-red) }
.log-status--skip { background:var(--bg-tertiary); color:var(--text-tertiary); border:1px solid var(--border-color) }
.log-status--loop { background:rgba(56,139,253,.14); color:#185FA5; border:1px solid #388bfd }
.log-status--prune { background:var(--accent-red-bg); color:var(--accent-red-text); border:1px solid var(--accent-red) }
.log-status--info { background:rgba(130,80,255,.14); color:#3C3489; border:1px solid #8250ff }

.log-section { padding:7px 10px; border-bottom:1px dashed var(--border-light) }
.log-section:last-child { border-bottom:none }
.log-section-title { font:700 8px var(--font-mono); text-transform:uppercase; letter-spacing:.06em; color:var(--text-tertiary); margin-bottom:3px; display:flex; align-items:center; gap:4px }

.log-code-snippet { font:600 11px var(--font-mono); background:#0d1117; color:#e6edf3; padding:6px 9px; border-radius:4px; display:block; overflow-x:auto; white-space:pre; border:1px solid #30363d }
.log-code-snippet .ck { color:#ff7b72 }    /* keyword */
.log-code-snippet .cs { color:#a5d6ff }    /* string */
.log-code-snippet .cn { color:#79c0ff }    /* number */
.log-code-snippet .cf { color:#d2a8ff }    /* function */
.log-code-snippet .cc { color:#8b949e; font-style:italic }  /* comment */

.log-kv { display:inline-flex; align-items:center; gap:3px; font:11px var(--font-mono); padding:2px 7px; border-radius:4px; background:var(--bg-secondary); border:1px solid var(--border-light) }
.log-kv-key { color:var(--text-tertiary); font-weight:500 }
.log-kv-val { color:var(--text-primary); font-weight:700 }
.log-kv--in { background:rgba(56,139,253,.08); border-color:rgba(56,139,253,.3) }
.log-kv--out { background:rgba(63,185,80,.08); border-color:rgba(63,185,80,.4) }
.log-kv--out .log-kv-val { color:#15803D }
.log-kv--changed { background:rgba(180,83,9,.10); border-color:rgba(180,83,9,.4) }

.log-verdict { display:flex; align-items:center; gap:6px; font:600 11px var(--font-mono); padding:5px 8px; border-radius:5px; background:var(--bg-secondary); border:1px solid var(--border-light) }
.log-verdict--true { background:var(--accent-green-bg); border-color:var(--accent-green); color:var(--accent-green-text) }
.log-verdict--false { background:var(--accent-red-bg); border-color:var(--accent-red); color:var(--accent-red-text) }

.log-analogy { font:italic 11px var(--font-sans); color:var(--text-secondary); background:#FFF8E7; border:1px solid #FBE8A6; border-left:3px solid var(--accent-amber); padding:6px 9px; border-radius:4px; line-height:1.55 }
.log-analogy strong { color:var(--accent-amber-text); font-style:normal; font-weight:700 }

.log-why { font-size:11px; line-height:1.55; color:var(--text-primary); padding:4px 0 }
.log-why strong { color:var(--accent-indigo-text) }
```


### richLog() helper full implementation

```js
function richLog(opts) {
  var statusMap = {
    success: "✅ Success", fail: "❌ Failed", skip: "⏭ Skipped",
    loop: "🔁 Loop iter", prune: "✂️ Pruned", info: "ℹ️ Info"
  };
  var status = opts.status || "info";
  var html = '<div class="log-card log-card--' + opts.action + '">';

  // Header
  html += '<div class="log-card-header">';
  html += '<span class="log-badge log-badge--' + opts.action + '">' + (LOG_BADGE[opts.action] || opts.action) + '</span>';
  html += '<span class="log-line-ref">L' + (opts.line + 1) + '</span>';
  html += '<span class="log-status-pill log-status--' + status + '">' + statusMap[status] + '</span>';
  html += '</div>';

  // Code snippet
  if (opts.code) {
    html += '<div class="log-section">';
    html += '<div class="log-section-title"><span class="log-section-title-icon">📜</span>Code Executing</div>';
    html += '<code class="log-code-snippet">' + hlCode(opts.code) + '</code>';
    html += '</div>';
  }

  // Inputs
  if (opts.inputs && opts.inputs.length) {
    html += '<div class="log-section">';
    html += '<div class="log-section-title"><span class="log-section-title-icon">📥</span>Input Values (line is using these)</div>';
    html += '<div class="log-kv-row">';
    opts.inputs.forEach(function(kv) {
      html += '<span class="log-kv log-kv--in"><span class="log-kv-key">' + kv.k + '</span>=<span class="log-kv-val">' + kv.v + '</span></span>';
    });
    html += '</div></div>';
  }

  // Outputs
  if (opts.outputs && opts.outputs.length) {
    html += '<div class="log-section">';
    html += '<div class="log-section-title"><span class="log-section-title-icon">📤</span>Output / State After Line</div>';
    html += '<div class="log-kv-row">';
    opts.outputs.forEach(function(kv) {
      var cls = kv.changed ? 'log-kv--changed' : 'log-kv--out';
      html += '<span class="log-kv ' + cls + '"><span class="log-kv-key">' + kv.k + '</span>=<span class="log-kv-val">' + kv.v + '</span></span>';
    });
    html += '</div></div>';
  }

  // Verdict
  if (opts.verdict) {
    var vCls = opts.verdictTrue === true ? 'log-verdict--true' : (opts.verdictTrue === false ? 'log-verdict--false' : '');
    var vIcon = opts.verdictTrue === true ? '✓' : (opts.verdictTrue === false ? '✗' : '•');
    html += '<div class="log-section">';
    html += '<div class="log-section-title"><span class="log-section-title-icon">⚖️</span>Verdict / Decision</div>';
    html += '<div class="log-verdict ' + vCls + '">' + vIcon + ' ' + opts.verdict + '</div>';
    html += '</div>';
  }

  // Why
  if (opts.why) {
    html += '<div class="log-section">';
    html += '<div class="log-section-title"><span class="log-section-title-icon">💡</span>Why This Happens</div>';
    html += '<div class="log-why">' + opts.why + '</div>';
    html += '</div>';
  }

  // Analogy
  if (opts.analogy) {
    html += '<div class="log-section">';
    html += '<div class="log-section-title"><span class="log-section-title-icon">🎭</span>Real-Life Analogy</div>';
    html += '<div class="log-analogy">' + opts.analogy + '</div>';
    html += '</div>';
  }

  html += '</div>';
  return html;
}
```

### hlCode() mini Python tokenizer

```js
function hlCode(raw) {
  var h = raw;
  h = h.replace(/("[^"]*"|'[^']*')/g, '\x01s\x02$1\x03');
  h = h.replace(/\b(def|for|in|if|return|break|continue|and|else|elif|True|False|None|not|or|while|lambda)\b/g, '\x01k\x02$1\x03');
  h = h.replace(/\b(len|range|int|str|list|append|pop|sorted|min|max|sum|abs|enumerate|zip|print|set|add)\b/g, '\x01f\x02$1\x03');
  h = h.replace(/(?<![.\w"\x01\x02\x03])(-?\d+)\b/g, '\x01n\x02$1\x03');
  h = h.replace(/(#.*$)/gm, '\x01c\x02$1\x03');
  h = h.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
  h = h.replace(/\x01([a-z])\x02/g, function(_, c) { return '<span class="c' + c + '">'; }).replace(/\x03/g, '</span>');
  return h;
}
```

### Step object schema (v2)

```js
{
  line: <0-indexed code line>,
  action: <action-type>,
  logHTML: <pre-rendered HTML from richLog()>,  // ← NEW: pre-rendered for performance
  // + deep-copied state snapshot:
  // arr, dp, path, grid, visited, graph, state, etc.
}
```

**Key changes from v1:**

- [ ] ❌ REMOVED: separate `log`, `detail`, `why` string fields
- [ ] ✅ NEW: single `logHTML` field with full richLog output pre-rendered
- [ ] ✅ `showLogPreview(s)` simply does `logPopupBody.innerHTML = s.logHTML`

### Log card MUST cover these scenarios

✅ Variable assignment | ✅ If true | ✅ If false (skip) | ✅ For/While iter | ✅ Function call | ✅ Function return | ✅ Break/prune | ✅ Found result | ✅ Failure | ✅ Loop natural exit

---

## 🔹 Footer Controls

Footer bar split: left 1/3 (under code) + right 2/3 (under viz).

```html
<div class="footer-bar">
  <div class="footer-left" id="footerLeft">
    <span id="stepInfo" class="step-info">0/0</span>
    <span id="actionBadge" class="action-badge"></span>
  </div>
  <div class="footer-right">
    <kbd>←</kbd><kbd>→</kbd>
    <button class="btn" type="button" onclick="resetStep()">↺</button>
    <button class="btn" type="button" onclick="stepBy(-1)">◀</button>
    <button class="btn btn-primary" type="button" onclick="stepBy(1)">▶ Next</button>
    <span class="footer-sep"></span>
    <button id="logToggleBtn" class="log-toggle-btn" type="button" onclick="toggleLogPreview()">📋 ON</button>
    <button class="btn-zoom" type="button" onclick="logZoom(-1)">−</button>
    <span id="logZoomLabel" class="log-zoom-label">100%</span>
    <button class="btn-zoom" type="button" onclick="logZoom(1)">+</button>
    <button class="btn-zoom" type="button" onclick="logZoomReset()">↺</button>
    <span class="footer-sep"></span>
    <button class="btn btn-fullscreen" type="button" onclick="toggleFullscreen()">⛶</button>
  </div>
</div>
```

### Footer checklist

- [ ] Split layout: left 1/3 (under code) + right 2/3 (under viz)
- [ ] Left: step counter + action badge
- [ ] Right (in order): kbd hints + ↺ + ◀ + ▶ Next + | + 📋 ON/OFF + − + 100% + + + ↺ + | + ⛶
- [ ] ❌ NO Play, NO speed selector, NO progress bar
- [ ] All buttons `type="button"`
- [ ] `.footer-left` width matches `.col-code` (synced via `syncFooterWidth()`)

### ⌨️ Keyboard shortcuts

| Key | Action |
|---|---|
| `→` | next step |
| `←` | previous step |
| `R` | reset to step 0 |

```js
document.addEventListener('keydown', function(e) {
  if (e.target.tagName === 'INPUT' || e.target.tagName === 'SELECT') return;
  if (e.key === 'ArrowRight') { e.preventDefault(); stepBy(1); }
  else if (e.key === 'ArrowLeft') { e.preventDefault(); stepBy(-1); }
  else if (e.key === 'r' || e.key === 'R') { e.preventDefault(); resetStep(); }
});
```


---

## 🛠️ JS Patterns

### State variables

```js
var codeLines = [/* python lines */];
var steps = [];
var currentStep = 0;
var codeWidthUserSet = false;
var curResult = null;
var logEnabled = true;
var logZoomLevel = 100;
```

### buildSteps — the core function (with richLog)

```js
function buildSteps(/* ...inputs */) {
  var st = [];
  var MAX_STEPS = 3000;  // safety cap
  var /* live state vars */;

  function snap(extra) {
    return Object.assign({
      /* freeze full state — ALWAYS clone arrays via .slice() */
    }, extra || {});
  }

  function pushStep(o) {
    if (st.length >= MAX_STEPS) return;
    st.push(Object.assign({ line: 0, action: "init", logHTML: "" }, snap(o)));
  }

  // Example pushStep with richLog:
  pushStep({
    line: 4,
    action: "if-check",
    logHTML: richLog({
      line: 4,
      action: "if-check",
      code: "if index == n:",
      status: baseHit ? "success" : "skip",
      inputs: [{k: "index", v: idx}, {k: "n", v: n}],
      verdict: "index (" + idx + ") " + (baseHit ? "==" : "!=") + " n (" + n + ")",
      verdictTrue: baseHit,
      why: baseHit
        ? "<strong>Base case HIT!</strong> Saari digits consume ho gayi..."
        : "Abhi <strong>" + (n - idx) + " digit(s) bachi hain</strong>...",
      analogy: baseHit
        ? "Jaise ek <strong>train apne final station</strong> pe pahunch gayi..."
        : "Train abhi raste mein hai — agle station ka decision lena hai."
    })
  });

  return st;
}
```

**Golden rules:**

- [ ] Snapshot deep copies — `.slice()`, `.map(r => r.slice())`. NEVER store references
- [ ] Every step has `{line, action, logHTML}` minimum + state snapshot
- [ ] `logHTML` is pre-rendered via `richLog()` (better perf than rendering on each step)
- [ ] One step PER code line PER execution (RULE #2)
- [ ] After recursion, RESTORE local vars (closure shares them across recursion frames)

### renderStep

```js
function renderStep(idx) {
  if (!steps.length) return;
  var s = steps[idx];
  currentStep = idx;
  document.getElementById('stepInfo').textContent = (idx + 1) + '/' + steps.length;
  var bd = document.getElementById('actionBadge');
  bd.textContent = BADGE_LABEL[s.action] || s.action;
  bd.className = 'action-badge action-badge--' + s.action;
  document.querySelectorAll('.code-line').forEach(function(el) { el.classList.remove('active'); });
  var al = document.querySelector('.code-line[data-line="' + s.line + '"]');
  if (al) { al.classList.add('active'); al.scrollIntoView({block: 'nearest'}); }
  renderViz(s);
  showLogPreview(s);
}

function showLogPreview(s) {
  if (!logEnabled) return;
  document.getElementById('logPopupBody').innerHTML = s.logHTML || '';
  document.getElementById('logPopup').classList.remove('log-popup--hidden');
  updateLogZoom();   // re-apply current --log-scale (fonts + full box transform)
}
```

### resetWithInput

```js
function resetWithInput() {
  curResult = null;
  var params = parseInputs();  // read ALL input boxes
  renderCode();
  fitCodeWidth();
  steps = buildSteps(params);
  currentStep = 0;
  renderStep(0);
  document.getElementById('liveOutput').textContent = curResult !== null ? curResult : '—';
}
```

### Resize handle (1 only)

```js
(function() {
  var col1 = document.getElementById('col1');
  document.getElementById('resizeH1').addEventListener('mousedown', function(e) {
    e.preventDefault();
    codeWidthUserSet = true;
    var startX = e.clientX, startW = col1.offsetWidth, h = this;
    document.body.classList.add('no-select');
    h.classList.add('active');
    function mv(e) {
      col1.style.width = Math.max(140, startW + (e.clientX - startX)) + 'px';
      syncFooterWidth();
    }
    function up() {
      document.removeEventListener('mousemove', mv);
      document.removeEventListener('mouseup', up);
      document.body.classList.remove('no-select');
      h.classList.remove('active');
    }
    document.addEventListener('mousemove', mv);
    document.addEventListener('mouseup', up);
  });
})();

function syncFooterWidth() {
  document.getElementById('footerLeft').style.width =
    document.getElementById('col1').offsetWidth + 'px';
}

window.addEventListener('resize', function() {
  if (!codeWidthUserSet) fitCodeWidth();
  else syncFooterWidth();
});
```

### Log popup controls

```js
function toggleLogPreview() {
  logEnabled = !logEnabled;
  var btn = document.getElementById('logToggleBtn');
  btn.textContent = logEnabled ? '📋 ON' : '📋 OFF';
  btn.classList.toggle('log-toggle-btn--off', !logEnabled);
  if (!logEnabled) hideLogPopup();
  else if (steps.length) showLogPreview(steps[currentStep]);
}

function hideLogPopup() {
  document.getElementById('logPopup').classList.add('log-popup--hidden');
}

function logZoom(dir) {
  logZoomLevel = Math.max(60, Math.min(180, logZoomLevel + dir * 20));
  updateLogZoom();
}
function logZoomReset() { logZoomLevel = 100; updateLogZoom(); }
function updateLogZoom() {
  document.getElementById('logZoomLabel').textContent = logZoomLevel + '%';
  document.getElementById('logPopup').style.setProperty('--log-scale', logZoomLevel / 100);
}

function toggleFullscreen() {
  var viz = document.querySelector('.col-viz');
  if (!document.fullscreenElement) viz.requestFullscreen().catch(function(){});
  else document.exitFullscreen();
}
```

**Important for full scaling effect (width + height + font size + all content):**
- Footer zoom buttons use `.btn-zoom` (for − + ↺) and `.log-zoom-label` (for the % span).
- Fullscreen uses `.btn btn-fullscreen`.
- `showLogPreview` must call `updateLogZoom()` to re-apply the scale.
- On `.log-popup` use `transform: scale(var(--log-scale)); transform-origin: bottom right;` (plus base width/height) so the whole popup visually grows/shrinks uniformly from its bottom-right anchor. This makes width, height, fonts, paddings, cards etc. all increase/decrease together. The label shows the current % (60–180).


---

## 🎨 CSS Design System

> 🔴 **Light theme ONLY. No dark mode.** Follow Claude/Anthropic white aesthetic: warm off-white backgrounds, clean borders, indigo accents.

### Core variables (light only)

```css
:root {
  /* Backgrounds — warm white */
  --bg-primary: #FAFAF8;
  --bg-secondary: #F1EFE8;
  --bg-tertiary: #E2E0D8;
  --bg-card: #FFFFFF;

  /* Text — warm greys */
  --text-primary: #2C2C2A;
  --text-secondary: #5F5E5A;
  --text-tertiary: #888780;

  /* Borders */
  --border-color: rgba(0,0,0,0.11);
  --border-light: rgba(0,0,0,0.06);
  --border-focus: #534AB7;

  /* Accents — Claude indigo primary */
  --accent-indigo: #534AB7;
  --accent-indigo-bg: #EEEDFE;
  --accent-indigo-text: #3C3489;
  --accent-indigo-hover: #4740A0;

  /* Teal — success/destination */
  --accent-teal: #0F6E56;
  --accent-teal-bg: #E1F5EE;
  --accent-teal-text: #085041;

  /* Red — error/failure */
  --accent-red: #E24B4A;
  --accent-red-bg: #FCEBEB;
  --accent-red-text: #791F1F;

  /* Amber — warning/visiting */
  --accent-amber: #B45309;
  --accent-amber-bg: #FEF3C7;
  --accent-amber-text: #78350F;

  /* Green — confirmed/done */
  --accent-green: #15803D;
  --accent-green-bg: #DCFCE7;
  --accent-green-text: #14532D;

  /* Syntax (code panel) */
  --syn-keyword: #534AB7;
  --syn-function: #185FA5;
  --syn-number: #993C1D;
  --syn-string: #0F6E56;
  --active-line-bg: #EEEDFE;

  /* Spacing & Typography */
  --radius-sm: 4px; --radius-md: 6px; --radius-lg: 10px; --radius-xl: 14px;
  --sp-2: 8px; --sp-3: 12px; --sp-5: 20px;
  --font-sans: 'DM Sans', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
  --fs-xs: 11px; --fs-sm: 12px; --fs-base: 14px; --fs-md: 15px; --fs-lg: 18px;
  --fw-medium: 500; --fw-semibold: 600; --fw-bold: 700;
  --lh-normal: 1.6; --lh-code: 1.9;
  --ease: .18s ease;
}
```

- [ ] ❌ **NO `@media(prefers-color-scheme:dark)` block** — removed entirely
- [ ] All colors via `var(--token)` — never hardcoded hex except in `@keyframes`
- [ ] Google Fonts `@import` at top of `<style>`
- [ ] Custom scrollbar: 5px, transparent track, `--border-color` thumb
- [ ] **Exception:** `.log-code-snippet` uses GitHub dark (`#0d1117`) for code readability

---

## 🎨 Action Types & Colors

```js
var LOG_BADGE = {
  "func-def": "📌 DEF", "init": "⚙️ INIT", "call": "📞 CALL", "return": "🏆 RETURN",
  "for-check": "🔁 FOR", "while-check": "🔁 WHILE", "if-check": "🔎 IF",
  "mark": "✅ MARK", "push": "⬆ PUSH", "pop": "⬇ POP", "restore": "↩ RESTORE", "found": "🎉 FOUND",
  "remove": "✏️ EDIT", "prune": "✂️ PRUNE", "fail": "❌ FAIL",
  "exit": "🔚 EXIT", "skip": "⏭ SKIP"
};
var BADGE_LABEL = {
  "func-def": "DEF", "init": "INIT", "call": "CALL", "return": "RETURN",
  "for-check": "FOR", "while-check": "WHILE", "if-check": "IF",
  "mark": "MARK", "push": "PUSH", "pop": "POP", "restore": "RESTORE", "found": "FOUND",
  "remove": "EDIT", "prune": "PRUNE", "fail": "FAIL",
  "exit": "EXIT", "skip": "SKIP"
};
```

### Action → Color mapping

| Action | Badge | Color | Use case |
|---|---|---|---|
| `func-def` | 📌 DEF | 🟣 `#8250ff` | Function definition / entry |
| `init` | ⚙️ INIT | 🟣 `#8250ff` | Variable initialization |
| `call` | 📞 CALL | 🟣 `#8250ff` | Function/recursion call |
| `return` | 🏆 RETURN | 🟣 `#8250ff` | Function return |
| `for-check` | 🔁 FOR | 🔵 `#388bfd` | For-loop iteration |
| `while-check` | 🔁 WHILE | 🔵 `#388bfd` | While-loop check |
| `if-check` | 🔎 IF | 🔵 `#388bfd` | If/elif evaluation |
| `mark` | ✅ MARK | 🟢 `#3fb950` | Boolean flag set |
| `push` | ⬆ PUSH | 🟢 `#3fb950` | Stack/queue push |
| `pop` | ⬇ POP | 🟢 `#3fb950` | Stack/queue pop |
| `restore` | ↩ RESTORE | 🟢 `#3fb950` | Backtrack undo |
| `found` | 🎉 FOUND | 🟢 `#3fb950` | Valid answer recorded |
| `remove` | ✏️ EDIT | 🔴 `#f85149` | In-place mutation |
| `prune` | ✂️ PRUNE | 🔴 `#f85149` | Branch cut |
| `fail` | ❌ FAIL | 🔴 `#f85149` | Negative outcome |
| `exit` | 🔚 EXIT | ⚫ `#555450` | Loop natural exit |
| `skip` | ⏭ SKIP | ⚫ `#555450` | Branch not taken |

---

## 🆕 🚦 Status Pills

Top-right of each log card header — appears via `richLog({status: "..."})`.

| Status | Pill text | Background | When to use |
|---|---|---|---|
| `success` | ✅ Success | green | Line executed and produced expected result |
| `fail` | ❌ Failed | red | Line ran but condition/check failed |
| `skip` | ⏭ Skipped | gray | Branch not taken (else, false-if) |
| `loop` | 🔁 Loop iter | blue | Loop iteration boundary |
| `prune` | ✂️ Pruned | red | Early termination (break, return on bad path) |
| `info` | ℹ️ Info | purple | Neutral execution (function call, init, return) |

### Choosing the right status

- [ ] `if` evaluates true → `success`
- [ ] `if` evaluates false → `skip`
- [ ] for/while iteration → `loop`
- [ ] for/while natural exit → `info`
- [ ] break from prune condition → `prune`
- [ ] Variable assignment / init → `success`
- [ ] Function call / return → `info`
- [ ] Found valid answer → `success`
- [ ] Returned without finding → `fail`


---

## 🆕 ✍️ Hinglish Writing Style

Every `why` and `analogy` field MUST be in conversational Hinglish.

### ✅ Good examples

> **why:** "<strong>Base case</strong> HIT! Saari digits <strong>consume</strong> ho gayi. Ab check karna hai ki current <code>expression</code> <code>target</code> ke barabar evaluate hua ya nahi."
>
> **analogy:** "Jaise cricket match mein last ball pe required runs ban gaye — match jeet liya, board pe naam likho!"

> **why:** "<strong>Leading zero</strong> detected! Multi-digit operand jo '0' se start hota hai (jaise '01', '012') invalid hai. <strong>Loop break</strong> karenge."
>
> **analogy:** "Bank cheque pe '0123' likhna invalid hai — leading zero allowed nahi. Cheque reject!"

### ❌ Avoid

- Pure English: "The base case has been hit, and now we check..."
- Pure Hindi: "आधार स्थिति पर पहुंच गए हैं..."
- Overly formal: "It is observed that the variable has been mutated."
- Robotic: "Step executed successfully. Variable updated."

### Style rules

| Rule | Example |
|---|---|
| Bold key technical terms with `<strong>` | `<strong>base case</strong>`, `<strong>pruning</strong>`, `<strong>backtracking</strong>` |
| Use `<code>` for variables | `<code>index</code>`, `<code>num[i:j]</code>` |
| Real-life analogies preferred | Cricket, cooking, banking, train, traffic, school, pizza, chess, cheque, salary |
| Emojis sparingly | 1–2 per section, semantic only (🎉 for found, ✂️ for prune) |
| Concise but complete | 1–3 sentences per section |
| Conversational tone | "Dekho", "Socho", "Jaise", "Bilkul", "Toh ab..." |

### Log-specific rules

- Keep `why` and `analogy` explanations concise (short, tight 1-2 sentences) but always complete — every decision and formula must be explained.
- **Every important word, logic step, and formula is highlighted:** Wrap key concepts and operations in `<strong>` (e.g. <strong>base case</strong>, <strong>recurrence</strong>, <strong>minimum operations</strong>, <strong>insert / delete / replace</strong>) and use `<code>` for every formula, cell, variable and math expression (e.g. <code>dp[i][j]</code>, <code>1 + min(...)</code>, <code>i-1, j</code>, <code>prefix</code>). This is now mandatory in all richLog why + analogy content for scannability and teaching value (refined from lc72).
- See lc72-edit-distance.html as the reference: every log uses bold + code markup even in the shortened explanations.

### Hinglish vocabulary cheat-sheet

| English concept | Hinglish |
|---|---|
| Now we check | "Ab check karenge" / "Ab dekhte hain" |
| It happens because | "Yeh isliye hota hai kyunki" |
| Let's go | "Chalo aage badhte hain" |
| Found a match | "Match mil gaya!" / "Mil gaya bhai!" |
| It failed | "Yeh fail ho gaya" / "Path bekaar nikla" |
| Just like | "Jaise ki" / "Bilkul aise jaise" |
| Therefore | "Toh isliye" / "Iska matlab" |
| Loop iteration | "Loop ka chakkar" / "Iteration" |

---

## 🚦 Pre-delivery Master Checklist

### File structure & output

- [ ] Single self-contained `.html` file
- [ ] `<!DOCTYPE>`, `<meta charset>`, `<meta viewport>`, `<title>`
- [ ] Only external dependency: Google Fonts `@import`
- [ ] No `<form>` elements
- [ ] **Static note comment as first line:** ``
- [ ] **File name header at top of response:** `## 📁 File name: lcXXX-name.html`
- [ ] Single fenced HTML code block
- [ ] Brief recap below + follow-up offer

### Section 1 — Problem Card

- [ ] GT LEARN button — top-left, indigo filled pill, reloads page
- [ ] Header: number + name + difficulty + tags
- [ ] Header right: ALL input params as labeled inputs + expected badge + ▲/▼ toggle
- [ ] Body left: problem statement + approach + core concepts + base case + complexity
- [ ] Body left: NO code logic, NO pseudocode
- [ ] Body right: 3–5 test case chips with full input + pt-tag + expected
- [ ] **Strict two-row test chips** (follow lc239/lc691 exactly per SKILL): 
  - Top row: `.pt-tag` + **descriptive label** (e.g. "Standard", "Impossible") + `.tc-exp` ("exp: 3" or "exp: -1")
  - Bottom row: `.tc-vals` showing the **actual input values** as `<code>` pills (e.g. `stickers=[...]` and `target="..."` or raw array). 
  - Never put the expected value as the middle label. Always show real values. lc239 is the reference.
- [ ] **🔴 Test chips MUST show ACTUAL parameter values** — never abstract labels like `"mixed"`, `"all valid"` — always show real values (e.g. `lines=["987-123-4567", "123 456 7890"]`) so the user knows exactly what data is being tested
- [ ] Body right: live output + ▶ Run button
- [ ] `loadTest()` fills ALL inputs + expected badge + runs
- [ ] `toggleProblem()` shows/hides body

### Section 2 — Python Code (1/3 width)

- [ ] RULE #3 tokenizer with both `"..."` and `'...'` strings
- [ ] Active line `.active` + scrollIntoView
- [ ] `fitCodeWidth()` capped at 45vw
- [ ] `overflow-y:auto`, line-height 1.9

### Section 3 — Visualization (2/3 width)

- [ ] Variables strip at top (single source of truth — V1)
- [ ] Graphical diagrams (V2): shaded regions, arrows, boundary lines
- [ ] Diagrams SMALL by default (V3): `max-height:180px`
- [ ] ALL data structures shown (V4): even when empty
- [ ] Contextual emoji per problem domain (V5)
- [ ] Multiple viz-cards in `.viz-row` flex layout (V6)
- [ ] Stack/Queue/Set as proper visual containers (V7)
- [ ] Footer-left syncs with code column width (V8)
- [ ] Grid/Matrix problems: SVG flow-arrow overlay (traversal or DP recurrence arrows) with clear candidate vs chosen distinction + legend/caption (V9)
- [ ] **UI Layout & 750px constraint (lc239/lc691)**: All visualization sections wrapped in `.viz-wrap { max-width:750px }` + `.vars-strip { max-width:750px }`. Use stacked `.viz-sec` for each logical part. No overflow.
- [ ] **UI Layout & Width (lc239/lc691 canonical)**: Wrap visualization content in `.viz-wrap {width:100%; max-width:750px}`. Apply `max-width:750px` to `.vars-strip`. Use `.viz-sec` (with `.viz-sec-head`) for **each logical section**, stacked vertically inside the wrap (no side-by-side cramping at 750px). All sections must stay contained with no horizontal overflow. lc239 is the reference for this pattern.
- [ ] **Output section heading + no pre-show final result (lc239/lc691 canonical)**: Heading inside `.viz-sec-head` for the output area must be plain text **without any green tick ✅ emoji** — e.g. exactly `Output (Max of each window)` or `Output (Min Stickers)`. Never use "✅ Output", "✅ Final Result", or tick-prefixed headings (this was explicitly removed per user feedback). 
  For the actual result value: when the final answer (scalar or list) is only known after full run, show placeholder in the detailed viz output card until the *very last step*:
  `Waiting for full computation to complete… Step ▶ to the end`
  Reveal the real result display (cards/chips) **only** when `currentStep === steps.length - 1`. The small liveOutput summary bar above Run may update early, but the main viz Output panel must stay placeholder until end. lc239 (progressive) + lc691 (scalar final) are the references. Never pre-show final answer in the Output viz section.
- [ ] State colors: active=indigo, done=green, visiting=amber, failed=red, future=grey
- [ ] CSS animations: `pulse-node`, `shake`, `win-pulse`
- [ ] Result banner when complete
- [ ] ⛶ fullscreen via footer button

### 🆕 Log Card v2 (richLog)

- [ ] Every step uses `richLog()` helper to pre-render `logHTML`
- [ ] Header: badge + line-ref (L+1) + status pill
- [ ] 📜 Code Executing section (ALWAYS) — actual highlighted code
- [ ] 📥 Input Values section (when line reads vars) — blue-tinted KV pills
- [ ] 📤 Output / State After Line section (when line writes) — green/amber KV pills with `changed:true` flag
- [ ] ⚖️ Verdict / Decision section (for if/for/while) — verdict + verdictTrue boolean
- [ ] 💡 Why This Happens section (ALWAYS) — Hinglish technical explanation
- [ ] 🎭 Real-Life Analogy section (ALWAYS) — Hinglish relatable metaphor
- [ ] Status pill matches step type (success/fail/skip/loop/prune/info)
- [ ] All `why` and `analogy` text in Hinglish (NEVER pure English)
- [ ] Bold key terms with `<strong>`, code with `<code>`
- [ ] **Every important word, logic step, and formula** in why + analogy is wrapped: `<strong>concept / logic</strong>` (bold + highlight) and `<code>formula / dp[i][j]</code>` (for all cells, math, key expressions) — this is now mandatory for scannability and teaching value (refined from lc72)

### Log Preview popup

- [ ] `position:fixed; bottom:52px; right:14px; z-index:100`
- [ ] Width 480px default (wider for v2 schema)
- [ ] Max-height 75vh
- [ ] NO toolbar in popup — controls in footer only
- [ ] Shows ONLY current step's card
- [ ] Zoom via `--log-scale` (60%–180%, 20% steps). **Use `transform: scale(var(--log-scale)); transform-origin: bottom right;` on `.log-popup`** (base width/height) so the **entire popup box + width + height + font size + all internal content ("and so on") scale uniformly** together from the bottom-right corner. Footer zoom buttons must use `.btn-zoom` class (for − / + / ↺) and `.log-zoom-label` class on the percentage `<span>`. Fullscreen button uses `.btn btn-fullscreen`. Always call `updateLogZoom()` inside `showLogPreview()` to re-apply the scale when a new log appears.
- [ ] `.log-popup--hidden` for opacity+scale transition
- [ ] Fullscreen (`⛶`) uses `.btn btn-fullscreen` and targets `.col-viz` for request/exitFullscreen
- [ ] (lc72 is the reference implementation for correct log-preview zoom behavior.)

### Footer Bar

- [ ] Split layout: left 1/3 (under code) + right 2/3 (under viz)
- [ ] Left: step counter + action badge
- [ ] Right (in order): kbd hints + ↺ + ◀ + ▶ Next + sep + 📋 ON/OFF + − + 100% + + + ↺ + sep + ⛶
- [ ] Log zoom controls (− / + / ↺) use `.btn-zoom` class; the percentage uses `.log-zoom-label` class. Fullscreen (⛶) uses `.btn btn-fullscreen` class.
- [ ] All buttons `type="button"`
- [ ] ❌ NO Play, NO speed selector, NO progress bar
- [ ] (See Log Preview popup section for full zoom implementation details using --log-scale + transform.)

### Theme

- [ ] ❌ NO dark mode — light theme only
- [ ] Claude white aesthetic: warm off-white, clean borders, indigo accents
- [ ] No `@media(prefers-color-scheme:dark)` block
- [ ] Exception: `.log-code-snippet` uses GitHub dark (`#0d1117`) for code readability

### JS

- [ ] **🔴 RULE #10: ZERO-MISS LOGGING** — every single code line, every execution has complete log
- [ ] Every log has: 📜 actual code with values + 📥 inputs + 📤 outputs + ⚖️ verdict + 💡 Hinglish why + 🎭 Hinglish analogy
- [ ] IF true AND IF false both get separate log steps
- [ ] FOR/WHILE: each iteration + exit step all logged
- [ ] BREAK/RETURN/SKIP: each gets own step with explanation
- [ ] No vague logs — every log shows `L<N> — <actual code> = <value> → <verdict>`
- [ ] All `why` and `analogy` in Hinglish teacher-voice (NEVER robotic English)
- [ ] RULE #2: every line every execution
- [ ] `richLog()` helper present and used for every step
- [ ] `hlCode()` mini Python tokenizer for log card snippets
- [ ] `renderStep` calls `renderViz` + `showLogPreview`
- [ ] 1 resize handle (H1) with `syncFooterWidth()` in move handler
- [ ] `window.addEventListener('resize', ...)` — auto-adjusts layout
- [ ] Keyboard: ← → R
- [ ] State vars: `logEnabled`, `logZoomLevel`, `codeWidthUserSet`, `curResult`
- [ ] `toggleLogPreview()`, `toggleProblem()`, `toggleFullscreen()`

### Emoji Compliance (RULE #6 + #7)

- [ ] Zero `??` in source
- [ ] Zero `&#x` entities
- [ ] All buttons: `↺ ◀ ▶` (direct Unicode)
- [ ] H1 starts with thematic emoji
- [ ] LOG_BADGE / BADGE_LABEL use real emoji
- [ ] `.btn`, `.section-label`, `.log-status-pill`, `.log-section-title-icon` CSS include `system-ui, "Segoe UI Emoji"`

### What is NOT in this architecture (removed)

- [ ] ❌ Hinglish Explain button/modal
- [ ] ❌ Decision Trace panel
- [ ] ❌ Scrollable Step Log (Log Preview shows current only)
- [ ] ❌ Dark mode
- [ ] ❌ Play/Pause button
- [ ] ❌ Speed selector
- [ ] ❌ Progress bar
- [ ] ❌ Animated demo block
- [ ] ❌ Tree/Graph fullscreen modal
- [ ] ❌ Tree tooltip on hover
- [ ] ❌ Click popup (centered modal)
- [ ] ❌ Auto-popup
- [ ] ❌ Vertical resize handles
- [ ] ❌ resizeH2
- [ ] ❌ Log Preview toolbar inside popup
- [ ] ❌ v1 separate `log` / `detail` / `why` string fields (replaced by `logHTML` from `richLog()`)


---

## 🔧 Emoji & Symbol Fix Guide

### Button Symbol Standards

| Button | Symbol |
|---|---|
| Reset | `↺` |
| Previous | `◀` |
| Next | `▶` |
| Run | `▶ Run` |
| Close | `✕` |
| Fullscreen | `⛶` |
| Hide/Show | `▲` / `▼` |

> 🔴 **NEVER use `▴`/`▾`** (use `▲`/`▼`), **NEVER use `⟲`** (use `↺`)

### Badge Emoji Standards

| Action | Emoji | Action | Emoji |
|---|---|---|---|
| DEF/FUNC | 📌 | FOUND/DONE | 🎉 |
| INIT/SETUP | ⚙️ | RETURN/EXIT | 🏆 |
| FOR/LOOP | 🔁 | PUSH/ADD | ⬆ |
| TRUE/YES | ✅ | POP/REMOVE | ⬇ |
| FALSE/NO | ❌ | BACKTRACK/RESTORE | ↩ |
| BASE/LEAF | 🏁 | SORT/SWAP | 🔀 |
| CHECK/SCAN | 🔎 | UPDATE/CALC | ✏️ |
| CALL | 📞 | PRUNE/CUT | ✂️ |
| SKIP | ⏭ | EXIT | 🔚 |

### Status Pill Emoji

| Status | Emoji |
|---|---|
| Success | ✅ |
| Failed | ❌ |
| Skipped | ⏭ |
| Loop iter | 🔁 |
| Pruned | ✂️ |
| Info | ℹ️ |

### Log Section Title Icons

| Section | Emoji |
|---|---|
| Code Executing | 📜 |
| Input Values | 📥 |
| Output / State | 📤 |
| Verdict / Decision | ⚖️ |
| Why This Happens | 💡 |
| Real-Life Analogy | 🎭 |

### H1 / Problem Title Emoji by Type

| Keywords | Emoji |
|---|---|
| coin / change / knapsack | 🪙 |
| stair / climb / jump | 🪜 |
| path / graph / dfs / bfs | 🗺️ |
| string / word / match | 📝 |
| truck / greedy / load | 🚛 |
| tree / arrangement | 🎨 |
| palindrome / split | ✂️ |
| water / rain | 🌊 |
| expression / operator | 🎨 |
| grid / island | 🧱 |
| default | 🎯 |

### Pre-delivery Emoji Checklist

- [ ] Zero `??` anywhere
- [ ] Zero `&#x` HTML entities
- [ ] Buttons show `↺ ◀ ▶` (not `?`)
- [ ] H1 has thematic emoji
- [ ] LOG_BADGE uses real emoji
- [ ] Log section titles use proper icons (📜 📥 📤 ⚖️ 💡 🎭)
- [ ] Status pills use proper icons (✅ ❌ ⏭ 🔁 ✂️ ℹ️)
- [ ] CSS includes `system-ui` font fallback on symbol elements

---

## 📚 Reference Implementations

| LC# | Problem | Highlights |
|---|---|---|
| 42 | Trapping Rain Water | Two pointers, water-area shaded regions, height array |
| 130 | Surrounded Regions | DFS iterative stack, board grid + stack + safe set viz-row |
| 239 | Sliding Window Maximum | **UI / width / section layout + output handling reference** (lc239 canonical): 750px .viz-wrap + .viz-sec stacking, deque front (left) / back (right) split, plain `Output (Max of each window)` heading **without ✅ tick**, progressive output list + placeholder messages where applicable. Never pre-show final answer in the Output viz panel. |
| 282 | Expression Add Operators | Backtracking + recursion stack viz + 6-section logs |
| 498 | Diagonal Traverse | **V9 traversal reference** — dark matrix grid + SVG flow arrows (red/purple/amber) |
| 691 | Stickers to Spell Word | Backtracking + memoization with recursion stack viz; follows lc239 UI patterns (750px .viz-wrap + stacked .viz-sec, plain `Output (Min Stickers)` heading **without ✅ tick**, placeholder until last step, correct test chips per strict two-row rule). Never use "Final Result" or tick-prefixed heading for the output viz section. |
| 72  | Edit Distance | **DP recurrence arrows reference** — small clean arrows (dashed candidates + solid green chosen with halo), live computation from snap.dp, high z-index + draw order, integrated caption. Also **log preview zoom reference** (full box + content scaling via transform + --log-scale, correct button classes + update call in showLogPreview). |
| 1568 | Min Days to Disconnect Island | Nested DFS, grid + helper-call counter, multi-phase |

*Add more as built.*

lc72 is the canonical example for **DP recurrence arrows** (showing the 3 predecessors + chosen min with small, high-visibility arrows + halo).

---

## 🚀 Future Enhancements (Roadmap)

- [ ] Memoization variant — show DP cache state alongside recursion tree
- [ ] SVG arithmetic tree for expression problems
- [ ] Graph layout with auto force-directed positioning
- [ ] Heatmap mode for 2D DP — color by value gradient
- [ ] Compare mode — split-screen for two algorithm variants
- [ ] Export to GIF — auto-step playback recorder
- [ ] Voice narration in Hinglish (Web Speech API)
- [ ] Code annotation overlay — pointer arrows on active line
- [ ] Step bookmarks — save interesting frames for later review
- [ ] Branch tree visualizer — show all backtracking branches as collapsible tree

---

## 🆕 What's new in v2.5 (vs v2.4)

| Section | Change |
|---|---|
| **Hinglish Writing Style** 🆕 | Added "Log-specific rules" subsection (under Hinglish Writing Style) that captures: concise (short but complete) why/analogy text + **mandatory** per-term highlighting using `<strong>` for every key logic/concept/operation and `<code>` for every formula, cell ref, variable, math (e.g. <code>dp[i][j]</code>, min, +1, insert/delete/replace). The requested Hindi phrase directing shorter logs was removed (per user instruction) but **all other guidance on conciseness + full highlighting rules + examples were preserved/restored exactly**. Good examples in the section now demonstrate the required `<strong>`/`<code>` markup. |
| **Pre-delivery Master Checklist** | Reinforced the Log Card v2 bullet: "Every important word, logic step, and formula in why + analogy is wrapped..." (already present; now cross-linked to the new log-specific rules). |
| **Changelog + version** | New v2.5 entry + frontmatter already at 2.5. lc72 remains the reference for highlighted concise logs + DP arrows + log-popup zoom. |
| **Viz Output heading rule (user feedback)** | Clarified in Pre-delivery checklist + Reference table: Output section `.viz-sec-head` must use **plain "Output (…)" text with zero ✅ green tick emoji** (never "✅ Output", "✅ Final Result"). lc239 and lc691 updated to match (lc239 heading fixed to plain version). Placeholder-until-last-step rule for final scalar results also reinforced. |

## 🆕 What's new in v2.4 (vs v2.3)

| Section | Change |
|---|---|
| **V9 + new DP Recurrence Arrows** 🆕 | Added detailed best practices for DP table recurrence/dependency arrows (small compact markers, candidate vs chosen + halo, live computation from snap.dp, z-index 100 + draw order, integrated explanatory caption). lc72 is now the canonical reference. |
| **Log Preview zoom** 🆕 | Full uniform scaling (width + height + font size + all content) of the log popup via `transform: scale(var(--log-scale))` + `transform-origin: bottom right` on `.log-popup` (base width/height kept). Footer buttons use `.btn-zoom` / `.log-zoom-label` / `.btn btn-fullscreen`. `showLogPreview` must call `updateLogZoom()`. Internal fonts can use `calc()` with the var. (See updated Log Preview + Footer + JS sections.) lc72 demonstrates the correct implementation for both DP arrows and log zoom. |

## 🆕 What's new in v2.3 (vs v2.2)

| Section | Change |
|---|---|
| **Test Chip Values Rule** 🆕🔴 | Test case chips MUST show actual parameter values (real numbers/strings/arrays), never abstract labels. Strict two-row chip pattern (exactly as in lc239/lc691):
- Top row: `.pt-tag` + **descriptive label** (e.g. "Standard", "Impossible", "Trivial") + `.tc-exp` containing `exp: value` (never put the exp value in the middle label).
- Bottom row (`.tc-vals`): actual input values as `<code>` pills (e.g. `<code>stickers=[...]</code> <code>target="..."</code>` or raw array for simple cases). For arrays, prefer clean representation matching lc239. |

## 🆕 What's new in v2.2 (vs v2.1)

| Section | Change |
|---|---|
| **V9 — Grid Flow Arrows** 🆕 | New visualization rule: matrix/grid problems MUST use SVG arrow overlay inside grid; full CSS+JS pattern documented with lc498 as reference |
| **Reference Implementations** | Added lc498 Diagonal Traverse as V9 canonical example |

## 🆕 What's new in v2.1 (vs v2.0)

| Section | Change |
|---|---|
| **Frontmatter** | Added `version: 2.1`, `last_updated`, mention of analogy in description |
| **CRITICAL RULES #8** 🆕 | Every log card MUST follow 6-section schema (richLog) |
| **CRITICAL RULES #9** 🆕 | File name header at top of every response |
| **CRITICAL RULES #10** 🆕🔴 | ZERO-MISS LOGGING — sabse important rule, ek bhi line miss nahi honi chahiye |
| **File naming convention** 🆕 | New section — patterns + rules for `lc<NNN>-<slug>.html` |
| **Output format** 🆕 | New section — static note + file header + code block + recap + follow-up |
| **Enhanced Log Card Schema v2** 🆕 | Full new section — richLog API, visual layout, full implementation, CSS |
| **Status Pills** 🆕 | New section — 6 status types with usage rules |
| **Hinglish Writing Style** 🆕 | New section — examples, vocabulary cheat-sheet, style rules |
| **Step object schema** | Updated — single `logHTML` field replaces v1's `log/detail/why` strings |
| **Log popup CSS** | Updated — width 480px (was 420px), max-height 75vh (was 65vh), no body padding |
| **Log Section Title Icons** 🆕 | Emoji guide for 📜 📥 📤 ⚖️ 💡 🎭 |
| **Reference Implementations** 🆕 | Track of which problems have been built |
| **Roadmap** 🆕 | Future enhancement ideas |

---

*End of Skill Document v2.5. This is a living document — update Reference Implementations and Roadmap as new visualizers are built and patterns emerge.*

---

## 🔴🔴🔴 RULE #10 — ZERO LINES MISSED + EVERY EXECUTION LOGGED (THE GOLDEN RULE)

> **⚠️ THIS IS THE SINGLE MOST IMPORTANT RULE. EVERY OTHER RULE EXISTS TO SERVE THIS ONE.**

### ✅ Mandatory checklist — EVERY line, EVERY execution

- [ ] **🔴 Ek bhi line miss nahi honi chahiye** — har code line jo Python interpreter touch karta hai, uska apna dedicated step + log MUST exist
- [ ] **🔴 Har execution type ka log hona chahiye** — success ho, fail ho, skip ho, loop ho, error ho, prune ho, ya koi bhi aur condition — sab logged
- [ ] **🔴 Har log mein ACTUAL CODE hona chahiye** — jo line execute ho rahi hai, uska exact Python code with concrete values substituted (e.g., `for i in range(2, 5): i=3`, NOT "loop iteration")
- [ ] **🔴 Har log mein RESPONSE VALUES hona chahiye** — line execute hone ke baad kya hua, kaunsi variable change hui, kya return hua, sab dikhao
- [ ] **🔴 Har log mein VERDICT hona chahiye** — line SUCCESS hui ya FAIL ya SKIP — clear status pill + boolean result for conditions
- [ ] **🔴 Har log mein HINGLISH WHY hona chahiye** — kyun yeh hua, technical reason Hinglish mein (e.g., "Yeh O border tak connected nahi hai, isliye surrounded region ka part hai")
- [ ] **🔴 Har log mein HINGLISH ANALOGY hona chahiye** — real-life metaphor Hinglish mein (e.g., "Jaise bank cheque pe leading zero invalid hota hai — '0123' reject!")

### ❌ Anti-patterns — STRICT VIOLATIONS

- [ ] ❌ **NEVER skip a line** — even if it's just `return` or `break` or a simple assignment, it MUST have its own step
- [ ] ❌ **NEVER merge lines** — `L2-4` is WRONG. Each line = own step, own log
- [ ] ❌ **NEVER write vague logs** — "checking condition" is WRONG. Write `if 3 > 2 and num[2]=='0' → False`
- [ ] ❌ **NEVER skip failed conditions** — if an `if` evaluates FALSE, that FALSE step is REQUIRED with status `skip`
- [ ] ❌ **NEVER skip loop exits** — when `for`/`while` terminates, the EXIT step is REQUIRED with status `info`
- [ ] ❌ **NEVER write pure English** — every `why` and `analogy` MUST be Hinglish teacher-voice
- [ ] ❌ **NEVER omit values** — log must show `curr=34, value=12+34=46` not just "value updated"

### 📋 Per-line log format checklist

Every single step MUST contain ALL of these:

```
┌─────────────────────────────────────────────────┐
│ ✅ 1. BADGE + LINE REF + STATUS PILL            │
│    [🔁 FOR] [L9] .............. [🔁 Loop iter]  │
│                                                  │
│ ✅ 2. 📜 ACTUAL CODE WITH VALUES                 │
│    for i in range(2, 5): i=3                     │
│    ← NOT "loop iteration", NOT "checking next"   │
│                                                  │
│ ✅ 3. 📥 INPUT VALUES (what line reads)           │
│    [index=2] [n=5] [num="12345"]                 │
│                                                  │
│ ✅ 4. 📤 OUTPUT VALUES (what changed)             │
│    [curr_str="34"] [curr=34] [i=3]               │
│    ← with changed:true flag for mutations        │
│                                                  │
│ ✅ 5. ⚖️ VERDICT (for conditions)                │
│    ✓ i (3) < n (5) → True                       │
│    ✗ value (12) == target (6) → False            │
│                                                  │
│ ✅ 6. 💡 HINGLISH WHY                            │
│    "i=3 pe try kar rahe hain. Operand banega     │
│     num[2:4] = '34'. Yeh valid hai kyunki        │
│     num[2]='3' jo '0' nahi hai."                 │
│                                                  │
│ ✅ 7. 🎭 HINGLISH ANALOGY                        │
│    "Jaise pizza ke slices decide karte ho —      │
│     2 slice loge ya 3? Har option try karo."     │
└─────────────────────────────────────────────────┘
```

### 📊 Every execution type → required log content

| Execution type | Status pill | Code shows | Values show | Verdict | Why (Hinglish) | Analogy (Hinglish) |
|---|---|---|---|---|---|---|
| **Line succeeds** | ✅ Success | exact code + result | all changed vars | ✓ (if condition) | kyun succeed hua | real-life success metaphor |
| **Line fails / condition false** | ❌ Failed / ⏭ Skip | exact code + both sides | inputs that were checked | ✗ with both values | kyun fail hua, kya galat tha | real-life rejection metaphor |
| **Loop iteration** | 🔁 Loop | `for i in range(a,b): i=X` | loop var + any mutations | loop bounds | kyun yeh iteration zaroori hai | real-life repetition metaphor |
| **Loop EXIT (natural)** | ℹ️ Info | `for loop exit: i reached N` | final loop var value | range exhausted | kyun loop khatam hua | real-life completion metaphor |
| **Break / Prune** | ✂️ Pruned | `break` + triggering condition | what caused the break | prune condition | kyun cut kiya, kya invalid tha | real-life shortcut metaphor |
| **Skip (else not taken)** | ⏭ Skip | the condition that was false | both sides of comparison | ✗ False | kyun yeh branch skip hua | real-life "nahi lena" metaphor |
| **Function call** | ℹ️ Info | `func(arg1, arg2, ...)` | all arguments with values | — | kyun yeh call zaroori hai | real-life delegation metaphor |
| **Function return** | 🏆 Return | `return VALUE` | what's being returned | — | kya result aaya | real-life "answer de do" metaphor |
| **Assignment** | ✅ Success | `var = expression = VALUE` | before → after | — | kyun yeh value set ki | real-life "likhna" metaphor |
| **Found result** | 🎉 Found | `res.append(expr)` | the found value + list size | match condition | kyun yeh valid answer hai | real-life "jackpot!" metaphor |

### 🔴 Self-verification before delivery

Before delivering ANY visualizer, verify:

- [ ] Count total code lines → count total unique `line` values in steps → **MUST match** (every line appears at least once)
- [ ] Count `if` statements in code → count steps with `action:"if-check"` → **MUST have both TRUE and FALSE** steps for each `if`
- [ ] Count `for`/`while` loops → each MUST have iteration steps AND an EXIT step
- [ ] Count `return` statements → each MUST have its own step
- [ ] Count `break`/`continue` → each MUST have its own step
- [ ] Grep all steps for empty `why` → **MUST be zero** (every step has Hinglish explanation)
- [ ] Grep all steps for empty `analogy` or `code` → **MUST be zero**
- [ ] Spot-check 5 random steps → each shows actual Python code with concrete values, NOT vague descriptions


