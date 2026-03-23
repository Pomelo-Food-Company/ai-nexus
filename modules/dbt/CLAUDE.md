# dbt Project Context

## Architecture
Layered dbt project:
- **`stg_`** — staging, 1:1 with source tables, light transformations only
- **`int_`** — intermediate, complex logic, bridge between staging and marts
- **`dim_` / `fct_`** — warehouse star schema
- **marts** — wide, denormalized, business-ready tables

```
models/
├── staging/
├── warehouse/
└── marts/
```

## Critical Rules
- Only `stg_` models select from `source()` — all others use `ref()` only
- Marts configured as `materialized='table'`
- Every model needs primary key tests (unique, not_null)
- Business terminology over technical source names

## Skills
Before writing SQL: @.claude/shared/modules/dbt/skills/sql-standards.md
Before naming anything: @.claude/shared/modules/dbt/skills/naming-conventions.md
When creating a new model: @.claude/shared/modules/dbt/skills/cte-pattern.md
When working with dims/facts: @.claude/shared/modules/dbt/skills/surrogate-keys.md
