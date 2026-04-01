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

## Available Skills

Load skill files on demand — read only what the current task requires.
Path prefix: `.claude/shared/modules/dbt/skills/`

| skill | description | when to load |
|---|---|---|
| `architecture-guideline.md` | Layer boundaries, golden reference table, materialization defaults | Every task — load first |
| `naming-conventions.md` | Model prefixes per layer, field naming, field ordering for dims/facts | Creating new models or fields |
| `sql-writing.md` | CTE structure, SQL formatting, Jinja conventions, config blocks | Writing or reviewing any .sql model |
| `surrogate-keys.md` | SK generation with dbt_utils, coalesce defaults for FK references | Creating or reviewing dim_* / fct_* |
| `yaml-and-testing.md` | YAML structure, required tests per layer, source freshness, singular tests | Writing or reviewing .yml files / tests |
| `incremental-models.md` | merge strategy on BigQuery, partition_by, backfill | fct_* models with high data volume |
| `pr-guidelines.md` | PR title format, change categorization, checklists | Generating a PR description |
