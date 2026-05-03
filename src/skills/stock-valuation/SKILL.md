---
name: stock-valuation
description: Comprehensive stock valuation (Comps + DCF + SOTP + ROIC + catalysts). Triggers when user asks for fair value, intrinsic value, comprehensive valuation, "is X undervalued", price target, or detailed financial analysis.
---

# Stock Valuation Skill (V2.0 Enhanced)

## Workflow Checklist

```
Stock Valuation Progress:
- [ ] Step 0: Identify industry → pick valuation method
- [ ] Step 1: Comparable companies analysis (Comps)
- [ ] Step 2: DCF valuation (3 scenarios)
- [ ] Step 3: ROIC vs WACC analysis
- [ ] Step 4: SOTP if multi-business
- [ ] Step 5: Catalysts & liquidity check
- [ ] Step 6: Margin of safety & final conclusion
```

## Step 0: Industry → Method

Use `get_financials` with query `"[TICKER] financial metrics snapshot"` to get sector/industry. Then pick method:

| Sector | Primary Method | Key Metrics |
|--------|---------------|-------------|
| Tech/Consumer/Industrial | Comps + DCF | P/E, EV/EBITDA, EV/S |
| Banks/Insurance | P/B + ROE decomposition | P/B, ROE, NIM, CET1 |
| REITs | P/FFO, P/AFFO | FFO, AFFO, Cap Rate |
| Cyclicals (steel/chem/shipping) | Cycle-adj P/E, P/NAV | 5-10yr avg P/E |
| Biotech/Loss-growth | rNPV / Peak Sales | Pipeline NPV |
| Conglomerates | SOTP (required) | Segment-level valuation |

## Step 1: Comparable Companies Analysis

### 1.1 Select 6-8 peers
Same sub-sector, 0.5x-5x size range.

### 1.2 Collect Data
Use `get_financials` with query `"[TICKER] financial metrics snapshot"` for:
- Revenue, Rev Growth, Gross Margin, EBITDA Margin
- EV/Revenue, EV/EBITDA, P/E, P/B

Use `web_search` for supplemental data (historical ranges).

### 1.3 Statistics
Cross-sectional: Max / 75th / Median / 25th / Min
Time-series: Current P/E vs 5-year range (percentile)

### 1.4 Implied Price
Weighted average: EV/EBITDA (40%), P/E (40%), EV/S (20%)

## Step 2: DCF Valuation

### 2.1 Historical Data
Use `get_financials` to extract: Revenue, EBITDA, CapEx, ΔNWC (3-5 years)

### 2.2 Three Scenarios (5-year)

| Parameter | Bear | Base | Bull |
|-----------|------|------|------|
| Revenue Growth | Conservative | Moderate | Optimistic |
| Terminal EBIT Margin | Low | Mid | High |
| Terminal Growth | 1-2% | 2-3% | 3-4% |

**Note:** If Terminal Value > 70% of EV, flag as high sensitivity.

### 2.3 WACC
- Rf: 10Y UST (use `web_search` for current yield)
- ERP: 5.0% (Damodaran NYU, but use `web_search` to confirm latest)
- Beta: `get_financials` with query `"[TICKER] financial metrics snapshot"`
- Ke = Rf + β × ERP
- WACC = E/V × Ke + D/V × Kd

### 2.4 Probability-Weighted Value
```
Bear 20% | Base 60% | Bull 20%
Expected = 0.2×Bear + 0.6×Base + 0.2×Bull
```

### 2.5 Sensitivity Matrices
A: WACC × Terminal Growth
B: Revenue Growth × Terminal Margin

## Step 3: ROIC vs WACC

```
ROIC = EBIT × (1-tax) / (Total Assets - Cash - Non-interest liabilities)
```

| ROIC vs WACC | Meaning |
|---|---|
| > WACC | Creating value ✅ |
| = WACC | Breaking even ⚠️ |
| < WACC | Destroying value ❌ |

**If ROIC < WACC, growth actually destroys value — adjust Base case downward.**

## Step 4: SOTP (for multi-business companies)

Value each segment separately using appropriate multiples. Apply 10-25% conglomerate discount if applicable.

## Step 5: Catalysts

| Type | Examples |
|------|----------|
| Earnings | Next quarter report, guidance |
| Product | Launch, FDA milestone, new contract |
| Capital allocation | Buyback, dividend hike, M&A |
| Industry | Regulation, tariffs, macro shift |

**No catalyst + deep discount = Value Trap warning.**

## Step 6: Margin of Safety

| Quality | Safety Margin |
|---------|--------------|
| Strong (ROIC>>WACC) | 10-15% |
| Average (ROIC≈WACC) | 20-30% |
| Struggling (ROIC<WACC) | 35-50% |

Entry price = Expected Value × (1 - safety margin)

## Conclusion Matrix

| Signal | Conditions |
|--------|------------|
| ✅ BUY — Strong | Expected > price + safety margin, catalysts exist, ROIC>WACC |
| ✅ BUY — Cautious | Expected > price + safety margin, but no catalysts or ROIC<WACC |
| ⚖️ HOLD | Marginal odds |
| ⚠️ Value Trap | Deep discount, no catalysts, low liquidity, bad ROIC |
| ❌ SELL/Short | Expected value < current price |
