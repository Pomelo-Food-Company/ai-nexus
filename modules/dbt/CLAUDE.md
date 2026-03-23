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

## ⚠️ Uruchamianie dbt — zawsze `dbt-op`

Nigdy nie sugeruj `dbt run`, `dbt test`, `dbt build` ani żadnej innej komendy `dbt` bezpośrednio.
Zawsze używaj `dbt-op` zamiast `dbt`:

```bash
# ✅
dbt-op run
dbt-op run --select model_name
dbt-op test --select model_name
dbt-op build

# ❌
dbt run
dbt test
```

`dbt-op` to wrapper który wstrzykuje credentials z 1Password. Bez niego połączenie z bazą danych nie zadziała.

## Critical Rules
- Only `stg_` models select from `source()` — all others use `ref()` only
- Marts configured as `materialized='table'`
- Every model needs primary key tests (unique, not_null)
- Business terminology over technical source names

## Skills
Before writing SQL: @.claude/shared/modules/dbt/skills/sql-standards.md <br>
Before naming anything: @.claude/shared/modules/dbt/skills/naming-conventions.md <br>
When creating a new model: @.claude/shared/modules/dbt/skills/cte-pattern.md <br>
When working with dims/facts: @.claude/shared/modules/dbt/skills/surrogate-keys.md <br>
When writing YAML: @.claude/shared/modules/dbt/skills/yaml-standards.md <br>
When writing Jinja: @.claude/shared/modules/dbt/skills/jinja-standards.md <br>
