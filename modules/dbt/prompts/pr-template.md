# PR Template

```markdown
## [Emoji] [Title describing main goal]

### 📊 Summary
[2-4 sentences: what changed, why, business impact]

### 🎯 Main Changes

#### 1. [Category, e.g. Staging Models]
* **[file_name]**: [description of changes]

#### 2. [Category, e.g. Warehouse — Dimensions]
* **[file_name]**: [description of changes]

#### 3. [Category, e.g. Marts]
* **[file_name]**: [description of changes]

### 🧪 Testing & Verification
- [ ] `dbt compile` — ✅ Success
- [ ] `dbt run --select [models]` — ✅ Success
- [ ] `dbt test --select [models]` — ✅ Success
- [ ] Manual verification in warehouse — ✅ Success

### 📈 Statistics
* **Files changed**: X
* **Lines added**: +X
* **Lines deleted**: -X
* **New models**: X
* **Updated models**: X

### ✅ Style Guide Checklist
- [ ] Model naming conventions followed
- [ ] SQL formatting standards applied
- [ ] CTE structure correct
- [ ] Surrogate keys implemented
- [ ] Primary keys properly named and tested
- [ ] Documentation complete

### 🔗 Related
* Issue: #XXX (if applicable)
* Dependencies: [list if any]
* Breaking changes: [description if any]

### 💡 Additional Notes
[Optional: context, architectural decisions, follow-up work]
```
