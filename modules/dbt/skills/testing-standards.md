# Testing Standards — dbt

## Required tests per layer

| Layer | Minimum required |
|---|---|
| `stg_*` | `unique` + `not_null` on PK |
| `int_*` | `unique` + `not_null` on PK |
| `dim_*` | `unique` + `not_null` on surrogate PK; `not_null` on natural PK; `relationships` on FKs |
| `fct_*` | `unique` + `not_null` on surrogate PK; `relationships` on key FKs; date range check on date column |
| `mrt_*` | `unique` + `not_null` on PK |

## Standard YAML examples

### Primary key
```yaml
- name: order_sk
  description: "Surrogate primary key"
  tests:
    - unique
    - not_null
```

### Foreign key
```yaml
- name: customer_sk
  description: "FK to dim_customers"
  tests:
    - not_null
    - relationships:
        to: ref('dim_customers')
        field: customer_sk
```

### Date range (fact tables)
```yaml
- name: event_date
  tests:
    - not_null
    - dbt_utils.accepted_range:
        min_value: "'2020-01-01'"
        max_value: "current_date()"
```

### Accepted values (enum columns)
```yaml
- name: order_status
  tests:
    - accepted_values:
        values: ['pending', 'processing', 'completed', 'cancelled']
```

Use for any column with a fixed set of expected values (`status`, `type`, `channel`, etc.).

## Source freshness

Configure in `_sourcename__sources.yml` for every source table that feeds staging models:

```yaml
version: 2

sources:
  - name: stripe
    database: raw
    schema: stripe
    freshness:
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}
    loaded_at_field: _fivetran_synced

    tables:
      - name: orders
      - name: payments
        freshness:                              # override per table if needed
          warn_after: {count: 1, period: hour}
          error_after: {count: 6, period: hour}
```

Run freshness check:
```bash
dbt-op source freshness
dbt-op source freshness --select source:stripe  # specific source
```

**Rules:**
- Every source table must have `loaded_at_field` defined — use `_fivetran_synced` or equivalent ingestion timestamp
- Set thresholds based on source SLA, not gut feeling — check with data engineering how often the source syncs
- `warn_after` < `error_after` always
- Freshness failures block downstream `stg_*` models from being considered fresh

## Singular tests

For complex business logic that can't be expressed with schema tests — put SQL in `tests/` folder:

```sql
-- tests/assert_fct_orders_no_negative_amounts.sql
-- Every order amount must be non-negative

select order_id
from {{ ref('fct_orders') }}
where total_amount < 0
```

The test passes when the query returns **zero rows**.

Use singular tests when:
- Validating cross-model business rules (e.g. every order in fct must exist in dim_customers)
- Checking aggregated invariants (e.g. total revenue can't drop more than 50% day-over-day)
- Logic is too complex for a schema test

## Incremental models

`dbt test` checks current table state — not incremental logic. Always test separately after each run.

```bash
dbt-op run --select fct_orders
dbt-op test --select fct_orders
```

Check for duplicates after merge:
```sql
select order_sk, count(*) as cnt
from fct_orders
group by 1
having cnt > 1
```

## Running tests

```bash
# All modified models and their downstream dependencies
dbt-op test --select state:modified+

# Specific model
dbt-op test --select model_name

# Model and all downstream
dbt-op test --select model_name+
```
