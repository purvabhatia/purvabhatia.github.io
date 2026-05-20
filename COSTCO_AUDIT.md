# Costco DCF Workbook — Audit Findings

**Source file:** `assets/Costco_DCF.xlsx`
**Audit method:** Paired formula + computed-value extraction of every non-empty cell across all 12 sheets, then manual review.
**Status:** Read-only review — no changes made to the workbook.

---

## Summary

| Category | Count |
|---|---|
| Critical math errors that change outputs | 6 |
| Moderate methodological / consistency issues | 8 |
| Minor labeling / hygiene / quirks | 10 |
| **Total findings** | **24** |

### Top three to fix first

1. **C1 (EPS double-discount)** — single-cell fix, but it nearly doubles the Method 3 target price and changes the recommendation narrative.
2. **C4 (BS-CF reconciliation)** — fundamental two-statement consistency; the model fails the "does it balance" check.
3. **C2 (IS net sales not linked to ETS Base Case)** — the entire DCF rests on revenue projections that disagree with the ETS analysis the workbook claims to use.

---

## 🔴 CRITICAL — likely changes conclusions

### C1. EPS-for-target-price applies an extra (1 − 27%) haircut to net income

**Sheet:** `Valuation DCF`, cell **B87**

```
B87 = Forecast!K22 * (1-27%) / 443.87   →   $13.4569
```

Net income from `Forecast!K22` is already after-tax ($8,182M). Multiplying again by `(1-27%)` reduces it a second time. The 27% appears to be the dividend payout ratio confused for a tax rate. Correct EPS NTM = $8,182M / 443.87M shares = **$18.43** (matches the FY25 reported diluted EPS of $18.21 trajectory).

**Impact on Method 3:** Target Price = EPS × Industry P/E
- As-built: $13.46 × 32 = **$430.62**
- Corrected: $18.43 × 32 = **~$590**

This roughly halves the apparent "discount to market" implied by Method 3 and weakens the SELL framing.

---

### C2. Forecast IS net sales do NOT come from the ETS Base Case — they use a flat hardcoded CAGR

**Sheet:** `Forecast`, cells **K7:N7** (and dependent membership fees K8:N8)

```
K7 = J7 * (1 + 'Revenue Forecast ETS'!$N$29)   →   291,262
L7 = K7 * (1 + $N$29)                          →   314,301
M7 = L7 * (1 + $N$29)                          →   339,162
```

`ETS!N29` = **0.0791** hardcoded (not a computed CAGR). Meanwhile the ETS Base Case net sales (`ETS!N27:Q27`) are 284,369 / 307,105 / 329,840 / 358,201. So the IS uses growth-rate compounding, not the actual ETS forecast values. **The Forecast IS net sales for 2026 is ~$7B higher than the ETS Base Case net sales** ($291K vs $284K). Membership fees have the same issue using `ETS!N20` = 0.0878 (hardcoded).

**Downstream effect:** NOPAT, FCFF, NOA all use the higher IS revenue → DCF intrinsic value is slightly higher than what a strict ETS Base Case would produce.

---

### C3. Bull Case Net Sales 2028 formula points at the membership-fee column

**Sheet:** `Revenue Forecast ETS`, cell **P26**

```
P26 = SUM(I23:I26)   →   10,346.43
```

Column `I` holds the membership-fee upper bound; should be **SUM(H23:H26)** (net-sales upper bound). Looking at the pattern: `N26=SUM(B15,H16:H18)`, `O26=SUM(H19:H22)`, `Q26=SUM(H27:H30)` — only P26 is wrong. The displayed Bull Case Net Sales for 2028 = $10,346M instead of ~$486,444M. Then `P24` (selected scenario row) inherits this: `P24 = CHOOSE($N$21, P26, P27, P28) = 10,346`.

---

### C4. Balance Sheet cash and Cash Flow Statement cash do NOT reconcile

**Sheets:** `Forecast` BS vs `Cash flow forecast`

| Year | BS cash (`Forecast!K31:M31`) | CF cash end-of-year (`CF!G40:I40`) | Gap |
|---|---|---|---|
| FY26E | 18,234 | 16,133 | **+2,101** |
| FY27E | 22,681 | 20,113 | **+2,569** |
| FY28E | 25,407 | 25,070 | +337 |

