# dbt Architecture Guideline

## Layer Architecture

dbt follows a strict layered architecture. Never violate layer boundaries.

```
sources
    ↓
staging (stg_*)      → raw sources, light transformation only
    ↓
intermediate (int_*) → complex logic, bridges staging to dims/facts
    ↓
dim_* / fct_*        → star schema — staging + intermediate only
    ↓
marts (mrt_*)        → denormalized reports — dims + facts + intermediate only
```

### Golden Rules

| Layer | Can reference | Cannot reference |
|---|---|---|
| `stg_*` | `source()` only | any `ref()` |
| `int_*` | `stg_*` only | `dim_*`, `fct_*`, `mrt_*` |
| `dim_*` | `stg_*`, `int_*` | `dim_*`, `fct_*`, `mrt_*` |
| `fct_*` | `stg_*`, `int_*` | `dim_*`, `fct_*`, `mrt_*` |
| `mrt_*` | `dim_*`, `fct_*`, `int_*` | `stg_*`, other `mrt_*` |

Never: `dim→dim`, `fct→dim`, `dim→fct`, `anything→mrt_*` as a dependency.

## Business Logic Placement

| Logic type | Layer |
|---|---|
| Raw transformation, type casting, column renaming | `stg_*` |
| Complex calculations, multi-source joins, pre-aggregations | `int_*` |
| Entity attributes, slowly changing dimensions | `dim_*` |
| Measurable events, metrics, grain-level facts | `fct_*` |
| Wide denormalized reports, business-ready tables | `mrt_*` |

## Materializations

Default materialization per layer:

| Layer | Default | Override when |
|---|---|---|
| `stg_*` | `view` | Almost never |
| `int_*` | `table` | `ephemeral` for simple pass-throughs with no direct queries |
| `dim_*` | `table` | Never — dims require full rebuild on every attribute change |
| `fct_*` | `table` | → `incremental` when >0.3 MB/day (see `incremental-models.md`) |
| `mrt_*` | `table` | Almost never |

## Bottom-up Implementation

Always implement from the lowest layer up — never start at the mart:

```
❌ User asks for field in mart → modify mart directly
✅ User asks for field in mart → identify correct layer
   → check staging has the data → implement bottom-up
```
