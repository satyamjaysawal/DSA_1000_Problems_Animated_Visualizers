# GT Learn — Naming & Structure Convention

## Folder Layout

```
Big_DSA_Problem/
├── 00_HomePage/
│   ├── gt_learn_homepage.html
│   ├── coding-problems.json
│   └── NAMING_CONVENTION.md
├── arrays/
├── dp/
├── graphs/
├── greedy/
├── backtrack/
├── sorting/
├── heap/
├── math/
└── misc/
```

All visualizer HTML files live exactly **one level deep** inside a category folder. Never at root.

---

## HTML File Naming

```
lc[number]-[slug].html
lc[number]-[slug]-[approach].html
```

Approach suffix only when multiple versions exist for the same problem.

### Platform Prefixes

| Prefix | Platform      |
|--------|---------------|
| `lc`   | LeetCode      |
| `gfg`  | GeeksForGeeks |
| `hr`   | HackerRank    |
| `cf`   | Codeforces    |

---

## coding-problems.json — Structure

Single dictionary with unified keys:

```json
{
  "categories": {
    "dp": { "name": "Dynamic Programming", "icon": "🧮", "color": "#2dd4bf", "short": "DP" }
  },
  "platforms": {
    "lc": { "name": "LeetCode", "icon": "🟡", "color": "#f89f1b" }
  },
  "paradigms": {
    "Dynamic Programming": { "icon": "🧮", "color": "#f472b6", "desc": "...", "complexity": "...", "examples": "..." }
  },
  "sites": {
    "LeetCode": { "icon": "🟠", "desc": "...", "type": "Problems", "url": "https://leetcode.com" }
  },
  "problems": [ ... ]
}
```

### Problem Entry Format

```json
{
  "id": 1,
  "name": "Coin Change",
  "diff": "Medium",
  "cat": "dp",
  "paradigm": "Dynamic Programming",
  "lcNum": 322,
  "plat": "lc",
  "url": "https://leetcode.com/problems/coin-change/",
  "viz": "../dp/lc322-coin-change-dp-visualizer.html"
}
```

| Field      | Required  | Notes                                      |
|------------|-----------|--------------------------------------------|
| `id`       | yes       | Unique integer                             |
| `name`     | yes       | Exact problem title                        |
| `diff`     | yes       | `"Easy"` / `"Medium"` / `"Hard"`           |
| `cat`      | yes       | Matches key in `categories` dict           |
| `paradigm` | yes       | Matches key in `paradigms` dict            |
| `lcNum`    | LC only   | Integer e.g. `322`                         |
| `plat`     | yes       | Platform key: `lc`, `gfg`, `hr`, `cf`, `custom` |
| `url`      | yes       | Direct problem URL                         |
| `viz`      | if exists | Relative path `../[cat]/[file].html`       |

The `viz` field links a problem to its HTML visualizer.
The `⚡` indicator and clickable name on the homepage are driven by this field.

---

## Adding a New Visualizer — Checklist

- [ ] HTML file placed in correct category subfolder
- [ ] File named `lc[num]-[slug].html`
- [ ] GT LEARN home button present (see below)
- [ ] `viz` field added to matching entry in `coding-problems.json`
- [ ] `viz` path is `../[category]/[filename].html`

---

## GT LEARN Home Button

Every HTML visualizer must have this button:

```html
<a href="../00_HomePage/gt_learn_homepage.html" class="gt-home-btn">GT LEARN</a>
```

Path is always `../00_HomePage/gt_learn_homepage.html` — all visualizer files are one level deep.