The BS cash is explicitly a **plug** (`Forecast!K31` formula: `K67 − SUM(K32:K35) − SUM(K38:K40)`; R31 note: "Cash is a plugged amount"). The CF builds cash bottom-up. They diverge by roughly the AFN surplus — the BS retains it, the CF doesn't pay it out. Either the CF should include the special dividends mentioned in AFN, or the BS plug needs to redirect surplus elsewhere.

---

### C5. Long-term debt on the Forecast BS shows beginning balance, not ending balance

**Sheet:** `Forecast`, cells **K51:M51**

```
K51 = K112   →   5,713    (K112 = J114 = FY25 ending debt)
L51 = L112   →   5,638    (L112 = K114 = FY26 ending debt)
M51 = M112   →   3,388    (M112 = L114 = FY27 ending debt)
```

Each projected year's BS LT Debt line is one year behind. Should be `K51 = K114`, `L51 = L114`, `M51 = M114`. Historicals are fine (`F51 = F114` = ending balance). This pushes FY26 BS LT Debt to $5,713 instead of $5,638 — a $75M overstatement of liabilities, growing to $2,250M in FY27. (Doesn't affect the BS plug since the plug absorbs the difference into cash — but it overstates LT debt and understates cash by the same amount.)

---

### C6. Bear Case "CAGR" is shown as a positive +7.91% despite declining revenue

**Sheet:** `Revenue Forecast ETS`, cell **N29** (in the Bear Case net-sales scenario block)

Bear Case Net Sales: 233,354 → 197,418 → 185,780 → 185,885 — this is a declining series. True 3-period CAGR ≈ **−7.4%**; from FY25 baseline of 269,912 it's **−8.9%**. The cell shows **+7.91%** because **all the "CAGR" cells in the ETS sheet are hardcoded literals**, not formulas (see issue M3 below).

---

## 🟡 MODERATE — math/methodology issues

### M1. Monthly returns in beta regression are computed backward in time

**Sheet:** `Beta Calculation`, cells **C3:C62 and E3:E62**

```
C3 = B3/B2 - 1   →   -0.0387   (Jan 30 / Feb 11 - 1)
```

Data is sorted descending by date (newest at top). The return formula divides each row's price by the row *above* it (a more recent date) — i.e., it computes `older / newer - 1`. The correct forward-monthly return for row 3 (Jan 2026) would be `B3/B4 - 1 = 940.25/862.34 - 1 = +9.04%`. The cell shows `-3.87%`.

Because the sign error applies symmetrically to BOTH stock returns (C column) and market returns (E column), the regression **slope (beta = 0.996) is approximately preserved** — but the alpha (−0.0068) is also sign-flipped from its conventional sign, and there's a small magnitude bias because `r_backward ≈ -r_forward / (1 + r_forward)`, not exactly `-r_forward`. The reported beta of 0.996 is close to but not identical to the standard CAPM beta on forward-period returns.

---

### M2. Three independent scenario selectors in ETS sheet currently mix scenarios

**Sheet:** `Revenue Forecast ETS`, cells **N2, N12, N21**

- `N2 = 2`  → Total Revenue row pulls **Base Case**
- `N12 = 1` → Membership Fee row pulls **Bull Case**
- `N21 = 1` → Net Sales row pulls **Bull Case**

The three "selected" output rows are showing **different scenarios simultaneously**. If the intent is "pull the selected scenario across all three rows," there should be one master selector. As-is, Total Revenue (290,232) and Net Sales (335,384) for FY26 imply membership fees of −$45K (impossible) because they come from different scenarios.

---

### M3. CAGR cells in ETS sheet are hardcoded — no formulas

- `ETS!N10` = 0.0793 (Total Revenue CAGR)
- `ETS!N20` = 0.0878 (Membership Fee CAGR)
- `ETS!N29` = 0.0791 (used downstream by Forecast IS)

None of these match a straightforward 4-period CAGR of the corresponding series:

- True Base Case Total Revenue CAGR: (365,768/275,235)^(1/4) − 1 = **7.36%** (not 7.93%)
- True Base Case Net Sales CAGR: (358,201/269,912)^(1/4) − 1 = **7.32%** (not 7.91%)

They look like values typed in at some intermediate point and not refreshed.

---

### M4. Net debt in Exit Multiple method omits short-term investments

**Sheet:** `Valuation DCF`, cell **B79**

```
B79 = C21 - Forecast!J31 = 9952 - 14161 = -4209
```

