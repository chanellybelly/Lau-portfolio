# Tesla, Inc. (TSLA) — Accounting & Performance Ratio Model · Technical Specification

> <span style="color:#024731; font-weight:700;">Post-build specification</span> documenting the Stage 2 Excel model, validating it against the data, and articulating the refinements required for production use. Drives the Stage 4 AI prompt and final analysis.

| Field | Value |
|------|------|
| **Created by** | Chanel Lau |
| **Updated by** | Chanel Lau |
| **Date Created** | 2026-04-24 |
| **Date Updated** | 2026-04-24 |
| **Version** | 1.0 |
| **Role** | Financial Analyst / FP&A Analyst |
| **Audience** | CFO / Director of FP&A |
| **Companion Workbook** | `Lau-Chanel-Stage2-model.xlsx` |

---

## 1. Problem Statement

Tesla, Inc. is a publicly traded electric vehicle and clean energy company operating in the automotive and technology industries. This specification documents the analytical framework used to evaluate Tesla’s fiscal year 2025 financial performance, with fiscal year 2024 serving as the prior-year comparison. The ratio model measures profitability, efficiency, leverage, liquidity, and market-based performance using Tesla’s published financial statements and selected market data. The purpose of the model is to support management decision-making, benchmark operating performance, and provide the foundation for the Stage 4 AI-assisted executive analysis.

---

## 2. Inputs (Known Variables)

All inputs are sourced from Tesla's Form 10-K (SEC EDGAR) unless otherwise noted. Figures are reported in $ millions except share price (USD) and shares outstanding (millions). Market and analyst inputs are the only values an analyst should adjust for scenario analysis.

> <span style="color:#024731;"><b>Naming-convention decoder.</b></span> Balance-sheet inputs generally use the year-suffixed format `BAL_[item]_[yr]`, where applicable. The calculation flow in Section 4 references these balances through two aliases: `startYear_[item]` for FY2024 values and `currentYear_[item]` for FY2025 values. Income-statement and cash-flow items (`INC_*`, `CASH_*`) use single-year naming because the model focuses on FY2025 operating results. This structure improves formula readability and supports reproducibility.

### 2.1 Balance Sheet Items (Current and Prior Year)

| Variable | Description | Named Range | Year(s) | FY2025 | FY2024 |
|----------|-------------|-------------|---------|---------:|----------:|
| Cash & marketable securities | Liquid assets | `BAL_cash_marketable_securities_[yr]` | Both | 36,563 | 29,094 |
| Receivables | Accounts receivable | `BAL_receivables_[yr]` | Both | 4,010 | 3,508 |
| Inventories | Inventory balance | `BAL_inventories_[yr]` | Both | 14,091 | 13,626 |
| Total current assets | Sum of current assets | `BAL_assets_current_[yr]` | Current | 58,369 |  |
| Net tangible fixed assets | PP&E less accumulated depreciation | `BAL_fixed_assets_net_[yr]` | Current | 51,707 |  |
| Total assets | All assets | `BAL_assets_total_[yr]` | Both | 122,070 |  106,618|
| Total current liabilities | Short-term obligations | `BAL_liabilities_current_[yr]` | Current | 28,748 |  |
| Accounts payable | Supplier balances (needed for DPO / cash-conversion-cycle extension — §6.2) | `BAL_accounts_payable_[yr]` | Both |  15,255| 13,720 |
| Long-term debt | Non-current borrowings | `BAL_debt_long_term_[yr]` | Both | 5,157 | 2,526 |
| Total liabilities | All liabilities | `BAL_liabilities_total_[yr]` | Current | 43,009 |  |
| Shareholders' equity | Book value of equity | `BAL_equity_shareholders_[yr]` | Both | 79,061 | 63,362 |

### 2.2 Income Statement Items (Current Year)

| Variable | Description | Named Range | FY2025 |
|----------|-------------|-------------|---------:|
| Net sales | Total revenue | `INC_sales` | 94,827 |
| Cost of goods sold | Direct costs | `INC_cost_goods_sold` | 74,727 |
| SG&A expenses | Operating expenses | `INC_sga` | 4,800 |
| Depreciation | Non-cash expense | `INC_depreciation` | 5,000 |
| EBIT | Operating income | `INC_ebit` | 7,076 |
| Other income | Non-operating income | `INC_other_income` | 1,788 |
| Interest expense | Cost of debt | `INC_interest_expense` | 191 |
| Taxes | Income tax expense | `INC_taxes` | 1,083 |
| Net income | Bottom line | `INC_net` | 7,153 |
| Dividends | Shareholder distributions | `INC_dividends` | 0 |

### 2.3 Cash Flow Statement Items (Current Year)

