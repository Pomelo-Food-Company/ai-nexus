# dbt Local Setup Pattern

## Architektura credentials

Lokalne środowisko używa tego samego mechanizmu co CI/CD — `profiles.yml` w repo, credentials wstrzykiwane przez zmienne środowiskowe:

```
profiles.yml (w repo)
  └── env_var('DBT_SERVICE_ACCOUNT_PRIVATE_KEY')
        ├── lokalnie:  1Password CLI (op run)
        └── CI/CD:     GitHub Secrets
```

## Funkcja `dbt-op` (~/.zshrc)

Generyczny wrapper który działa w dowolnym projekcie dbt z `profiles.yml` i `.env.op`:

```bash
dbt-op() {
    if [[ ! -f ".env.op" ]]; then
        echo "Error: .env.op not found. Copy .env.op.example to .env.op first."
        return 1
    fi
    if [[ ! -f "profiles.yml" ]]; then
        echo "Error: profiles.yml not found in current directory."
        return 1
    fi
    DBT_PROFILES_DIR=. op run --env-file=".env.op" -- dbt "$@"
}
```

Użycie — z katalogu projektu:
```bash
dbt-op run
dbt-op run --select model_name
dbt-op run --select +model_name       # model + upstream
dbt-op run --select model_name+       # model + downstream
dbt-op build                          # run + test
dbt-op test
dbt-op compile
dbt-op debug                          # sprawdź połączenie
```

## Targets

| Target | Użycie |
|--------|--------|
| `dev` | lokalne developowanie (domyślny) |
| `ci` | CI/CD |
| `prod` | produkcja — ostrożnie! |

```bash
dbt-op run -t dev
dbt-op run -t prod
```

## Wymagania projektu

Każdy projekt dbt który używa tego wzorca potrzebuje:
- `profiles.yml` w katalogu głównym (commitowany)
- `.env.op` z referencjami do secret managera (NIE commitowany, w `.gitignore`)
- `.env.op.example` — szablon dla nowych członków zespołu (commitowany)

## CI/CD — spójność z lokalnym flow

```yaml
# .github/workflows/ci.yml
env:
  DBT_PROFILES_DIR: .
  DBT_SERVICE_ACCOUNT_PRIVATE_KEY: ${{ secrets.DBT_SERVICE_ACCOUNT_PRIVATE_KEY }}
steps:
  - run: dbt build --target ci
```

Szczegóły konfiguracji (konto 1Password, vault, konkretne zmienne) → `_project_docs/dbt_local_setup.md` w projekcie.
