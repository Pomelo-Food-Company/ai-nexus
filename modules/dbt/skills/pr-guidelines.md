# PR Guidelines — dbt Projects

## Structure
Every PR must contain:
- **Title** — concise, descriptive, with category emoji
- **Summary** — 2-4 sentences on context and purpose
- **Main Changes** — organized by dbt layer
- **Statistics** — files and lines changed
- **Testing** — commands run and results
- **Checklist** — verified before submission

## Title Emojis
| Emoji | Type |
|---|---|
| 📊 | new/changed data models |
| 🔧 | bug fixes |
| ✨ | new features |
| 📈 | analytics extensions |
| 🏗️ | architectural changes |
| 📝 | documentation |
| ⚡ | performance optimizations |
| 🧪 | tests |

**Good titles:**
```
✅ 📊 Add CLV tracking and customer segmentation to dim_users
✅ 🔧 Fix timezone conversion in staging models
✅ 🏗️ Refactor order models — split into dims and facts
❌ Update files
❌ Fix bug
❌ WIP changes
```

## Change Categorization
Group changes by dbt layer:
- Staging models (`stg_*`)
- Intermediate models (`int_*`)
- Warehouse — Dimensions (`dim_*`)
- Warehouse — Facts (`fct_*`)
- Marts (`mrt_*`)
- Macros & utilities
- Configuration & setup
- Documentation

## Style Guide Checklist
- [ ] Naming conventions followed (`stg_`, `dim_`, `fct_`, `mrt_`)
- [ ] `snake_case` for all names
- [ ] Primary keys named `<object>_id`
- [ ] Timestamps use `_at` suffix
- [ ] Booleans prefixed `is_` or `has_`
- [ ] CTEs at top of file, source/ref CTEs with blank lines
- [ ] Trailing commas, 4-space indentation
- [ ] Explicit join types, descriptive aliases
- [ ] Surrogate keys generated with `dbt_utils`
- [ ] Surrogate FKs use `coalesce` with defaults

## Testing Checklist
- [ ] Every model has entry in `.yml`
- [ ] Primary keys have `unique` + `not_null` tests
- [ ] Relationship tests for foreign keys
- [ ] `dbt-op compile` passed
- [ ] `dbt-op run --select state:modified+` passed
- [ ] `dbt-op test --select state:modified+` passed
- [ ] Manual spot-check in warehouse

## Special Cases

**Breaking changes** — mark `[BREAKING]` in title, add migration guide and impact analysis.

**Large refactors (>10 files)** — consider splitting into smaller PRs. If not possible, add migration strategy and rollback plan.

**Dependencies** — link dependent PRs, specify merge order.

## Best Practices
- One PR = one logical change
- Self-review before submission
- Start with business context, then technical details
- Show impact with statistics
- Update docs/README if needed