| Variable | Description | Named Range | FY2025 |
|----------|-------------|-------------|---------:|
| Cash from operations | Operating cash flow | `CASH_operating` | 14,923 |
| Cash from investing | Investing cash flow | `CASH_investments` | (11,382) |

### 2.4 Market / Analyst Inputs (Assumptions)

| Variable | Description | Named Range | Value |
|----------|-------------|-------------|------:|
| Share price | FY-end closing price | `share_price` | 479.86 |
| Shares outstanding (M) | Diluted weighted avg | `shares_outstanding` | 3,520 |
| Cost of capital | Estimated WACC | `cost_capital` | 10.0% |
| Tax rate | Effective or statutory | `tax_rate` | 21.0% |

> <span style="color:#024731;">**Tip:**</span> Keep labels short and standardized — these names become Excel named ranges *and* AI prompt parameters in Stage 4.

---

## 3. Assumptions & Constraints

- All figures are reported in $ millions unless otherwise noted.
- Share price is stated in USD per share; shares outstanding are shown in millions.
- Tax rate is assumed at the U.S. statutory corporate rate of 21% for consistency in after-tax operating profit calculations.
- Cost of capital is estimated at 10.0% based on a simplified WACC assumption suitable for classroom valuation analysis.
- Interest expense is quoted on a simple annual basis; no accrued-interest adjustment is applied.
- Start-of-year balances are taken from Tesla’s prior fiscal year (FY2024) balance sheet.
- Average denominators use the arithmetic mean of beginning and ending balances.
- Depreciation is sourced from Tesla’s cash flow statement and supporting disclosures when available.
- No off-balance-sheet items, contingent liabilities, operating lease adjustments, or purchase commitments are included unless specifically disclosed in core statements.
- Market inputs such as share price and shares outstanding are external assumptions and may differ from intra-year averages.
- The model uses historical data only and does not include forecasted operating results or scenario analysis.

---

## 4. Calculation Flow

Described in named-range pseudocode so the logic is portable to Excel, Python, or an AI prompt. All formulas assume the naming conventions in §2.

### Step 1 — Derived Inputs

1. `market_capitalization` = `share_price` × `shares_outstanding`
2. `after_tax_operating_income` = `INC_ebit` × (1 - `tax_rate`)
3. `daily_sales_average` = `INC_sales` / 365
4. `daily_cogs_average` = `INC_cost_goods_sold` / 365
5. `net_working_capital` = `BAL_assets_current_2025` - `BAL_liabilities_current_2025`
6. `startYear_total_capitalization` = `BAL_debt_long_term_2024` + `BAL_equity_shareholders_2024`
7. `currentYear_total_capitalization` = `BAL_debt_long_term_2025` + `BAL_equity_shareholders_2025`
8. `avg_equity` = AVERAGE(`BAL_equity_shareholders_2024`, `BAL_equity_shareholders_2025`)
9. `avg_total_assets` = AVERAGE(`BAL_assets_total_2024`, `BAL_assets_total_2025`)
10. `avg_total_capitalization` = AVERAGE(`startYear_total_capitalization`, `currentYear_total_capitalization`)


### Step 2 — Performance Ratios

- **MVA** = `market_capitalization` - `BAL_equity_shareholders_2025`
- **Market-to-Book** = `market_capitalization` / `BAL_equity_shareholders_2025`
- **EVA** = `after_tax_operating_income` - (`cost_capital` × `avg_total_capitalization`)

### Step 3 — Profitability Ratios (start-of-year and average denominators)

- **ROA** = `after_tax_operating_income` / `BAL_assets_total_2024`
- **ROC** = `after_tax_operating_income` / `startYear_total_capitalization`
- **ROE** = `INC_net` / `BAL_equity_shareholders_2024`
- **ROA (avg)** = `after_tax_operating_income` / `avg_total_assets`
- **ROC (avg)** = `after_tax_operating_income` / `avg_total_capitalization`
- **ROE (avg)** = `INC_net` / `avg_equity`

### Step 4 — Efficiency Ratios

- **Asset Turnover** = `INC_sales` / `BAL_assets_total_2024`
- **Receivables Turnover** = `INC_sales` / `BAL_receivables_2024`
- **Average Collection Period (days)** = `BAL_receivables_2024` / `daily_sales_average`
- **Inventory Turnover** = `INC_cost_goods_sold` / `BAL_inventories_2024`
- **Days in Inventory** = `BAL_inventories_2024` / `daily_cogs_average`
- **Profit Margin** = `INC_net` / `INC_sales`
- **Operating Profit Margin** = `after_tax_operating_income` / `INC_sales`

### Step 5 — Leverage Ratios

