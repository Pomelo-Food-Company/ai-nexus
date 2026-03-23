# ai-nexus

Centralne repozytorium plików kontekstowych, agentów, skilli i promptów dla Claude Code.

Działa jako **single source of truth** — poszczególne projekty pobierają z niego to czego potrzebują przez skrypt `sync-claude.sh`.

---

## Struktura

```
ai-nexus/
├── core/                        # kontekst wspólny dla wszystkich projektów
│   ├── CLAUDE.md                # główny kontekst: firma, dane, zasady pracy
│   ├── agents/                  # agenci generyczni
│   ├── skills/
│   │   └── git-conventions.md
│   └── prompts/
│
└── modules/                     # kontekst domenowy — projekt wybiera co potrzebuje
    ├── dbt/
    │   ├── CLAUDE.md
    │   ├── agents/
    │   │   ├── dbt-architect.md    # step 1: projektuje i pisze kod
    │   │   ├── agent-devops.md     # step 2: syntax, testy, PR description
    │   │   └── dbt-reviewer.md     # step 3: review logiki biznesowej
    │   ├── skills/
    │   │   ├── sql-standards.md
    │   │   ├── naming-conventions.md
    │   │   ├── cte-pattern.md
    │   │   ├── surrogate-keys.md
    │   │   ├── yaml-standards.md
    │   │   ├── jinja-standards.md
    │   │   ├── git-conventions.md
    │   │   └── pr-guidelines.md
    │   ├── prompts/
    │   │   └── pr-template.md
    │   └── docs/
    │       ├── docs_hosting.md
    │       └── dbt-local-setup-pattern.md
    ├── bi-reports/
    │   ├── CLAUDE.md
    │   └── skills/
    └── python/
        ├── CLAUDE.md
        └── skills/
```

---

## Jak używać w projekcie

### 1. Dodaj skrypt sync

Skopiuj `sync-claude.sh` do swojego repo i dostosuj moduły które potrzebujesz:

```bash
#!/bin/bash
# sync-claude.sh
# Pobiera pliki z ai-nexus do .claude/shared/
# Uruchom: bash sync-claude.sh

GITHUB_USER="TWOJ_USERNAME"
NEXUS="https://raw.githubusercontent.com/${GITHUB_USER}/ai-nexus/main"
DEST=".claude/shared"

echo "🔄 Syncing from ai-nexus..."

# --- core (zawsze) ---
mkdir -p $DEST/core/agents $DEST/core/skills $DEST/core/prompts
curl -sf $NEXUS/core/CLAUDE.md -o $DEST/core/CLAUDE.md

# --- moduł domenowy (odkomentuj to czego potrzebujesz) ---
# mkdir -p $DEST/modules/dbt/agents $DEST/modules/dbt/skills
# curl -sf $NEXUS/modules/dbt/CLAUDE.md -o $DEST/modules/dbt/CLAUDE.md

echo "✅ Done. Commit the changes to .claude/shared/"
```

### 2. Struktura w projekcie

Po odpaleniu skryptu Twój projekt wygląda tak:

```
my-project/
├── .claude/
│   ├── shared/          # ⚠️ AUTO-GENERATED — nie edytuj ręcznie
│   │   ├── core/
│   │   └── modules/dbt/
│   └── local/           # ✅ Twoje lokalne rozszerzenia — edytuj do woli
│       └── CLAUDE.md
└── CLAUDE.md            # importuje shared/ i local/
```

### 3. Główny CLAUDE.md projektu

```markdown
# Project context

@.claude/shared/core/CLAUDE.md
@.claude/shared/modules/dbt/CLAUDE.md
@.claude/local/CLAUDE.md
```

---

## CLAUDE.md vs Agenci vs Skille

Trzy różne narzędzia, trzy różne cele — i bezpośredni wpływ na zużycie tokenów.

| | CLAUDE.md | Agent | Skill |
|---|---|---|---|
| **Kiedy się ładuje** | zawsze, automatycznie | na żądanie | na żądanie |
| **Cel** | pamięć długoterminowa | złożony przepływ z narzędziami | jedna precyzyjna umiejętność |
| **Docelowa długość** | ~200–400 tokenów | ~300–800 tokenów | ~100–300 tokenów |

**CLAUDE.md** — to co Claude powinien wiedzieć od pierwszego zdania każdej sesji. Kontekst projektu, technologie, globalne zasady. Musi być krótki — ładuje się zawsze, niezależnie od zadania.

**Agent** — samodzielnie wykonuje sekwencję kroków, używa narzędzi (bash, pliki, search) i podejmuje decyzje po drodze. Uruchamiasz go świadomie do konkretnego, złożonego zadania, np. code review modeli dbt.

**Skill** — ekspert od jednej konkretnej czynności. Zero zbędnego kontekstu, maksymalna precyzja. Np. zasady formatowania SQL, naming conventions, checklista dla PR.

> **Najczęstszy błąd:** wrzucanie wszystkiego do CLAUDE.md. Efekt: marnujesz tokeny na kontekst potrzebny raz na tydzień, a Claude ma mniej miejsca na Twój aktualny kod.

```
Czy Claude potrzebuje tego ZAWSZE, od pierwszego zdania?
├── TAK → CLAUDE.md
└── NIE → Czy to sekwencja kroków z narzędziami?
           ├── TAK → agent
           └── NIE → skill
```

---

## Zasady kontrybucji

- **`core/`** — tylko zmiany które mają sens dla *każdego* projektu
- **`modules/`** — zmiany domenowe, nie wrzucaj tu nic co jest specyficzne dla jednego projektu
- **`modules/*/docs/`** — dokumentacja operacyjna (setup, deployment, howto) reużywalna między projektami tej domeny
- Jeśli coś jest specyficzne dla jednego projektu → należy do `.claude/local/` w tamtym repo, nie tutaj
- Testuj zmiany lokalnie zanim pushniesz do `main` — `main` trafia do wszystkich projektów po sync

---

## Aktualizacja w projekcie

```bash
bash sync-claude.sh
git add .claude/shared/
git commit -m "chore: sync claude context from ai-nexus"
```