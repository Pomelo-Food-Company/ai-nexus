# YAML Standards

## Formatting
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
        tests:
          - unique
          - not_null

      - name: customer_id
        description: "Natural primary key from source"
        tests:
          - not_null

      - name: email
        description: "Customer email address"
        tests:
          - not_null
          - relationships:
              to: ref('stg_source__users')
              field: email
```

## File Naming
- Staging: `_sourcename__sources.yml` + `_sourcename__models.yml`
- Other folders: `_foldername__models.yml`
- Every subdirectory must have a `.yml` file with tests for each model

## Testing
See `testing-standards.md`.
