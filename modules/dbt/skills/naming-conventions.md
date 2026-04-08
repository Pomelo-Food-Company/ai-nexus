---
name: naming-conventions
description: Model prefixes per layer, field naming rules, field ordering for dims/facts, file naming for YAML
when_to_use: Creating new models or fields, reviewing names, designing a model's column list
---

# Naming Conventions

## Models
| Layer | Pattern | Example |
|---|---|---|
| Base | `base__source_object` | `base__stripe_payments` |
| Staging | `stg_source__table` | `stg_stripe__payments` |
| Intermediate | `int_[entity]s_[verb]s` | `int_orders_pivoted` |
| Dimension | `dim_entity` | `dim_customers` |
| Fact | `fct_events` | `fct_orders` |

All model names should be **plural** (e.g. `stg_stripe__invoices`, not `stg_stripe__invoice`).

Base models (`base__`) are used only when a source table requires pre-staging cleanup before the standard `stg_` layer.

## Fields
- Format: `snake_case`
- Language: business terminology (not source system names)
- Primary keys: `entity_id` (e.g. `customer_id`, `order_id`)
- Timestamps: use past-tense action verb as prefix — `{action_event_name}_at/date/time`:
  - `{action}_at` — full datetime, UTC (e.g. `delivered_at`, `purchased_at`)
  - `{action}_date` — date only, YYYY-MM-DD (e.g. `delivered_date`, `purchased_date`)
  - `{action}_time` — time only, HH:MM (e.g. `delivered_time`, `purchased_time`)
- If the action verb is not obvious from context — ask the user before naming the column
- Booleans: `is_` or `has_` prefix (e.g. `is_active`, `has_discount`)
- Prices: decimal format (`19.99` not `1999` cents)
- **Price suffixes**: product/ecommerce prices are implicitly gross — do NOT add `_gross` suffix. Use `_gross` / `_net` only in accounting/financial contexts where the distinction matters (e.g. `invoice_amount_net`, `payment_amount_gross`).
- Avoid SQL reserved words as column names (e.g. `order`, `date`, `value`, `name`)
- Use consistent field names across models — e.g. a FK to `customers` is always `customer_id`, never `user_id`

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