Conventional Net Debt = Total Debt − Cash − ST Investments. ST Investments (`Forecast!J32` = 1,123) is missing. Correct net cash = 9,952 − 14,161 − 1,123 = **−5,332**. Impact on Method 2 share price: roughly +$2.50/share ($1,123M / 443.87M shares).

---

### M5. WACC weights in Valuation DCF sheet are hardcoded

**Sheet:** `Valuation DCF`, cells **C31, C32**

```
C31 = 0.0217         (debt weight - LITERAL)
C32 = 0.9783         (equity weight - LITERAL)
```

On the standalone WACC sheet, C31 and C32 are formulas (`=C21/(B28+C21)` and `=B28/(B28+C21)`). On Valuation DCF they're typed values. If share price changes (B27), equity value updates but the WACC won't, because the weights it depends on are fixed.

---

### M6. WACC inputs are duplicated in Valuation DCF without linking to the WACC sheet

The Valuation DCF sheet has its own copy of: Beta (B2 — but this one IS linked to `Beta Calculation!O1`), Rf (B3 = 0.0402 literal), Risk premium (B4 = 0.055 literal), and the entire cost-of-debt tranche table (B12:C20 — all literals). Updating the WACC sheet does not propagate.

Minor side effect:
- `WACC!B2` = 0.9958 (rounded) vs `Valuation DCF!B2` = 0.995851 → WACC sheet gives 0.093333, Valuation DCF gives 0.093335 (0.2 bps drift).

---

### M7. AFN row mixes net sales (years 2025–2028) with total revenue (year 2029)

**Sheet:** `AFN`, cells **C3:G3** (the "S0" row)

```
C3 = Forecast!J7                  →   269,912    (net sales)
D3 = Forecast!K7                  →   291,262    (net sales)
E3 = Forecast!L7                  →   314,301    (net sales)
F3 = Forecast!M7                  →   339,162    (net sales)
G3 = 'Revenue Forecast ETS'!Q5    →   365,768    (Total Revenue — Base Case)
```

Units mismatch in the last column. ΔS for the final year (`F6 = G3 − F3 = 26,606`) is comparing net sales to total revenue — overstated by the membership-fee piece. Downstream AFN for FY28 (`F15 = −2,175`) is slightly off as a result.

---

### M8. Industry P/E of 32x is hardcoded with no comparable basis

**Sheet:** `Valuation DCF`, cell **B56** = 32 (literal). No peer table, no derivation. Drives the entire Method 3 ($430.62 target). Recommended to anchor to a named comparable set (Walmart, Target, BJ's Wholesale, etc.) and show how 32x was selected.

---

## 🟢 MINOR — labeling / hygiene / numeric quirks

### m1. Cash on Forecast BS is explicitly a plug

`R31` note: "Cash is a plugged amount". This is a common modeling choice but means cash on the BS is NOT a forecasted variable — it just absorbs whatever's needed to make A = L + E. Flagged because it makes the BS look like a forecast but it's really a residual.

---

### m2. WCM Cash Turnover formula is broken

**Sheet:** `WCM`, cells **C7:H7** — formula returns `#NAME?` (visible in the data-only dump). Likely an array-formula typo; the value isn't usable.

---

### m3. WCM Fixed Asset Turnover for FY24 is hardcoded

`WCM!D8` = 8.96 (literal); all other years use the formula `IS_Sales / Avg(BS_PPE_t, BS_PPE_t-1)`. The value happens to be roughly consistent but breaks the formula pattern.

---

### m4. WCM Fixed Asset Turnover for FY21 = 16.35 is artificially inflated

**Sheet:** `WCM`, cell **G8** uses `BALANCE_SHEET!N27` which is empty (FY20 PPE missing from the 10-K extract). The "average PPE" = (23,492 + 0)/2 = 11,746 instead of a real two-year average. Result: 16.35x vs 8–9x for all other years. This **drags the 5-year average up to 10.54x**.

---

### m5. Inventory turnover labeled "COGS/Inventory" but uses average inventory

`WCM!B6` label says `COGS/Inventory`; formulas use `COGS / ((Inv_t + Inv_t-1)/2)`. Either fix the label or fix the formula — both are defensible, just inconsistent.

---

### m6. Cash flow forecast has multiple unexplained hardcoded values

- `G24` = −816 (FY26 ST borrowings repayment) — no formula, no comment
- `G29` = −1,558.5 (FY26 share buyback) — magic number
- `K26` = 1,962 (residual buyback authorization) — magic number used for I29
- `I19` = `H19 - 375.3` (FY28 ST investments) — where does 375.3 come from?

---

### m7. Sensitivity table base-case cell uses rounded WACC (9.33%) vs actual (9.3335%)

The sensitivity rows are 6.33%, 7.33%, 8.33%, 9.33%, 10.33%. None exactly equals the model's actual WACC of 9.3335%. The "base case" cell at WACC=9.33%, g=3.5% reads **$304.16** while the actual DCF returns **$303.98**. Minor (~$0.18/share) but the dashboard heatmap presents the 9.33% cell as if it equals the DCF output.

---

### m8. Beta on WACC sheet (0.9958) is typed; Valuation DCF sheet links to regression

`WACC!B2` is a hand-entered rounded value; `Valuation DCF!B2 = 'Beta Calculation'!O1` is dynamic. Best practice: link both to the regression cell.

---

### m9. Valuation DCF B28 (IV of equity) formula divides by 1,000 not 1,000,000

```
B28 = B26*B27/1000   →   448,828,027.90
```

The label says "$M" but the value is in thousands, so the displayed figure is 1000× too large. Doesn't propagate to WACC because weights are hardcoded (see M5), but the cell itself is wrong by a factor of 1000. (The WACC sheet's B28 uses `/1000000` correctly.)

