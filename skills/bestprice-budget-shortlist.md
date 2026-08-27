---
name: bestprice-budget-shortlist
description: Build a short, honest shortlist of Greek-market products under a stated budget, without over-claiming features the API does not verify.
api: BestPrice Shopping MCP
endpoint: https://mcp.bestprice.gr/mcp
transport: streamable-http
auth: none
operations:
  - search_products
generated: '2026-08-27'
method: generated
source: Grounded in the live search_products inputSchema/outputSchema at https://mcp.bestprice.gr/mcp (fetched 2026-08-27) and the limits published on https://www.bestprice.gr/mcp.
---

# Shortlist under a budget

## Steps

1. **Call `search_products`** with `query` plus `price_max` for the budget ceiling (and
   `price_min` if the shopper wants to avoid the bottom of the market). Both bounds are in EUR and
   both apply to the item price *before shipping*.
2. **Set `sort` deliberately.** Default is `relevance`. Use `price_asc` only when the shopper
   explicitly asked for cheapest-first — sorting by price silently reorders the answer away from
   what they asked for.
3. **Keep the list short.** `limit` is 1–8 and defaults to 8. Three to five is usually a better
   answer than eight.
4. **Handle an empty result properly.** When nothing fits, the response may carry
   `suggested_queries[]` — offer one of those as a narrower retry rather than inventing your own
   broader search. Say plainly that nothing matched.
5. **Present each candidate** with `title`, `brand`, `price_from`, `offer_count`, `availability`
   and the `bestprice_url`. State that `price_from` excludes delivery.

## Cautions

- `required_features` (max 10 hints) is **not verified**. The response says so:
  `applied_filters.feature_hints_verified: false`. Present features as "listed as" and point the
  shopper at the product page to confirm.
- `match_confidence` and `match_basis` are published for a reason. A 0.79 match to a different
  model is a wrong answer wearing a right answer's clothes — surface the confidence.
- There is no pagination. `total_matches` counts what was returned, not what exists; you cannot
  reach result nine, and you should not imply the list is exhaustive.
- Out of scope: services, digital products, regulated goods, and any category outside reviewed
  technology / appliances / home and garden.
