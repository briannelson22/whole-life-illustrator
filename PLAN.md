# Whole Life Illustrator — Project Plan

## Overview

An interactive, single-page web application that compares the rate of return of a Guardian Whole Life Policy to different investments since 1970.

The other investments will be:
1. S & P 500
2. BND Total Bond or equivalent
3. 6 month CD
4. High Yield Savings Account


a Guardian Whole Life Insurance policy over any 30-year window using Guardian's actual published Dividend Interest Rates (DIR) from 1989 to present.

**Audience:** Personal use / research  
**Stack:** HTML + CSS + JavaScript (vanilla), Chart.js for visualizations  
**Deployment:** Static site (GitHub Pages or any static host)

---

## Goals

1. Let the user input policy parameters (annual premium, initial death benefit).
2. Let the user select any valid 30-year start year (1970–1996, giving full 30-year windows through 2026).
3. Compute and display four key policy metrics over the 30-year period using Guardian's historical DIR data.
4. Clearly communicate the distinction between DIR and personal rate of return.

---

## Data Sources

### Guardian Whole Life
Guardian's published Dividend Interest Rates (DIR), sourced from:
- NFP Historical Whole Life Dividends table (1989–2016)
- topwholelife.com annual updates (2017–2025)
- Guardian press releases (2023–2026)

| Year | DIR (%) | Year | DIR (%) | Year | DIR (%) |
|------|---------|------|---------|------|---------|
| 1989 | 11.50 | 2002 | 8.00 | 2015 | 6.05 |
| 1990 | 11.00 | 2003 | 7.00 | 2016 | 6.05 |
| 1991 | 10.50 | 2004 | 6.60 | 2017 | 5.85 |
| 1992 | 10.25 | 2005 | 6.75 | 2018 | 5.85 |
| 1993 | 9.75  | 2006 | 6.50 | 2019 | 5.85 |
| 1994 | 9.00  | 2007 | 6.75 | 2020 | 5.65 |
| 1995 | 8.50  | 2008 | 7.25 | 2021 | 5.65 |
| 1996 | 8.00  | 2009 | 7.30 | 2022 | 5.65 |
| 1997 | 8.50  | 2010 | 7.00 | 2023 | 5.75 |
| 1998 | 8.75  | 2011 | 6.85 | 2024 | 5.90 |
| 1999 | 8.75  | 2012 | 6.95 | 2025 | 6.10 |
| 2000 | 8.50  | 2013 | 6.65 | 2026 | 6.25 |
| 2001 | 8.50  | 2014 | 6.25 |      |      |

Future years beyond 2026 default to 6.10% (recent average) for projections.

---

## Policy Model

### Inputs
- `annualPremium` — annual level premium (default: $5,000)
- `initialDeathBenefit` — face amount at policy issue (default: $500,000)
- `startYear` — policy issue year (range: 1989–1996)

### Cash Value Calculation
Cash value grows year-over-year using DIR applied to accumulated value, net of load factors:

```
loadFactor = 35% (year 1), 10% (years 2–5), 5% (years 6+)
netPremium = annualPremium × (1 - loadFactor)
CV[t] = (CV[t-1] + netPremium[t]) × (1 + DIR[t])
```

> Note: Load factors are approximations. Actual Guardian policy loads depend on age, health class, and product design.

### IRR Calculation
Internal rate of return at year `t` is the discount rate `r` such that:

```
NPV = 0 = -P×(1 + 1/(1+r) + ... + 1/(1+r)^t) + CV[t]/(1+r)^t
```

Solved via bisection over [-50%, +200%].

### Death Benefit Growth (Paid-Up Additions)
Each year's dividend partially purchases paid-up additional insurance:

```
PUA[t] = CV[t-1] × DIR[t] × 0.85
DB[t] = DB[t-1] + PUA[t] + annualPremium × 0.02
```

### Dividend Accumulation
Dividends are reinvested and compound at half the DIR rate for simplicity:

```
yearDiv[t] = CV[t-1] × DIR[t]
totalDivAcc[t] = totalDivAcc[t-1] + yearDiv[t] × (1 + DIR[t] × 0.5)
```

---

## Features

### Charts (Chart.js, tabbed)
1. **IRR over time** — annualized return from year 1–30 as cash value builds
2. **Cash value vs premiums paid** — shows break-even crossover point
3. **Death benefit growth** — initial DB + PUAs over 30 years
4. **Dividend accumulation** — reinvested dividend pool growth

### Summary Metrics (always visible)
- 30-year IRR
- Year-30 cash value (vs total premiums paid)
- Year-30 death benefit
- Total accumulated dividends

### Controls
- Annual premium input
- Initial death benefit input
- Start year slider (1989–1996, showing the full 30-year window label)
- Average DIR badge updates dynamically with each window

---

## File Structure

```
whole-life-illustrator/
├── index.html          # Single-file app (HTML + CSS + JS)
├── PLAN.md             # This file
└── README.md           # Usage notes and data attribution
```

---

## Suggested Future Enhancements

- [ ] S&P 500 CAGR overlay for the same 30-year window (apples-to-apples premium comparison)
- [ ] Year-by-year data table (exportable to CSV)
- [ ] Age-at-issue input affecting load factors and guaranteed cash value curve
- [ ] Health class selector (Preferred / Standard) affecting base CV
- [ ] Comparison mode: two start years side by side
- [ ] "What if DIR stays flat" scenario toggle

---

## Disclaimers

- DIR ≠ personal rate of return. The DIR applies to accumulated cash value, not to premiums paid.
- Load factors used here are approximations; actual Guardian illustrations will differ.
- This tool is for personal research only and does not constitute financial or insurance advice.
- Guardian dividends are not guaranteed, though Guardian has paid them every year since 1868.
- Data sources: NFP Historical Whole Life Dividends, topwholelife.com, Guardian press releases.
