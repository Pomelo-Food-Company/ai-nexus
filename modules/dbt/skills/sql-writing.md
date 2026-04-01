---
name: sql-writing
description: How to write dbt SQL models — CTE structure, SQL formatting, Jinja conventions, config blocks
when_to_use: Writing or reviewing any .sql model file
---

# SQL Writing Standards

## CTE Pattern

Every dbt model follows this structure — no exceptions.

### Structure
1. `config` block (if needed)
2. All `ref()` / `source()` calls as CTEs at the top
3. Transformation CTEs
4. `final` CTE
5. `select * from final`

### Template
```sql
{{
  config(
    materialized = 'table'
  )
}}

with

source_data as (

    select * from {{ ref('stg_source__table') }}

),

-- transformed_data: brief explanation if logic is non-obvious
transformed_data as (

    select
        id,
        field_1,
        field_2,
        case
            when condition_1 then 'value_1'
            else 'value_2'
        end as derived_field,
    from source_data
    where field_1 is not null

),

final as (

    select * from transformed_data

)

select * from final
```

### CTE Rules
- All `{{ ref() }}` and `{{ source() }}` calls live in their own CTE at the top — **with blank lines inside the parentheses**
- Never select directly from `ref()` inline — always alias to a CTE first
- `final` is always the last CTE, always selected from at the bottom
- Config block only when needed (materialization, tags, etc.)
- CTEs with non-obvious logic should have a comment above them
- Logic duplicated across multiple models should be extracted into a shared intermediate model, not copy-pasted

---

## SQL Formatting

- Indentation: **4 spaces** (WHERE clauses align with keyword)
- **Trailing commas** always
- Line limit: **80 characters**
- **Lowercase** field and function names

### Joins
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

### Grouping
```sql
-- ✅
group by 1, 2

-- ❌
group by customer_id, status
```

### Aliases
- Always use `as` keyword when aliasing a field or table
- Table aliases must be descriptive — never single letters:
```sql
-- ✅
left join dim_users as drivers on trips.driver_id = drivers.user_id

-- ❌
left join dim_users u on trips.driver_id = u.user_id
```

### Column Prefixing
- When joining two or more tables: always prefix column names with table alias
- When selecting from a single table: prefixes not needed

### Set Operations
```sql
-- ✅
union all

-- ❌ (removes duplicates — be explicit if you need it)
union
```

### General
- Explicit over implicit — always name what you're doing
- Fields before aggregates / window functions
- Aggregate as early as possible before joining
- Consistency over brevity — "NEWLINES ARE CHEAP, BRAIN TIME IS EXPENSIVE"

---

## Jinja Conventions

### Delimiters
Always use spaces inside Jinja delimiters:

```sql
-- ✅
{{ this }}
{{ ref('model_name') }}
{{ source('source_name', 'table_name') }}

-- ❌
{{this}}
{{ref('model_name')}}
```

### Config Block
Only include when needed (materialization override, tags, etc.):
```sql
{{
  config(
    materialized = 'table',
    sort = 'id',
    dist = 'id'
  )
}}
```
Marts should always be `materialized = 'table'`.

Model-specific attributes go in the model's config block. Settings that apply to an entire directory go in `dbt_project.yml` — don't repeat them per-model.
