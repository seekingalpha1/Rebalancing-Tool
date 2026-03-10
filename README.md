# ETF Rebalancing Tool

A lightweight, single-file web app that calculates a **fixed monthly savings plan** for your ETF portfolio — no server, no install, runs entirely in the browser.

## Features

- **Fixed savings plan**: Computes a constant monthly contribution per ETF that gradually rebalances your portfolio toward your target allocation over the selected time horizon
- **Categories with sub-splits**: Assign multiple ETFs to the same category with a relative ratio (e.g. 90 / 10 for MSCI World + Emerging Markets) — the tool normalizes automatically
- **Multi-broker support**: Track the same ETF across multiple brokers by adding a broker/account label
- **Current vs. target view**: Visual bar chart showing how far each category is from its target
- **Simulation**: Month-by-month table showing how portfolio allocation and deviation evolves
- **Dark mode**: Automatically follows your device's light/dark preference
- **Mobile-friendly**: Numeric keyboard on mobile for number fields, responsive layout
- **Export / Import**: Save and restore your portfolio as a JSON file
- **Local persistence**: Saves automatically to browser localStorage

## How it works

1. **Define categories** — e.g. *World 70 %, Quality 15 %, Momentum 15 %*. They must sum to 100 %.
2. **Add ETFs** — enter name, broker (optional), current portfolio value, category, and ratio within the category.
   - Ratio is relative: entering `90` and `10` for two ETFs is identical to `9` and `1` — the tool normalizes automatically.
   - Same ETF across multiple brokers? Add two rows, one per broker.
3. **Set your monthly savings rate** and the simulation horizon (months).
4. Click **Calculate Allocation** — the tool computes how to split the monthly savings across ETFs so that the portfolio converges to the target over the chosen horizon.

## Example portfolio

| ETF | Broker | Category | Ratio |
|-----|--------|----------|-------|
| iShares MSCI World | Broker A | World (70 %) | 90 |
| iShares MSCI Emerging Markets | Broker A | World (70 %) | 10 |
| iShares MSCI Quality | Broker A | Quality (15 %) | 1 |
| iShares MSCI Quality | Broker B | Quality (15 %) | 1 |
| Xtrackers MSCI Momentum | Broker A | Momentum (15 %) | 1 |
| iShares MSCI Momentum | Broker B | Momentum (15 %) | 1 |

Within *World*, MSCI World receives 90 % of the World allocation and Emerging Markets receives 10 %. Both Quality ETFs share the Quality allocation equally (50/50).

## Algorithm

For a given monthly savings `S` and simulation horizon `N` months:

1. Compute the **target final value** of each ETF: `targetValue[i] = globalFraction[i] × (currentTotal + N × S)`
2. Compute **investment needed**: `needed[i] = max(0, targetValue[i] − currentValue[i])`  (no selling)
3. **Normalize**: `monthlyContribution[i] = (needed[i] / sum(needed)) × S`

This results in a **constant** monthly amount per ETF that is the same every month.

## Usage

Open `index.html` in any modern browser — no build step required.
