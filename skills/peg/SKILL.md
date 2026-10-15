---
name: "peg"
description: "Use when Mike types /peg followed by a ticker or company, or asks for a stock's PEG ratio with rationale plus a detailed profile of what the company does and what it has."
---

# /peg TICKER-or-company

One-company deep dive: PEG 1Y/2Y/3Y with rationale, a detailed company profile (what it does + what it has), scorecard, bull/bear, and how it fits Mike's portfolio. Mike is a non-US investor on IBKR (W-8BEN, 30% dividend withholding, avoids dividend-heavy names), runs a wheel strategy (covered calls 3-5% OTM avoiding earnings; cash-secured puts only on stocks he'd own), and wants blunt, direct answers with honest pushback.

## 0. Resolve the ticker
- If given a company name or a wrong/old ticker (e.g. "GY" for Goodyear = GT), resolve it with IBKR `search_contracts` (exact symbol, US primary listing) and say the correct ticker in one line.
- If no argument was given, ask which company.

## 1. Gather data (run independent fetches in parallel)
1. **Live price**: IBKR `get_price_snapshot` (fields: last, change, implied_vol_underlying). Fall back to StockAnalysis price.
2. **Valuation stats**: WebFetch `https://stockanalysis.com/stocks/TICKER/statistics/` -> price, market cap, EV, P/E, forward P/E, PEG (site), EV/EBITDA, P/S, P/B, cash, total debt, net cash/debt, FCF TTM, operating cash flow, revenue TTM, net income TTM, gross/operating/profit margins, ROE, ROIC, dividend yield, shares change YoY (dilution/buybacks), short interest %, beta, Altman Z, analyst rating + target, next earnings date.
3. **Forecasts**: WebFetch `https://stockanalysis.com/stocks/TICKER/forecast/` -> EPS last actual FY, this FY, next FY (with growth %), revenue estimates, estimate high/low spread, price target low/avg/high, rating breakdown and how it changed (revision trend).
4. **Company profile**: WebFetch `https://stockanalysis.com/stocks/TICKER/company/` -> description, sector/industry, HQ, founded, employees, CEO, key executives. Also `/financials/` (or `?p=quarterly`) for revenue/EPS trend if needed.
5. **Segments & recent results**: WebSearch "COMPANY latest quarter results segments revenue guidance YEAR" and fetch 1-2 good sources (company press release / SEC 8-K/6-K/10-Q, Investing.com, Reuters, StockTitan). Get: segment revenue mix and growth, geographic mix, latest quarter beat/miss, guidance, key management commentary, big one-offs (impairments, tax valuation allowances, asset-sale gains).
6. **News & catalysts**: WebSearch "TICKER stock news MONTH YEAR" -> why it moved recently, upgrades/downgrades, target changes, M&A, regulation, lawsuits.
7. **Portfolio fit**: IBKR `get_account_positions` + `get_account_summary` -> does Mike own it (shares, cost basis, % of net liq), cash available, overlap with existing holdings (e.g. semis concentration: TSM/NVDA/AVGO; MSFT is employer exposure).
8. (Optional, if relevant) Options: `get_option_parameters` + `get_option_data` + snapshots for 1-2 put/call strikes so the wheel angle uses real bid/ask, not guesses.

Yahoo analysis pages are unreliable; Zacks/GuruFocus are often paywalled. Prefer StockAnalysis + primary filings. Never invent numbers: if a figure is unavailable, say "n/a" and why.

## 2. Compute PEG (numerator = forward P/E for every horizon)
- **PEG 1Y** = Fwd P/E / next-FY EPS growth %
- **PEG 2Y** = Fwd P/E / annualized growth from last reported FY EPS to next-FY estimate: ((EPS_nextFY / EPS_lastFY)^(1/2) - 1)
- **PEG 3Y** = Fwd P/E / StockAnalysis 3Y EPS growth forecast %
- Run the arithmetic in code (python) rather than in your head.
- Negative/near-zero EPS or negative growth -> "n/m" and explain (losses, one-off gains, depressed base).
- No free consensus for 5Y/10Y: say so, don't invent.
- For ADRs, check the ADR ratio/currency (e.g. TSM: 1 ADR = 5 shares, EPS in TWD) before mixing figures.

**Verdict** (weight PEG 2Y/3Y): under 1 Cheap | 1-2 Fair | over 2 Expensive | no earnings = Speculative.

**PEG rationale (always write this, 3-5 bullets)**: why the PEG is what it is and whether to trust it:
- Is growth real (revenue-driven, margin expansion) or optical (recovery from a depressed base, buybacks, one-offs)?
- Cycle-peak risk (memory, storage, commodities, equipment): a tiny PEG can mean the market is pricing a peak.
- Quality of the estimate: analyst count, high/low spread, revisions up or down in the last 1-3 months.
- Balance-sheet risk the PEG ignores (net debt, interest burden, dilution).
- Jurisdiction/structure risk (China VIE, ADRs, export controls).
- Compare to where the PEG sat last time it was checked (if known from the conversation) and why it moved (price vs estimates).

## 3. Output format (mobile-friendly markdown, lead with the answer)

**Line 1 - Verdict**: "TICKER: Cheap/Fair/Expensive/Speculative - one-sentence reason."

**A. Valuation table**
| Ticker | Price | P/E | Fwd P/E | Net cash/debt | PEG 1Y | PEG 2Y | PEG 3Y | Verdict |

**B. PEG rationale** - 3-5 bullets (see above).

**C. What the company does** (detailed, plain English):
- One-paragraph business summary: what it sells, to whom, how it makes money.
- Segment table: Segment | % of revenue | growth | margin/notes.
- Geographic mix, key customers/concentration, key products/brands.
- Competitive position / moat (or lack of one) and main competitors.
- Management: CEO (+ tenure), notable recent leadership changes.

**D. What the company has** (balance sheet & resources) as a table: Item | Value | Comment. Rows: Cash, Total debt, Net cash/debt, FCF TTM, Operating cash flow, Margins (gross/op/net), ROE/ROIC, Buybacks/dilution (shares YoY), Dividend yield (+ W-8BEN withholding note if above 0), Key assets (fabs, IP, patents, users, brands, backlog), Altman Z.

**E. Scorecard** as a table: Check | Reading. Rows: Beat record (last 2-4 qtrs), Estimate revisions, Estimate spread, FCF vs net income, Balance sheet, Short interest. Mark each good/neutral/bad.

**F. Bull vs bear** - 3 bullets each, concrete and number-backed.

**G. Catalysts** - next earnings date, product launches, regulatory events, debt maturities.

**H. Fit for Mike** (blunt):
- Owned? shares, cost basis, gain/loss, % of account.
- Does it pass his rules (positive earnings, PEG under ~1.2, low dividend, strong balance sheet, liquid options)?
- Concentration/overlap warnings.
- Wheel angle: covered calls (needs 100 shares; strikes above cost basis, expiries before earnings) or cash-secured puts (cash needed = strike x 100 vs his cash). Only suggest puts if he'd be happy to own it.
- Clear action: buy / wait for a level / sell puts at X / avoid - and why.

End with a "Not a financial advisor" one-liner and a **Sources** list of markdown links to every page used.

## Style
- Direct, blunt; push back when the stock fails his own criteria or a move contradicts earlier warnings.
- Detailed but scannable: tables + short bullets; roughly 3-4 phone screens max.
- Date-stamp prices (intraday vs close). Flag when sources disagree.
- If he asks to act, create IBKR order instructions (`create_order_instruction`) and remind him they must be submitted in IBKR.