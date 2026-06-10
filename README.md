# GT Learn

> **GT Learn** — Interactive DSA visualizers + LeetCode / coding problems tracker with progress, filters, paradigms, and platform links. Built as self-contained HTML files.

**🚀 Live Demo:** [https://dsa-1000-visualizers.vercel.app](https://dsa-1000-visualizers.vercel.app)

PR preview deployments are enabled via Vercel (connect Git in dashboard for auto previews on every PR).

**1243+ problems** tracked across 15+ categories, multiple platforms (LeetCode, GFG, HackerRank, Codeforces, InterviewBit, and custom).

## 🚀 Quick Start (Clean Home)

Open the homepage directly:

- Double-click `index.html` (works offline for the tracker UI + localStorage progress)
- Recommended: `npx serve .` or `python -m http.server 8000` then visit http://localhost:8000

The root `index.html` is the full-featured **GT Learn Coding Problems Tracker** (sourced from `00_HomePage/gt_learn_homepage.html` with corrected asset paths).

## ✨ Features

- 📊 Live stats header (Total / Solved / Easy-Med-Hard / Viz count / % progress)
- 🔍 Powerful filters: search, difficulty, category (sidebar progress), paradigm, platform, solved/hidden/visualizer-only
- ✅ Persistent solved checkboxes (localStorage)
- 🧠 Problem-Solving Paradigms explorer (click rows to filter)
- 📚 Curated learning resources / platform quick links
- ⚡ Many problems link to dedicated interactive visualizers (step-by-step, Hinglish explanations in some)
- 🌗 Light/Dark theme toggle
- 📱 Responsive (mobile hides some columns)

## 📁 Project Structure

```
.
├── index.html                 # ← Main home (tracker). Open this.
├── 00_HomePage/
│   ├── gt_learn_homepage.html # Source homepage (also works if opened directly)
│   ├── coding-problems.json   # All problem metadata, categories, platforms, paradigms
│   └── NAMING_CONVENTION.md
├── arrays/, dp/, graphs/, ... # Individual problem pages + visualizers (lc###-name.html)
├── misc/, sorting/, ...       # More categories
└── README.md
```

All visualizer/problem HTML files are standalone (open directly in browser).

## 🛠️ Development Notes

- The tracker is a single HTML + JSON (no build step).
- Progress is saved per-browser via `localStorage` key `gt_s`.
- Filters persist in `gt_f`.
- To contribute new visualizers or problems, see patterns in existing files under category folders and update `00_HomePage/coding-problems.json`.
- Naming convention details in `00_HomePage/NAMING_CONVENTION.md`.

## 📖 Usage

1. Open `index.html`
2. Use sidebar categories or top filters to explore
3. Click ⚡ visualizer links (where available) to open interactive step-throughs
4. Check off problems as you solve — your progress sticks across refreshes
5. Click paradigm rows for focused practice on specific techniques

## 🙌 Credits

Built with Tailwind (CDN), vanilla JS, and ❤️ for clean learning tools.

---

**Live URL idea (GitHub Pages / Vercel):** point to root — `index.html` gives the clean experience without subfolder redirect.

PRs welcome for new visualizers, more problems, or UI improvements!