- **Long-term Debt Ratio** = `BAL_debt_long_term_2025` / (`BAL_debt_long_term_2025` + `BAL_equity_shareholders_2025`)
- **Debt-to-Equity Ratio** = `BAL_debt_long_term_2025` / `BAL_equity_shareholders_2025`
- **Total Debt Ratio** = `BAL_liabilities_total_2025` / `BAL_assets_total_2025`
- **Times Interest Earned** = `INC_ebit` / `INC_interest_expense`
- **Cash Coverage Ratio** = (`INC_ebit` + `INC_depreciation`) / `INC_interest_expense`
- **Debt Burden** = `INC_net` / `after_tax_operating_income`
- **Leverage Ratio** = `BAL_assets_total_2025` / `BAL_equity_shareholders_2025`

### Step 6 — Liquidity Ratios

- **NWC to Assets** = `net_working_capital` / `BAL_assets_total_2025`
- **Current Ratio** = `BAL_assets_current_2025` / `BAL_liabilities_current_2025`
- **Quick Ratio** = (`BAL_cash_marketable_securities_2025` + `BAL_receivables_2025`) / `BAL_liabilities_current_2025`
- **Cash Ratio** = `BAL_cash_marketable_securities_2025` / `BAL_liabilities_current_2025`

### Step 7 — Du Pont Decomposition

- **Du Pont ROA** = `Asset Turnover` × `Operating Profit Margin`
- **Du Pont ROE** = `Leverage Ratio` × `Asset Turnover` × `Operating Profit Margin` × `Debt Burden`
- **Reasonableness Check:**  
  Du Pont ROA should closely approximate the directly calculated ROA, subject to rounding differences.  
  Du Pont ROE should approximate direct ROE, although minor differences may occur when using average versus year-end balances.

> <span style="color:#024731;">**Reasonableness checks:**</span> Du Pont ROA should tie to direct ROA within rounding. Du Pont ROE may diverge from direct ROE when leverage uses current-year equity while ROE uses start-of-year equity — flag any divergence greater than ~3 percentage points for review.

---

## 5. Outputs

| Output | Description | Format | Purpose |
|--------|-------------|--------|---------|
| Ratio Summary Table | Consolidated performance, profitability, efficiency, leverage, and liquidity ratios | Excel worksheet | Primary analytical output |
| Performance Metrics Panel | MVA, Market-to-Book, EVA results | Summary block | Measures shareholder value creation |
| Profitability Section | ROA, ROC, ROE using beginning-year and average balances | Table | Evaluates return generation |
| Efficiency Section | Asset turnover, receivables turnover, inventory turnover, collection period | Table | Measures operating efficiency |
| Leverage Section | Debt ratios, interest coverage, leverage multiplier | Table | Assesses financial risk |
| Liquidity Section | Current ratio, quick ratio, cash ratio, NWC to assets | Table | Measures short-term solvency |
| Du Pont Analysis | ROA and ROE decomposed into operational drivers | Sub-table | Identifies return drivers |
| Executive Summary (Stage 4) | Written management interpretation of key findings | Memo / Markdown | Final decision-support deliverable |

### 5.1 Computed Output Values

The computed ratio values below serve as a regression checkpoint for future versions.

| Category | Ratio | Value | Notes |
|----------|-------|------:|-------|
| Performance | MVA | 1,610,126 | $ millions |
| Performance | Market-to-Book | 21.37x | Market cap / equity |
| Performance | EVA | (860) | Approx., depends on WACC |
| Profitability | ROA (SOY) | 6.64% | Using FY2024 assets |
| Profitability | ROC (SOY) | 10.86% | |
| Profitability | ROE (SOY) | 11.29% | |
| Profitability | ROA (avg) | 6.27% | |
| Profitability | ROC (avg) | 10.11% | |
| Profitability | ROE (avg) | 10.04% | |
| Efficiency | Asset Turnover | 0.89x | |
| Efficiency | Receivables Turnover | 27.03x | |
| Efficiency | Avg Collection Period | 13.5 days | |
| Efficiency | Inventory Turnover | 5.48x | |
| Efficiency | Days in Inventory | 66.6 days | |
| Efficiency | Profit Margin | 7.54% | |
| Efficiency | Operating Profit Margin | 5.89% | After-tax EBIT margin |
| Leverage | Long-term Debt Ratio | 6.12% | LT debt / (LT debt + equity) |
| Leverage | Debt-Equity Ratio | 0.07x | LT debt only |
| Leverage | Total Debt Ratio | 35.23% | Liabilities / assets |
| Leverage | Times Interest Earned | 37.05x | |
| Leverage | Cash Coverage Ratio | 63.23x | |
| Leverage | Debt Burden | 1.35x | See model definition |
| Leverage | Leverage Ratio | 1.54x | Assets / equity |
| Liquidity | NWC to Assets | 24.25% | |
| Liquidity | Current Ratio | 2.03x | |
| Liquidity | Quick Ratio | 1.41x | |
| Liquidity | Cash Ratio | 1.27x | |
| Du Pont | Du Pont ROA | 5.24% | |
| Du Pont | Du Pont ROE | 10.92% | |

