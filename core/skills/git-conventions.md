# Git Conventions

## Branch Naming
Format: `<type>/<descriptive_name>` in `snake_case`

| Type | Kiedy używać |
|---|---|
| `feature/` | nowa funkcjonalność |
| `fix/` | bug fix |
| `chore/` | maintenance, config, CI/CD |
| `refactor/` | restrukturyzacja bez zmiany zachowania |
| `docs/` | zmiany dokumentacji |

**Examples:**
```
feature/add_customer_lifetime_value
fix/incorrect_order_totals
chore/update_dbt_version
refactor/simplify_int_orders
docs/update_onboarding_guide
```

## Commit Messages
Format: `<type>: <short description>` (lowercase, present tense)

```
feat: add customer segmentation model
fix: correct null handling in fct_orders
chore: bump dbt-core to 1.8
refactor: extract payment logic to intermediate model
docs: add architecture diagram to readme
```
