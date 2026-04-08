---
name: yaml-and-testing
description: YAML documentation structure, file naming, required tests per layer, source freshness, singular tests
when_to_use: Writing or reviewing .yml files, adding tests, documenting models
---

# YAML & Testing Standards

## YAML Formatting

- Indentation: **2 spaces**
- List items should be indented
- New line to separate list items that are dictionaries
- Line limit: **80 characters**

## Model Documentation Template

```yaml
version: 2

models:
  - name: dim_customers
    description: "One row per customer"
    columns:
      - name: customer_sk
        description: "Surrogate key"
        data_tests:
          - unique
          - not_null

      - name: customer_id
        description: "Natural primary key from source"
        data_tests:
          - not_null

      - name: email
        description: "Customer email address"
        data_tests:
          - not_null
          - relationships:
              to: ref('stg_source__users')
              field: email
```

## File Naming

- Staging: `_sourcename__sources.yml` + `_sourcename__models.yml`
- Other folders: `_foldername__models.yml`
- Every subdirectory must have a `.yml` file with tests for each model

---

## Required Tests Per Layer

| Layer | Minimum required |
|---|---|
| `stg_*` | `unique` + `not_null` on PK |
| `int_*` | `unique` + `not_null` on PK |
| `dim_*` | `unique` + `not_null` on surrogate PK; `not_null` on natural PK; `relationships` on FKs |
| `fct_*` | `unique` + `not_null` on surrogate PK; `relationships` on key FKs; date range check on date column |
| `mrt_*` | `unique` + `not_null` on PK |

## Standard YAML Examples

### Primary key
```yaml
- name: order_sk
  description: "Surrogate primary key"
  data_tests:
    - unique
    - not_null
```

### Foreign key
```yaml
- name: customer_sk
  description: "FK to dim_customers"
  data_tests:
    - not_null
    - relationships:
        to: ref('dim_customers')
        field: customer_sk
```

### Date range (fact tables)
```yaml
- name: event_date
  data_tests:
    - not_null
    - dbt_utils.accepted_range:
        min_value: "'2020-01-01'"
        max_value: "current_date()"
```

### Accepted values (enum columns)
```yaml
- name: order_status
  data_tests:
    - accepted_values:
        values: ['pending', 'processing', 'completed', 'cancelled']
```

Use for any column with a fixed set of expected values (`status`, `type`, `channel`, etc.).

---

## Source Freshness

Configure in `_sourcename__sources.yml` for every source table:

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
        freshness:
          warn_after: {count: 1, period: hour}
          error_after: {count: 6, period: hour}
```

```bash
dbt-op source freshness
dbt-op source freshness --select source:stripe
```

**Rules:**
- Every source table must have `loaded_at_field` defined
- `warn_after` < `error_after` always

---

## Singular Tests

For complex business logic that can't be expressed with schema tests — put SQL in `tests/` folder:

```sql
-- tests/assert_fct_orders_no_negative_amounts.sql
select order_id
from {{ ref('fct_orders') }}
where total_amount < 0
```

The test passes when the query returns **zero rows**.

Use when:
- Validating cross-model business rules
- Checking aggregated invariants
- Logic is too complex for a schema test

---

## Incremental Model Tests

`dbt test` checks current table state — not incremental logic. Always test separately after each run:

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

---

## Running Tests

```bash
# All modified models and downstream
dbt-op test --select state:modified+

# Specific model
dbt-op test --select model_name

# Model and all downstream
dbt-op test --select model_name+
```
