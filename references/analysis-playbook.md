# Options Volatility Analysis Playbook

Read this file when producing a full options-volatility report, calculating IV percentiles, analyzing term structure or skew, or diagnosing empty/ambiguous option data.

## Data Planning

Use the smallest method set that answers the question:

| Question | Data plan |
|---|---|
| What options exist for this underlying? | `get_option_underlying_detail` then `get_option_detail` |
| What does the latest chain look like? | `get_option_detail` plus `get_option_daily` on the target trade date |
| Is IV rich or cheap? | `get_option_implied_volatility` plus `get_option_underlying_volatility` with matched dates |
| What is the term structure? | Cross-section of IV by expiry, filtered to comparable moneyness |
| What is the skew/smile? | IV by strike and option type within one expiry, with liquidity filters |
| Need underlying context? | `get_index_daily`, `get_stock_daily`, or `get_future_daily` according to the underlying type |

Always verify exact method signatures and fields through `pandadata-api` before using the map.

## Default Windows

- Snapshot date: latest available trading day unless the user provides a date.
- Percentile window: 252 trading days by default; use 60 or 120 trading days for short-history contracts and label the shorter sample.
- HV windows: 20, 60, and 120 trading days when available. Do not compare IV to an unlabeled HV window.
- Liquidity lookback: latest trading day for chain tables; recent 5 to 20 trading days for volume/open-interest sanity checks when stale quotes are suspected.

## Field And Unit Checks

Before analysis, inspect a small result and identify:

- date field and latest available date;
- contract code, underlying code, option type, strike, expiry, and listing status fields;
- IV/HV units: decimal such as `0.24` versus percent such as `24`;
- price units and whether values are adjusted;
- volume/open-interest field names and whether zero rows mean no listing, no trading, or missing data.

Normalize volatility units before calculations. If one source returns `24` and another returns `0.24`, convert both to percent for reporting and state the conversion.

## Calculations

- **Days to expiry**: `expiry_date - trade_date` in calendar days. Flag `< 7` days as near-expiry noise.
- **IV-HV premium**: `IV - HV` in volatility points after unit normalization.
- **IV/HV ratio**: `IV / HV` only when HV is positive and measured over a stated window.
- **IV percentile**: percentile rank of current IV against the selected historical IV sample after removing nulls, stale rows, and obvious no-trade observations.
- **Term-structure slope**: back-month ATM/comparable-moneyness IV minus front-month ATM/comparable-moneyness IV.
- **Skew proxy**: put-side IV minus call-side IV at comparable distance from ATM, or low-strike IV minus high-strike IV when option type pairing is unavailable.

Show formulas briefly in formal reports when calculations drive a conclusion.

## Liquidity And Quality Filters

- Prefer contracts with nonzero volume or open interest when selecting representative strikes.
- Treat deep OTM wings, zero-volume rows, stale last prices, and near-expiry contracts as lower-confidence evidence.
- Do not mix expiries when drawing a smile unless the report explicitly labels it as a cross-expiry comparison.
- If contract metadata and quote data disagree, trust metadata for contract identity and report the mismatch.
- Empty results are not automatically API failures. Check date availability, symbol format, listing status, market holiday, and credentials/runtime setup.

## Report Structure

Use this outline for full reports:

1. **结论摘要**: 3 to 5 bullets with IV level, premium, term structure, skew, and confidence.
2. **标的与合约范围**: underlying/options variety, trade date, expiries, contract count, and methods used.
3. **期权链快照**: table by expiry and representative strikes; include liquidity columns where available.
4. **IV vs HV**: current IV, HV windows, premium, percentile, and unit notes.
5. **期限结构**: front/back expiry comparison and near-expiry caveats.
6. **偏度/微笑**: put/call or strike-side skew with liquidity confidence.
7. **数据说明**: methods, parameters, missing sections, stale data, and calculation formulas.
8. **研究结论**: volatility environment only, with the standard investment disclaimer from `SKILL.md`.

## Tone And Risk Controls

- Use research language, not instructions to buy, sell, short vol, or write options.
- Prefer "IV 偏高/偏低", "卖方环境可能更有利", or "需要结合流动性和风险预算验证" instead of direct trade commands.
- When the user asks for strategy implementation, finish the diagnosis first and then hand off to strategy-specific skills.
