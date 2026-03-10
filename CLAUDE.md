# ETF Rebalancing Tool — CLAUDE.md

## Project Overview

Single-file web app (`index.html`) — no build step, no dependencies except Chart.js (CDN). Opens directly in any browser.

## Architecture

Everything lives in `index.html`:
- **CSS** (lines ~1–200): CSS variables for theming (light + dark mode via `prefers-color-scheme`), responsive layout
- **HTML** (lines ~200–250): Two-panel layout — setup panel (left) + results panel (right)
- **JavaScript** (lines ~250–end): All state, logic, and rendering

## State Model

```js
state = {
  categories: [{ id, name, targetPct }],         // sum of targetPct must equal 100
  etfs: [{ id, name, broker, currentValue,        // broker is optional label
            categoryId, ratio,                    // ratio = relative weight within category (auto-normalized)
            savingsEnabled }],                    // false = ETF held constant, excluded from optimizer
  settings: { monthlySavings, simulationMonths }
}
```

- `localStorage` key: `etf-rebalancing-v3`
- Migration shim in `migrateEtfs()` handles v1 (`weight`) and v2 exports

## Optimization Algorithm

**Quadratic Programming via Water-Filling (Active-Set)**

Finds constant monthly savings `x[i] ≥ 0` per active ETF that:
1. Sum to `monthlySavings`
2. Minimize `Σ (currentValue[i] + N·x[i] − globalFraction[i] · FutureTotal)²`

Solved analytically (KKT conditions → iterative active-set):
```
a[i] = (currentValue[i] - globalFraction[i] * FutureTotal) / N
mu   = (savings + Σ a[active]) / |active|
x[i] = mu - a[i]   →  clamp negatives to 0, remove from active set, repeat
```

Key functions: `solveOptimalSavings()`, `computeGlobalTargets()`, `simulate()`

## "Other" Category

For categories with multiple ETFs, the within-category ratio imbalance is computed as:
```
Other = Σᵢ max(0, actualValue[i] − normalizedRatio[i] · totalCategoryValue)
```
This represents e.g. the EM surplus in a 90/10 MSCI World + EM split. Target = 0%. The optimizer naturally drives Other toward 0 by routing savings to the underweight ETF. Computed in `computeOtherInfo()`.

## Key Functions

| Function | Purpose |
|---|---|
| `solveOptimalSavings()` | QP water-filling optimizer |
| `computeGlobalTargets()` | ETF global target fractions (category% × normalized ratio) |
| `computeOtherInfo()` | Within-category ratio imbalance → "Other" bucket |
| `simulate()` | Runs N-month simulation with constant allocation |
| `normalizedWeights()` | Normalizes ETF ratios within each category to 0–100% |
| `renderSetup()` | Re-renders all three setup sections |
| `renderResults()` | Renders stats, cards, allocation bars, charts, sim table |
| `migrateEtfs()` | Migrates old state formats (v1/v2 → v3) |

## Design Conventions

- **No framework** — plain ES5-compatible JS (no arrow functions in state-critical paths, `var` throughout)
- **CSS variables** for all colors; dark mode via `@media (prefers-color-scheme: dark)` on `:root`
- `inputmode="decimal"` on all numeric inputs (mobile numeric keyboard)
- Font-size ≥ 16px on inputs (prevents iOS auto-zoom)
- `escHtml()` used on all user-supplied strings in HTML output

## Files

| File | Description |
|---|---|
| `index.html` | Complete app (CSS + HTML + JS) |
| `README.md` | User-facing documentation |
| `CLAUDE.md` | This file — developer/AI context |

## Version History

| Version | LS Key | Notable changes |
|---|---|---|
| v1.0 | `etf-rebalancing-v1` | Initial: categories, ETFs with `weight` field |
| v2.0 | `etf-rebalancing-v2` | Dark mode, `ratio` field, broker field, constant savings |
| v3.0 | `etf-rebalancing-v3` | QP optimizer, `savingsEnabled` checkbox, "Other" category |
