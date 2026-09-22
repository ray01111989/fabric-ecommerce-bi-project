# DAX Measures

All measures below are defined on the `ecommerce_semantic_model` semantic model, built on top of the star schema (fact table `orderitems`, dimensions `orders_clean`, `products_clean`, `sellers`, `customers`, `dim_date`, `reviews_clean`).

| Measure | DAX | What it answers |
|---|---|---|
| **Total Sales** | `SUM(orderitems[price])` | Total revenue across all order line items. |
| **Sales LY** | `CALCULATE([Total Sales], SAMEPERIODLASTYEAR(dim_date[Date]))` | Revenue for the same period one year earlier, used as the year-over-year comparison baseline. |
| **YoY %** | `DIVIDE([Total Sales] - [Sales LY], [Sales LY])` | Percentage change in revenue versus the same period last year. |
| **Sales PM** | `CALCULATE([Total Sales], DATEADD(dim_date[Date], -1, MONTH))` | Revenue for the prior month, used as the month-over-month comparison baseline. |
| **MoM %** | `DIVIDE([Total Sales] - [Sales PM], [Sales PM])` | Percentage change in revenue versus the previous month. |
| **Avg Delivery Days** | `AVERAGE(orders_clean[delivery_duration_days])` | Average number of days between an order being placed and delivered. |
| **On-Time Delivery %** | `AVERAGE(orders_clean[on_time])` | Share of orders delivered on or before their estimated delivery date. |
| **Avg Review Score** | `AVERAGE(reviews_clean[review_score])` | Average customer review score (1-5 scale) across all reviewed orders. |

## Notes

- `YoY %` and `MoM %` are formatted as Percentage, but their underlying value is still a decimal (e.g. `0.15` for 15%). This matters when building conditional formatting rules against them in the Power BI service — rule thresholds must be entered on the decimal scale (`0` / `999`), not the displayed percentage scale (`0` / `100`), or the rule silently never fires.
- `dim_date` is a hand-built date table (via Spark `sequence()` in the transform notebook) and is marked as the model's official Date Table, which is required for `SAMEPERIODLASTYEAR` and `DATEADD` to return correct results.
- The fact table is at item-level grain (`orderitems`), a deliberate choice to preserve per-product price breakdowns rather than rolling up to order-level.
