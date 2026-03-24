# dbt-architect — szablon prompta

Użyj tego szablonu gdy zlecasz zadanie agentowi `dbt-architect`.
Wypełnij wszystkie sekcje — im więcej kontekstu, tym lepsza propozycja architektoniczna.

---

```
@dbt-architect

Kontekst biznesowy:
[Co to za encja lub zdarzenie? Jaki problem biznesowy rozwiązujesz?]

Wymaganie:
[Co chcesz zbudować? Jedno zdanie.]

Źródła danych:
[Jakie modele/tabele są dostępne? Wklej nazwy kolumn jeśli masz.]

Grain:
[Jeden wiersz = co? Np. "jeden wiersz na zamówienie" / "nie wiem, zaproponuj"]

Oczekiwany output:
[Lista pól które chcesz widzieć, albo "zaproponuj sam"]

Dodatkowe wymagania:
[Np. incremental, konkretna materializacja, zależności których nie możesz zmienić]
```

---

## Przykład wypełnionego szablonu

```
@dbt-architect

Kontekst biznesowy:
Chcemy śledzić zamówienia klientów z podziałem na produkty,
żeby analizować marżowość per produkt i per klient.

Wymaganie:
Zbuduj fct_order_items dla zamówień z caterings.

Źródła danych:
stg_caterings__ecommerce_order, stg_caterings__ecommerce_order_item,
dim_products, dim_customers

Grain:
Jeden wiersz na pozycję zamówienia (order_id + product_id)

Oczekiwany output:
Klucze, status zamówienia, cena jednostkowa, ilość, wartość łączna, timestampy

Dodatkowe wymagania:
Tabela jest duża — rozważ incremental jeśli ma sens
```

---

## Co możesz pominąć

Agent sam zadba o:
- zasady SQL, naming, surrogate keys (czyta ze skilli)
- `dbt-op` zamiast `dbt`
- propozycję architektury przed implementacją (Phase 2)
- czekanie na Twój approval przed pisaniem kodu (Phase 3)
