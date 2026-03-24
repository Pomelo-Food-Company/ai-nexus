# agent-devops — szablon prompta

Użyj tego szablonu gdy zlecasz zadanie agentowi `agent-devops`.
Uruchamiaj po tym jak `dbt-architect` napisał kod i chcesz go zweryfikować przed PR.

---

```
@agent-devops

Branch:
[Nazwa brancha, np. feature/fct_order_items]

Zmienione modele:
[Lista modeli które zmieniłeś/aś, albo "sprawdź sam przez git diff"]

Czy to nowe modele czy modyfikacje?
[Nowe / Modyfikacje / Mieszane]

Dodatkowy kontekst:
[Cokolwiek co agent powinien wiedzieć, np. "celowo pominęłam testy na X bo Y"]
```

---

## Przykład wypełnionego szablonu

```
@agent-devops

Branch:
feature/fct_order_items

Zmienione modele:
fct_order_items (nowy), _facts__models.yml (nowy)

Czy to nowe modele czy modyfikacje?
Nowe

Dodatkowy kontekst:
Model ma być incremental — sprawdź czy config jest poprawny
```

---

## Co agent zrobi automatycznie

- `git diff --name-only main` — znajdzie wszystkie zmienione pliki
- weryfikacja stylu, naming, surrogate keys, field ordering
- `dbt-op compile` + `dbt-op run` + `dbt-op test`
- wygeneruje gotowy opis PR do wklejenia na GitHub

## Kiedy NIE używać

- Gdy kod nie jest jeszcze gotowy — najpierw `dbt-architect`, potem `agent-devops`
- Gdy chcesz tylko przetestować jeden model — użyj `dbt-op test --select model_name` bezpośrednio
