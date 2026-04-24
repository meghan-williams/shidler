# MSFT Ratio Model — Technical Specification
## BUS-314 · International Corporate Finance · Stage 3

**Created by:** Meghan Williams
**Date Created:** 2026
**Version:** 1.0
**LLM Used:** Claude Sonnet (Anthropic)

**Role:** Financial Analyst
**Audience:** CFO / Director of FP&A
**Data Source:** Microsoft 10-K FY2024, SEC EDGAR
**Units:** All figures in $M (USD)

---

## 1. Problem Statement

Microsoft Corporation (MSFT) is a publicly traded Technology company operating across Cloud Computing, Software, and AI. This specification documents the analytical framework for computing 25+ accounting and performance ratios from the company's FY2024 financial statements, enabling management to assess financial health, operational efficiency, capital structure, and value creation. Results are intended for a CFO briefing and internal performance review against prior-year benchmarks.

The model ingests two fiscal years of balance sheet data (FY2023 as the start-of-year anchor, FY2024 as the current year), a single-year income statement, and an indirect-method cash flow statement — all sourced from Microsoft's FY2024 10-K filed with the SEC. Four analyst-supplied assumptions (share price, shares outstanding, cost of capital, and effective tax rate) complete the input set. The output is a fully structured ratio table organized by analytical category, with named-range documentation to support auditability and future AI-assisted refinement.

---

## 2. Inputs (Known Variables)

All inputs are organized by source. Balance sheet items are pulled for both fiscal years; income statement items for FY2024 only. Color coding follows the workbook's convention:

- **Blue** — Analyst assumptions (hard-coded inputs)
- **Yellow** — Financial statement data
- **Gold** — Computed ratio outputs

### Balance Sheet Items

| Variable | Named Range | FY2024 ($M) | FY2023 ($M) | Source |
|---|---|---|---|---|
| **Current Assets** |||||
| Cash & marketable securities | `BAL_cash_marketable_securities_[year]` | 75,543 | 111,262 | Balance Sheet |
| Receivables | `BAL_receivables_[year]` | 56,924 | 48,688 | Balance Sheet |
| Inventories | `BAL_inventories_[year]` | 1,246 | 2,500 | Balance Sheet |
| Total current assets | `BAL_assets_current_[year]` | 159,734 | 184,257 | Balance Sheet |
| **Fixed & Other Assets** |||||
| Net tangible fixed assets (PP&E net) | `BAL_fixed_assets_net_[year]` | 135,591 | 95,641 | Balance Sheet |
| Total assets | `BAL_assets_total_[year]` | 512,163 | 411,976 | Balance Sheet |
| **Liabilities** |||||
| Total current liabilities | `BAL_liabilities_current_[year]` | 125,286 | 104,149 | Balance Sheet |
| Long-term debt | `BAL_debt_long_term_[year]` | 42,688 | 41,990 | Balance Sheet |
| Total liabilities | `BAL_liabilities_total_[year]` | 243,686 | 205,753 | Balance Sheet |
| **Equity** |||||
| Shareholders' equity | `BAL_equity_shareholders_[year]` | 268,477 | 206,223 | Balance Sheet |

### Income Statement Items (FY2024)

| Variable | Named Range | FY2024 ($M) | % of Sales |
|---|---|---|---|
| Net sales | `INC_sales` | 245,122 | 100.0% |
| Cost of goods sold | `INC_cost_goods_sold` | 74,114 | 30.2% |
| Selling, general & administrative | `INC_sga` | 171,008 | 69.8% |
| Depreciation & amortization | `INC_depreciation` | 22,287 | 9.1% |
| EBIT | `INC_ebit` | 109,433 | 44.6% |
| Other income (expense), net | `INC_other_income` | −1,646 | −0.7% |
| Interest expense | `INC_interest_expense` | 0 | 0.0% |
| Taxes | `INC_taxes` | 19,651 | 8.0% |
| Net income | `INC_net` | 88,136 | 36.0% |
| Dividends | `INC_dividends` | 22,296 | 9.1% |

### Market / Analyst Inputs

