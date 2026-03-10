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
            categoryId, splitPct,                 // 0 = direct to category; >0 = % within synthetic pair
            savingsEnabled }],                    // false = ETF held constant, excluded from optimizer
  settings: { monthlySavings, simulationMonths }
}
```

- `localStorage` key: `etf-rebalancing-v3`
- Migration shim in `migrateEtfs()` handles v1 (`weight`), v2, and v3 (`ratio`) exports

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

## "Other" Category — Min-Scale Formula

ETFs with `splitPct > 0` form a **synthetic pair** (e.g. MSCI World 90% + EM 10% → synthetic ACWI).
ETFs with the same name in the same category are auto-grouped (handles multi-broker positions).

```
combinedValue[nameGroup] = sum of values of all ETFs with that name in the category
scale = min over name-groups of (combinedValue[G] / (splitPct[G] / 100))
syntheticValue = scale
Other from category = Σ max(0, combinedValue[G] − splitPct[G]/100 × scale)
effectiveCatValue = directTotal + syntheticValue
```

Target for "Other" = 0%. The optimizer routes savings to the underweight synthetic ETF to drive Other → 0.
Computed in `computeOtherInfo()`. Global fractions computed in `computeGlobalTargets()`.

## Key Functions

| Function | Purpose |
|---|---|
| `solveOptimalSavings()` | QP water-filling optimizer |
| `computeGlobalTargets()` | ETF global target fractions (direct vs. synthetic per min-scale) |
| `computeOtherInfo()` | Min-scale synthetic imbalance → "Other" bucket |
| `syntheticNameGroups()` | Groups synthetic ETFs by name for min-scale calculation |
| `simulate()` | Runs N-month simulation with constant allocation |
| `renderSetup()` | Re-renders all three setup sections |
| `renderResults()` | Renders stats, cards, allocation bars, charts, sim table |
| `migrateEtfs()` | Migrates old state formats (v1/v2/v3 → v4) |

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
| v4.0 | `etf-rebalancing-v3` | Replace `ratio` with `splitPct`; min-scale synthetic-ACWI formula; multi-broker name-grouping |
