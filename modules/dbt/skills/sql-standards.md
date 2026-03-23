# SQL Standards

## Formatting
- Indentation: **4 spaces** (WHERE clauses align with keyword)
- **Trailing commas** always
- Line limit: **80 characters**
- **Lowercase** field and function names

## Joins
Always explicit (`inner join`, `left join` — never bare `join`).
Blank line before first join, blank line between each join:

```sql
-- ✅
from source_orders

left join dim_customers
    on source_orders.customer_id = dim_customers.customer_id

left join dim_products
    on source_orders.product_id = dim_products.product_id

-- ❌
join orders on customers.id = orders.customer_id
left join payments on orders.id = payments.order_id
```

## Grouping
```sql
-- ✅
group by 1, 2

-- ❌
group by customer_id, status
```

## Aliases
- Always use `as` keyword when aliasing a field or table
- Table aliases must be descriptive — never single letters:
```sql
-- ✅
left join dim_users as drivers on trips.driver_id = drivers.user_id
left join dim_users as riders on trips.rider_id = riders.user_id

-- ❌
left join dim_users u on trips.driver_id = u.user_id
```

## Column Prefixing
- When joining two or more tables: always prefix column names with table alias
- When selecting from a single table: prefixes not needed

## Set Operations
```sql
-- ✅
union all

-- ❌ (removes duplicates, rarely what you want — be explicit if you need it)
union
```

## General
- Explicit over implicit — always name what you're doing
- Fields before aggregates / window functions
- Aggregate as early as possible before joining
- Consistency over brevity — "NEWLINES ARE CHEAP, BRAIN TIME IS EXPENSIVE"