| Variable | Named Range | Value | Basis |
|---|---|---|---|
| Share price | `share_price` | $446.34 | Closing price Jun 28, 2024 (fiscal year-end) |
| Shares outstanding | `shares_outstanding` | 7,434 M | As of Jun 30, 2024 (10-K cover page) |
| Cost of capital | `cost_capital` | 9.0% | Estimated WACC; refine using CAPM + class methodology |
| Tax rate | `tax_rate` | 18.2% | Effective rate: $19,651M ÷ $107,787M EBT |

---

## 3. Assumptions & Constraints

- All figures reported in millions of USD unless otherwise noted.
- The effective tax rate of 18.2% is used (taxes $19,651M ÷ EBT $107,787M); the statutory 21% may be substituted for scenario analysis.
- Cost of capital is estimated at 9.0% (WACC); students may refine using CAPM and class methodology.
- Start-of-year values use the FY2023 (prior year) balance sheet. This convention prevents circularity and reflects the capital base that was deployed to generate FY2024 income.
- Depreciation is taken from the Income Statement (D&A line: $22,287M), consistent with the cash flow add-back.
- Microsoft reported zero interest expense on the FY2024 income statement; Times Interest Earned and Cash Coverage ratios are therefore undefined (N/A) and excluded from the Du Pont decomposition.
- After-tax operating income is approximated as Net Income + (1 − `tax_rate`) × Interest Expense. Because interest expense = $0, this reduces to Net Income ($88,136M) — meaning the debt burden ratio equals exactly 1.0.
- No off-balance-sheet items, contingent liabilities, or operating lease adjustments are included.
- Market capitalization uses fiscal year-end share price, not a trailing average.
- Average denominators for ROA [AVG], ROC [AVG], and ROE [AVG] use a simple two-point average of start-of-year and end-of-year values.
- Interest rates are quoted on a simple annual basis.

---

## 4. Calculation Flow

The Ratios sheet follows a strict top-to-bottom dependency chain. Step 1 derived inputs must be computed before any ratio formula references them downstream.

### Step 1 — Derived Inputs

1. `market_capitalization` = `share_price` × `shares_outstanding` → **$3,318,091.56M**
2. `startYear_equity` = `BAL_equity_shareholders_2023` → **$206,223M**
3. `startYear_inventory` = `BAL_inventories_2023` → **$2,500M**
4. `startYear_receivables` = `BAL_receivables_2023` → **$48,688M**
5. `startYear_total_assets` = `BAL_assets_total_2023` → **$411,976M**
6. `startYear_total_capitalization` = `BAL_debt_long_term_2023` + `BAL_equity_shareholders_2023` → **$248,213M**
7. `currentYear_after_tax_operating_income` = `INC_net` + (1 − `tax_rate`) × `INC_interest_expense` → **$88,136M**
8. `currentYear_daily_sales_average` = `INC_sales` / 365 → **$671.57M/day**
9. `currentYear_cost_goods_sold_daily` = `INC_cost_goods_sold` / 365 → **$203.05M/day**
10. `currentYear_working_capital_net` = `BAL_assets_current_2024` − `BAL_liabilities_current_2024` → **$34,448M**
11. `currentYear_total_capitalization` = `BAL_debt_long_term_2024` + `BAL_equity_shareholders_2024` → **$311,165M**
12. `avg_equity` = AVERAGE(`startYear_equity`, `BAL_equity_shareholders_2024`) → **$237,350M**
13. `avg_total_assets` = AVERAGE(`startYear_total_assets`, `BAL_assets_total_2024`) → **$462,069.5M**
14. `avg_total_capitalization` = AVERAGE(`startYear_total_capitalization`, `currentYear_total_capitalization`) → **$279,689M**

### Step 2 — Performance Ratios

| Ratio | Formula | Output |
|---|---|---|
| Market Value Added (MVA) | `market_capitalization` − `currentYear_equity` | $3,049,614.56M |
| Market-to-Book Ratio | `market_capitalization` / `currentYear_equity` | 12.36× |
| Economic Value Added (EVA) | `currentYear_after_tax_operating_income` − (`cost_capital` × `startYear_total_capitalization`) | $65,796.83M |

