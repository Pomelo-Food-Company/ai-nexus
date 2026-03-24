# dbt-reviewer — szablon prompta

Użyj tego szablonu gdy zlecasz zadanie agentowi `dbt-reviewer`.
Uruchamiaj po tym jak `agent-devops` wystawił PR — to ostatni krok przed mergem.

---

```
@dbt-reviewer

Link do PR:
[URL do PR na GitHubie, np. https://github.com/org/repo/pull/123]

Co zostało zbudowane:
[Krótki opis — 1-2 zdania o celu zmian]

Na co zwrócić szczególną uwagę:
[Opcjonalne — np. "skomplikowana logika w CTE X", "nie byłam pewna co do grain"]

Kontekst biznesowy:
[Opcjonalne — co ten model ma robić, jakie decyzje biznesowe wspiera]
```

---

## Przykład wypełnionego szablonu

```
@dbt-reviewer

Link do PR:
https://github.com/org/dbt-cateringi/pull/42

Co zostało zbudowane:
Nowy model fct_order_items — granularność na poziomie pozycji zamówienia,
używany do analizy marżowości per produkt.

Na co zwrócić szczególną uwagę:
Nie byłam pewna czy logika filtrowania anulowanych zamówień jest poprawna (CTE filtered_orders)

Kontekst biznesowy:
Model będzie zasilał mrt_product_margin — kluczowy dla raportów finansowych
```

---

## Co agent zrobi automatycznie

- `git diff main...HEAD` — pobierze cały diff PR
- review layer dependencies, grain, business logic, edge cases
- zaproponuje refactoring jeśli widzi duplikację lub logikę w złej warstwie
- wyda werdykt: ✅ APPROVED / ⚠️ APPROVED WITH COMMENTS / 🔴 CHANGES REQUESTED

## Czego agent NIE robi

- Nie uruchamia `dbt-op` — to zrobił już `agent-devops`
- Nie sprawdza stylu i formatowania — to zrobił już `agent-devops`
- Skupia się wyłącznie na logice, architekturze i poprawności biznesowej

## Kiedy NIE używać

- Gdy PR nie przeszedł jeszcze przez `agent-devops` — wróć krok wcześniej
- Gdy zmiany są czysto techniczne (np. bump wersji pakietu) — nie ma sensu review logiki
