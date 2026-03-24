# Surrogate Keys

Required for all dimension and fact models.

## Why
- Better join performance than natural keys
- Handle missing/null foreign keys gracefully
- Enable slowly changing dimensions (SCD)
- Stable keys across environments

## Generating Surrogate PKs

```sql
-- Single natural key
{{ dbt_utils.generate_surrogate_key([
    'cast(order_id as string)'
]) }} as order_sk,

-- Composite natural key
{{ dbt_utils.generate_surrogate_key([
    'cast(order_id as string)',
    'cast(line_number as string)'
]) }} as order_item_sk,
```

## Surrogate FK References with Defaults

When joining to a dimension, always use `coalesce` to handle missing rows:

```sql
-- String surrogate key (default: '-1')
coalesce(dim_customers.customer_sk, '-1') as customer_sk,

-- Integer surrogate key (default: -1)
coalesce(dim_products.product_sk, -1) as product_sk,
```

Never leave surrogate FKs nullable — a missing dimension gets `-1` / `'-1'`.

## Full Example

```sql
with

source_orders as (
    select * from {{ ref('stg_source__orders') }}
),

dim_customers as (
    select * from {{ ref('dim_customers') }}
),

dim_products as (
    select * from {{ ref('dim_products') }}
),

final as (
    select
        {{ dbt_utils.generate_surrogate_key([
            'cast(source_orders.order_id as string)'
        ]) }} as order_sk,

        source_orders.order_id,

        coalesce(dim_customers.customer_sk, '-1') as customer_sk,
        coalesce(dim_products.product_sk, '-1') as product_sk,

        source_orders.customer_id,
        source_orders.product_id,

        source_orders.order_status,
        source_orders.total_amount,

        source_orders.created_at

    from source_orders

    left join dim_customers
        on source_orders.customer_id = dim_customers.customer_id

    left join dim_products
        on source_orders.product_id = dim_products.product_id
)

select * from final
```