### Step 3 — Profitability Ratios

| Ratio | Formula | Output |
|---|---|---|
| ROA (start-of-year) | `currentYear_after_tax_operating_income` / `startYear_total_assets` | 21.4% |
| ROC (start-of-year) | `currentYear_after_tax_operating_income` / `startYear_total_capitalization` | 35.5% |
| ROE (start-of-year) | `INC_net` / `startYear_equity` | 42.7% |
| ROA [AVG] | `currentYear_after_tax_operating_income` / `avg_total_assets` | 19.1% |
| ROC [AVG] | `currentYear_after_tax_operating_income` / `avg_total_capitalization` | 31.5% |
| ROE [AVG] | `INC_net` / `avg_equity` | 37.1% |

### Step 4 — Efficiency Ratios

| Ratio | Formula | Output |
|---|---|---|
| Asset Turnover | `INC_sales` / `startYear_total_assets` | 0.595× *(→ `RATIO_asset_turnover`)* |
| Receivables Turnover | `INC_sales` / `startYear_receivables` | 5.03× |
| Avg. Collection Period (days) | `startYear_receivables` / `currentYear_daily_sales_average` | 72.5 days |
| Inventory Turnover | `INC_cost_goods_sold` / `startYear_inventory` | 29.6× |
| Days in Inventory | `startYear_inventory` / `currentYear_cost_goods_sold_daily` | 12.3 days |
| Profit Margin | `INC_net` / `INC_sales` | 36.0% |
| Operating Profit Margin | `currentYear_after_tax_operating_income` / `INC_sales` | 36.0% *(→ `RATIO_operating_profit_margin`)* |

### Step 5 — Leverage Ratios

| Ratio | Formula | Output |
|---|---|---|
| Long-term Debt Ratio | `currentYear_debt_long_term` / (`currentYear_debt_long_term` + `currentYear_equity`) | 13.7% |
| Long-term Debt-Equity Ratio | `currentYear_debt_long_term` / `currentYear_equity` | 0.159× |
| Total Debt Ratio | `currentYear_liabilities_total` / `currentYear_assets_total` | 47.6% |
| Times Interest Earned | `INC_ebit` / `INC_interest_expense` | N/A (÷0) |
| Cash Coverage Ratio | (`INC_ebit` + `INC_depreciation`) / `INC_interest_expense` | N/A (÷0) |
| Debt Burden | `INC_net` / `currentYear_after_tax_operating_income` | 1.000 *(→ `RATIO_debt_burden`)* |
| Leverage Ratio | `currentYear_assets_total` / `currentYear_equity` | 1.908× *(→ `RATIO_leverage`)* |

### Step 6 — Liquidity Ratios

| Ratio | Formula | Output |
|---|---|---|
| Net Working Capital to Assets | `currentYear_working_capital_net` / `currentYear_assets_total` | 6.7% |
| Current Ratio | `currentYear_assets_current` / `currentYear_liabilities_current` | 1.275× |
| Quick Ratio | (`currentYear_cash_marketable_securities` + `BAL_receivables_2024`) / `currentYear_liabilities_current` | 1.057× |
| Cash Ratio | `currentYear_cash_marketable_securities` / `currentYear_liabilities_current` | 0.603× |

### Step 7 — Du Pont Decomposition

| Metric | Formula | Output |
|---|---|---|
| Du Pont ROA | `RATIO_asset_turnover` × `RATIO_operating_profit_margin` | 21.4% |
| Du Pont ROE | `RATIO_leverage` × `RATIO_asset_turnover` × `RATIO_operating_profit_margin` × `RATIO_debt_burden` | 40.8% |
| &nbsp;&nbsp;&nbsp;└ Leverage Ratio | → from Step 5 | 1.908 |
| &nbsp;&nbsp;&nbsp;└ Asset Turnover | → from Step 4 | 0.595 |
| &nbsp;&nbsp;&nbsp;└ Operating Profit Margin | → from Step 4 | 0.360 |
| &nbsp;&nbsp;&nbsp;└ Debt Burden | → from Step 5 | 1.000 |