---

## 6. Model Review — What Worked & What to Improve

Reflect candidly on the Stage 2 model. This section is what makes a *post-build* spec more valuable than a pre-build plan.

### 6.1 What Worked

- The Stage 2 model successfully grouped ratios into performance, profitability, efficiency, leverage, liquidity, and Du Pont categories.
- Tesla 2025 and 2024 financial statement data was organized into one workbook, allowing consistent calculations.
- Average balance formulas improved ROA, ROE, and ROC analysis compared with using only ending balances.
- Market-based metrics such as MVA and Market-to-Book added valuation insight beyond accounting ratios.
- The model produced a clear summary of Tesla’s operating performance, liquidity position, and capital structure.

### 6.2 What to Improve

- Named ranges should be applied consistently across all formulas for better readability and auditability.
- Inputs, assumptions, calculations, and outputs should be separated into dedicated tabs for a cleaner workbook design.
- Key assumptions such as WACC, tax rate, and share price should be placed in a scenario input section.
- Error checks should be added to test balance sheet balancing, negative signs, and unusual ratio outputs.
- Charts and dashboards would improve management presentation quality.
- Peer comparison against companies such as Ford, GM, or Rivian would strengthen interpretation.

### 6.3 Auditability Checklist

- [x] All key ratios are documented with named-range formulas in this specification.
- [x] Inputs, assumptions, calculations, and outputs are logically separated.
- [x] Prior-year and current-year balances are consistently labeled FY2024 and FY2025.
- [x] Average balance calculations are used where appropriate for ROA, ROE, and ROC.
- [x] Major assumptions (tax rate, WACC, share price, shares outstanding) are disclosed.
- [x] Ratio outputs were reviewed for reasonableness and sign consistency.
- [ ] Additional automated error checks could be added in a future version.
- [ ] Peer benchmarking and dashboard visuals remain future enhancements.

---

## 7. Limitations & Next Steps

### Limitations

- The model is based primarily on historical FY2025 and FY2024 financial statement data and does not include forward-looking forecasts.
- Ratio analysis is limited to one company and does not include peer benchmarking against competitors such as Ford, GM, or Rivian.
- Certain market assumptions, including share price, WACC, and shares outstanding, are simplified estimates and may change over time.
- Some ratios rely on summarized financial statement line items rather than deeper segment-level operating data.
- Non-financial factors such as product pipeline, regulatory risk, competition, and management execution are not captured in ratio analysis alone.

### Next Steps

- Expand the model to include three- to five-year trend analysis.
- Add peer comparison metrics to improve interpretation of Tesla’s relative performance.
- Build sensitivity analysis for valuation inputs such as WACC, margins, and revenue growth.
- Add dashboard visualizations for executive presentation use.
- Use this technical specification as the framework for Stage 4 AI-assisted narrative analysis and recommendations.

---

## 8. Writing a Strong Specification

> This technical specification is designed to function as a professional handoff document that can be used by another analyst, manager, or AI assistant to reproduce and extend the Tesla ratio model.

- **Professional communication:** The model logic, assumptions, and outputs are presented in a clear and structured format suitable for management review.
- **Forward-looking design:** The specification is intentionally structured to support the Stage 4 AI prompt and final executive analysis.
- **Internal consistency:** Input labels, formulas, named ranges, and ratio definitions are aligned throughout the document.
- **Reproducibility:** A separate analyst should be able to rebuild the workbook using the documented inputs, assumptions, and calculation flow.
- **Reflective improvement mindset:** The Model Review section identifies both strengths and areas for improvement from the Stage 2 build.
- **Executive relevance:** Outputs focus on profitability, liquidity, leverage, efficiency, and shareholder value metrics that are useful for decision-making.
  
---

## 9. How This Sets Up Stage 4

| Stage 3 Deliverable | Stage 4 Benefit |
|---|---|
| Defined inputs and named ranges | Enables consistent AI analysis using standardized variables |
| Documented calculation flow | Supports accurate explanation of ratio methodology |
| Completed ratio outputs | Provides factual basis for interpretation and recommendations |
| Model review observations | Helps identify improvements and management priorities |
| Limitations and next steps | Guides deeper strategic analysis in the final report |
| Professional specification format | Allows efficient handoff to AI-assisted executive memo development |

---

## Appendix A — Change Log

| Version | Date | Author | Change |
|---------|------|--------|--------|
| 1.0 | 2026-04-24 | Chanel Lau | Stage 3 technical specification for Tesla ratio model |

---
