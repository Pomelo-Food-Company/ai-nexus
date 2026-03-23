# Jinja Standards

## Delimiters
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

## Newlines for Readability
Use newlines to visually separate logical blocks of Jinja:

```sql
{{
  config(
    materialized = 'table'
  )
}}

with

source_data as (
    select * from {{ source('source_name', 'table_name') }}
),
```

## Surrogate Key Generation
```sql
-- Single key
{{ dbt_utils.generate_surrogate_key([
    'cast(order_id as string)'
]) }} as order_sk,

-- Composite key
{{ dbt_utils.generate_surrogate_key([
    'cast(order_id as string)',
    'cast(line_number as string)'
]) }} as order_item_sk,
```

## Config Block
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

Model-specific attributes (sort/dist keys, custom materializations) go in the model's config block. Settings that apply to an entire directory go in `dbt_project.yml` — don't repeat them per-model.