---

## 5. Outputs

| Output | Description | Purpose |
|---|---|---|
| Ratio Summary Table | 25+ ratios organized across Performance, Profitability, Efficiency, Leverage, and Liquidity categories | Core analytical deliverable |
| Du Pont Decomposition | Two-tier breakdown isolating the drivers of ROA and ROE; cross-references named ranges from Efficiency and Leverage sections | Identifies return drivers |
| Formula Documentation | Named-range pseudocode in Column C of the Ratios sheet | Full auditability and AI-prompt reproducibility |
| Common-Size Income Statement | % of Sales column alongside each income line item | Instant margin visibility |
| Balance Check | TA − TL&E formula confirms the balance sheet ties to zero | Validates data entry |
| Executive Memo Input | Ratio outputs feed directly into Stage 4 interpretation and strategic recommendation memo | Senior management briefing |

---

## 6. Model Review — What Worked & What to Improve

### What Worked Well

- Named-range architecture made all ratio formulas self-documenting and easy to audit. Cross-sheet references resolved cleanly.
- The side-by-side two-year balance sheet layout surfaced year-over-year changes immediately (e.g., cash declined $35.7B while PP&E surged $48.1B — Activision capex + acquisition).
- Du Pont decomposition values matched direct ROA and ROE calculations within rounding, confirming internal consistency.
- Balance check formula (TA − TL&E = 0) provided instant validation after data entry.
- The three-tier color key (blue / yellow / gold) made input vs. output cells visually unambiguous.

### What to Improve

- Times Interest Earned and Cash Coverage return N/A due to zero reported interest expense. A conditional formula (`IF(INC_interest_expense=0,"N/A",...)`) should be added to suppress divide-by-zero errors explicitly.
- Profit Margin and Operating Profit Margin produce identical outputs (36.0%) because MSFT's after-tax operating income equals net income when interest = $0. Adding a note cell would prevent misreading this as a model error.
- The SG&A line aggregates all operating expenses below COGS, limiting granularity. A supplemental breakdown (R&D vs. S&M vs. G&A) from the 10-K footnotes would sharpen profitability analysis.
- No industry peer comparison is included. Benchmarking against Alphabet and Apple would dramatically increase the analytical value for a CFO audience.
- Multi-year trend data (FY2022–FY2024) would reveal whether MSFT's margin expansion and ROE improvement are durable trends or single-year anomalies.

---

## 7. Limitations & Next Steps

This specification does not incorporate industry peer benchmarks, multi-year trend analysis, operating lease capitalization adjustments (ASC 842), or off-balance-sheet items. The cost of capital is an estimate; a formal CAPM derivation using current beta and risk-free rate would improve precision. Times Interest Earned and Cash Coverage ratios are analytically unavailable given MSFT's zero reported interest expense line.

The next phase (Stage 4) will use this specification as a structured AI prompt to interpret ratio results, identify key performance drivers, and draft an executive memo with strategic recommendations for senior management. The named-range framework established here allows the Stage 4 prompt to reference variables by name rather than value, ensuring the AI-generated analysis is tied directly to the model structure.

### How This Spec Feeds Stage 4

| What You Built in Stage 3 | What It Enables in Stage 4 |
|---|---|
| Named ranges with precise definitions | AI uses standardized variable names — no improvisation or hallucinated values |
| Step-by-step calculation flow | AI generates correct, auditable ratio formulas in the right order |
| Model review and improvement notes | AI builds an improved version (peer benchmarks, N/A guards) rather than replicating flaws |
| Explicit output requirements | AI produces exactly the tables, analysis sections, and summary format you need |
| Assumptions & constraints section | AI applies consistent conventions (start-of-year denominators, effective tax rate) throughout the memo |

---

*BUS-314 · International Corporate Finance · University of Hawaiʻi at Mānoa*
*Meghan Williams · Stage 3 Technical Specification · MSFT FY2024*