---

### m10. "Net debt" label with a negative value

`Valuation DCF!B79` is labeled "Net debt" but shows −4,209 (a net CASH position). The B80 formula `= B78 - B79` correctly handles the sign, but the label is misleading — "Net cash" would be clearer.

---

## Severity legend

- 🔴 **Critical**: Material errors that change valuation outputs or break the model's internal consistency. Should be fixed before sharing with anyone evaluating the analysis.
- 🟡 **Moderate**: Methodological or consistency issues that don't break the model but reflect modeling errors a reviewer would flag. Worth fixing if time permits.
- 🟢 **Minor**: Labeling, hardcoded values where formulas would be cleaner, or cosmetic inconsistencies. Hygiene-level cleanup.

## Status tracking

Add a column or mark items as you address them:

| ID  | Title                                                                  | Status     | Notes |
|-----|------------------------------------------------------------------------|------------|-------|
| C1  | EPS double-discount in target-price calc                               | Open       |       |
| C2  | Forecast IS net sales use hardcoded CAGR, not ETS Base Case            | Open       |       |
| C3  | Bull Case Net Sales 2028 references wrong column                       | Open       |       |
| C4  | BS cash and CF cash don't reconcile                                    | Open       |       |
| C5  | LT Debt on BS shows beginning balance, not ending                      | Open       |       |
| C6  | Bear Case CAGR shown as positive despite declining series              | Open       |       |
| M1  | Monthly returns in beta regression computed backward                   | Open       |       |
| M2  | Three independent scenario selectors mix scenarios                     | Open       |       |
| M3  | CAGR cells in ETS sheet are hardcoded literals                         | Open       |       |
| M4  | Net debt omits short-term investments                                  | Open       |       |
| M5  | WACC weights hardcoded in Valuation DCF                                | Open       |       |
| M6  | WACC inputs duplicated without linking                                 | Open       |       |
| M7  | AFN row mixes net sales with total revenue                             | Open       |       |
| M8  | Industry P/E of 32x hardcoded without comparable basis                 | Open       |       |
| m1  | Cash on Forecast BS is a plug                                          | Open       |       |
| m2  | WCM Cash Turnover formula returns #NAME?                               | Open       |       |
| m3  | WCM Fixed Asset Turnover FY24 hardcoded                                | Open       |       |
| m4  | WCM Fixed Asset Turnover FY21 inflated by missing FY20 PPE             | Open       |       |
| m5  | Inventory turnover label doesn't match its formula                     | Open       |       |
| m6  | Multiple unexplained hardcoded values in Cash Flow forecast            | Open       |       |
| m7  | Sensitivity base-case cell uses rounded WACC vs actual                 | Open       |       |
| m8  | Beta typed on WACC sheet; linked on Valuation DCF — drift              | Open       |       |
| m9  | Valuation DCF B28 unit error (×1,000 instead of ×1,000,000)            | Open       |       |
| m10 | "Net debt" label shows net cash position (sign confusion)              | Open       |       |
