# Naming Conventions

## Models
| Layer | Pattern | Example |
|---|---|---|
| Staging | `stg_source__table` | `stg_stripe__payments` |
| Intermediate | `int_entity_verb` | `int_orders_pivoted` |
| Dimension | `dim_entity` | `dim_customer` |
| Fact | `fct_events` | `fct_orders` |

## Fields
- Format: `snake_case`
- Language: business terminology (not source system names)
- Primary keys: `entity_id` (e.g. `customer_id`, `order_id`)
- Timestamps: `event_at` (UTC), `event_at_pt` (other timezone)
- Booleans: `is_` or `has_` prefix (e.g. `is_active`, `has_discount`)
- Prices: decimal format (`19.99` not `1999` cents)

## Field Ordering (mandatory for dims and facts)

Always in this sequence:

```sql
select
    -- 1. Surrogate primary key
    order_sk,

    -- 2. Natural primary key
    order_id,

    -- 3. Surrogate foreign keys (alphabetical)
    customer_sk,
    product_sk,

    -- 4. Natural foreign keys (alphabetical)
    customer_id,
    product_id,

    -- 5. Business fields
    order_status,
    order_type,

    -- 6. Boolean flags
    is_first_order,
    has_discount,

    -- 7. Metrics
    quantity,
    unit_price,
    total_amount,

    -- 8. Timestamps (always last)
    created_at,
    updated_at
```

## Files
- Each subdirectory has a `.yml` file with model tests
- Staging: `_sourcename__sources.yml` + `_sourcename__models.yml`
- Other folders: `_foldername__models.yml`
