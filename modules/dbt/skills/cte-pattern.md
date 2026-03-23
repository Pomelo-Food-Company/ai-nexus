# CTE Pattern

Every dbt model follows this structure — no exceptions.

## Structure
1. `config` block (if needed)
2. All `ref()` / `source()` calls as CTEs at the top
3. Transformation CTEs
4. `final` CTE
5. `select * from final`

## Template
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

## Rules
- All `{{ ref() }}` and `{{ source() }}` calls live in their own CTE at the top — **with blank lines inside the parentheses**
- Never select directly from `ref()` inline — always alias to a CTE first
- `final` is always the last CTE, always selected from at the bottom
- Config block only when needed (materialization, tags, etc.)
- CTEs with non-obvious logic should have a comment above them explaining what they do
- Logic duplicated across multiple models should be extracted into a shared intermediate model, not copy-pasted
