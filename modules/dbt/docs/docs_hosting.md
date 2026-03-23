# Hosting dokumentacji dbt na Cloudflare Pages

## Architektura

Każdy pipeline dbt ma osobny projekt na Cloudflare Pages, co pozwala na niezależne zarządzanie dostępem per pipeline.

```
dbt-cateringi-docs.pages.dev  →  Cloudflare Pages  →  Cloudflare Access (lista emaili)
dbt-inny-docs.pages.dev       →  osobny projekt     →  osobna policy dostępowa
```

## Wymagania

- **wrangler** - dostępny przez `npx` (nie wymaga globalnej instalacji)
- **Cloudflare API Token** - z uprawnieniami:
  - `Account` → `Cloudflare Pages` → `Edit`
  - `Account` → `Account Settings` → `Read`
  - `User` → `Memberships` → `Read`
  - `User` → `User Details` → `Read`
- **Cloudflare Account ID** - Dashboard → Overview → po prawej stronie

## Pliki w repo

| Plik | Opis |
|------|------|
| `scripts/deploy-docs.sh` | Skrypt do ręcznego deployu (tylko upload do CF) |
| `dev_ops/setup-cloudflare-access.sh` | Konfiguracja Cloudflare Access (w repo `dev_ops`) |
| `.github/workflows/deploy_to_prod.yml` | Automatyczny deploy po merge do main |

## Ręczny deploy (lokalnie)

```bash
# 1. Aktywuj venv + wygeneruj docs
source ../dbt-env/bin/activate
DBT_PROFILES_DIR=. dbt-op docs generate -- --target prod

# 2. Zaloguj się do 1Password (konto Pomelo) + deploy
eval $(op signin --account pomelo)
CLOUDFLARE_API_TOKEN=$(op read "op://SuperAdmin/Claudflare - API Token - Pomelo/credential") \
./scripts/deploy-docs.sh
```

## Automatyczny deploy (CI/CD)

Po każdym merge do main workflow `deploy_to_prod.yml` automatycznie:
1. Buduje modele (`dbt build --target prod`)
2. Generuje dokumentację (`dbt docs generate --target prod`)
3. Deployuje na Cloudflare Pages

### Wymagane GitHub Secrets

| Secret | Opis |
|--------|------|
| `CLOUDFLARE_API_TOKEN` | API token z odpowiednimi uprawnieniami |
| `CLOUDFLARE_ACCOUNT_ID` | ID konta Cloudflare |

## Zabezpieczenie dostępu (Cloudflare Access)

Dostęp do dokumentacji jest ograniczony przez Cloudflare Access.
Użytkownicy widzą stronę logowania i otrzymują jednorazowy kod na email.

Darmowy plan: do 50 użytkowników.

### Konfiguracja przez skrypt

```bash
# 1. Zaloguj się do 1Password (konto Pomelo)
eval $(op signin --account pomelo)

# 2. Uruchom skrypt (z repo dev_ops)
#    Wymaga dodatkowego uprawnienia tokena: Account → Access: Apps and Policies → Edit
cd ~/Repozytoria/dev_ops
CLOUDFLARE_API_TOKEN=$(op read "op://SuperAdmin/Claudflare - API Token - Pomelo/credential") \
CLOUDFLARE_ACCOUNT_ID=$(op read "op://SuperAdmin/Claudflare - API Token - Pomelo/Account ID") \
./setup-cloudflare-access.sh
```

### Dodawanie/usuwanie użytkowników

Edytuj listę `ALLOWED_EMAILS` w `dev_ops/setup-cloudflare-access.sh` i uruchom skrypt ponownie.
Skrypt jest idempotentny - wykrywa istniejącą konfigurację i ją aktualizuje.

### Konfiguracja ręczna (przez dashboard)

1. Otwórz **https://one.dash.cloudflare.com**
2. **Access** → **Applications** → **Add an application**
3. Typ: **Self-hosted**
4. Konfiguracja:
   - Application name: `dbt-cateringi-docs`
   - Session duration: `24 hours`
   - Application domain: `dbt-cateringi-docs.pages.dev`
5. Policy:
   - Policy name: `Allowed users`
   - Action: `Allow`
   - Selector: `Emails` → dodaj emaile uprawnionych osób

## Dodawanie nowego pipeline'a

Aby wystawić docs dla kolejnego pipeline'a dbt:

1. Utwórz projekt na Cloudflare Pages:
   - Dashboard → Workers & Pages → Create → Pages → Direct Upload
   - Nazwa: `dbt-NAZWA-docs`
2. Deploy analogicznie (zmień `--project-name` w komendach)
3. Skonfiguruj osobną policy w Cloudflare Access
4. Dodaj step w odpowiednim workflow CI/CD

## Rozwiązywanie problemów

### `wrangler login` - bot challenge (403)
Cloudflare blokuje OAuth. Użyj API tokena zamiast logowania:
```bash
export CLOUDFLARE_API_TOKEN="token"
```

### Deploy idzie na preview zamiast production
Dodaj flagę `--branch=main`:
```bash
npx wrangler pages deploy target --project-name=dbt-cateringi-docs --branch=main
```

### Stara wersja docs po deploy
Sprawdź czy `dbt docs generate` się wykonało poprawnie - w `target/` muszą być pliki `index.html`, `manifest.json`, `catalog.json`.
