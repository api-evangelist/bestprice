---
name: bestprice-price-timing
description: Decide whether a product's price today is genuinely low, ordinary or high against its own 30/90/180-day history on BestPrice.gr.
api: BestPrice Shopping MCP
endpoint: https://mcp.bestprice.gr/mcp
transport: streamable-http
auth: none
operations:
  - search_products
  - get_price_history
generated: '2026-08-27'
method: generated
source: Grounded in the live tools/list schemas at https://mcp.bestprice.gr/mcp (fetched 2026-08-27, saved to ../mcp/bestprice-tools.json) and the tool contracts on https://www.bestprice.gr/mcp.
---

# Is this actually a good price?

"€680" means nothing on its own. `get_price_history` exists to give it a denominator.

## Steps

1. **Resolve the product** with `search_products` and take a returned `product_id`
   (`bp_<id>`). Price history is per grouped product; there is no history for a search phrase.
2. **Call `get_price_history`** with that `product_id`. `period_days` accepts 30, 90 or 180 and
   defaults to 180. Ask for the longest window the question deserves — a seasonal item needs 180.
3. **Check coverage before you quote a statistic.** Read `data_gaps.missing_days`,
   `longest_gap_days`, `stale_days` and each window's `coverage_pct` / `observations`. A median
   drawn from a sparse window is not a median worth acting on, and this API is unusual in telling
   you so.
4. **Compare current to typical.** `current_price` against each window's `minimum_price` and
   `median_price`, plus `price_delta_vs_180d_median_pct`. The server also classifies it for you in
   `deal_classification`.
5. **Say which method produced the number.** `methodology_id` (e.g. `bestprice_daily_min_v1`)
   states that the series is a daily *minimum* across merchants, not an average of listings.
6. **Report, don't forecast.** The contract makes no future-price prediction and neither should
   you. "Today is 12.4% above the six-month median of €605" is supportable. "It will drop next
   month" is not.

## Cautions

- `current_price_source` tells you where today's number came from (e.g. `live_offer_minimum`), and
  `source_last_observed_at` tells you when. Quote the freshness alongside the price.
- The daily minimum ignores delivery entirely. If the shopper is deciding *where* to buy rather
  than *when*, use `compare_offers` with a postal code — see `bestprice-best-total-price`.
- `series[]` can be long (the provider's own example runs to 64 points). Summarise; do not dump it.
