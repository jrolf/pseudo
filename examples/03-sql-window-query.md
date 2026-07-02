# Example 3: A SQL Window-Function Query

Pseudo is not limited to imperative code. Declarative languages like SQL
describe *what* they want, and the engine decides *how* - which means the
logic a reader needs is often even harder to see. Translating SQL to Pseudo
means narrating the transformation the query performs, stage by stage.

## The original

```sql
WITH ranked AS (
  SELECT
    customer_id,
    region,
    SUM(amount) AS total_spend,
    RANK() OVER (
      PARTITION BY region
      ORDER BY SUM(amount) DESC
    ) AS spend_rank
  FROM orders
  WHERE order_date >= DATE '2026-01-01'
  GROUP BY customer_id, region
)
SELECT customer_id, region, total_spend
FROM ranked
WHERE spend_rank <= 3
ORDER BY region, total_spend DESC;
```

## The Pseudo translation

```python
# TOP THREE CUSTOMERS BY SPEND, WITHIN EACH REGION

Start from all orders placed this year.

Group the orders by customer within each region:
    For each customer-and-region pair:
        Add up everything that customer spent in that region.

Rank the customers inside each region separately:
    # Ranking restarts per region, so a modest spender in a small region
    # can still rank first there. Ties share a rank.
    Order each region's customers from highest total spend to lowest.
    Assign each customer their rank within their own region.

Keep only customers ranked in the top three of their region.

Present the results:
    Sort by region, and within each region from highest spend to lowest.
    Report each customer, their region, and their total spend.
```

## What the translation reveals

`PARTITION BY` is the piece that trips up every SQL newcomer, and the
translation states its consequence in plain terms: the ranking restarts per
region, so small-region customers compete only with each other. It also
surfaces a subtlety hiding inside `RANK()` - ties share a rank, which means
"top three" can return more than three customers per region. A product
manager reviewing this query in Pseudo would catch that; one reviewing the
raw SQL almost certainly would not.
