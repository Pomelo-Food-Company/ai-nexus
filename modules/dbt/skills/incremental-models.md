# Incremental Models — BigQuery

## Kiedy stosować incremental vs table

Domyślnie używaj `table`. Przejdź na `incremental` tylko gdy:
- Model `fct_*` przetwarza **>0.3 MB danych dziennie**
- Pełny rebuild jest kosztowny czasowo lub finansowo

```
fct_* z dużym wolumenem  →  incremental
fct_* z małym wolumenem  →  table
dim_*                    →  table (zawsze — dane mutowalne, SCD)
int_*                    →  table lub ephemeral
stg_*                    →  view (domyślnie)
```

Nigdy nie używaj `incremental` dla `dim_*` — wymiary wymagają pełnego rebuildu przy każdej zmianie atrybutu.

---

## Strategia: merge na BigQuery

Dane są **immutable** — źródło nigdy nie modyfikuje rekordów, tylko dodaje nowe. Używamy `merge` żeby obsłużyć late-arriving data i deduplikację.

### Config template

```sql
{{
  config(
    materialized = 'incremental',
    incremental_strategy = 'merge',
    unique_key = 'event_sk',
    partition_by = {
      'field': 'event_date',
      'data_type': 'date',
      'granularity': 'day'
    },
    cluster_by = ['customer_id'],
    on_schema_change = 'append_new_columns'
  )
}}
```

### Filtr incremental

Zawsze używaj `is_incremental()` — bez niego każdy run przebudowuje całą tabelę:

```sql
with

source_events as (
    select * from {{ ref('stg_source__events') }}

    {% if is_incremental() %}
    where event_date >= date_sub(current_date(), interval 3 day)
    {% endif %}
),
```

**Dlaczego 3 dni, nie 1?** Late-arriving data — zdarzenia mogą dotrzeć z opóźnieniem. Okno 3 dni jest bezpiecznym buforem dla większości źródeł. Dostosuj do charakterystyki swojego źródła.

### unique_key

Zawsze surrogate key (`event_sk`) — nie natural key. Surrogate key jest deterministyczny i stabilny:

```sql
{{ dbt_utils.generate_surrogate_key([
    'cast(event_id as string)',
    'cast(event_date as string)'
]) }} as event_sk,
```

### partition_by

Partycjonuj po kolumnie date, nie timestamp — tańsze zapytania i prostszy filtr incremental:

```sql
-- ✅ partycja po date
partition_by = {'field': 'event_date', 'data_type': 'date', 'granularity': 'day'}

-- ❌ unikaj partycji po timestamp — BigQuery zaokrągla do godziny/dnia i tak
partition_by = {'field': 'created_at', 'data_type': 'timestamp'}
```

### cluster_by

Dodaj `cluster_by` na kolumnach często używanych w filtrach downstream:

```sql
cluster_by = ['customer_id', 'brand_id']
```

---

## Testowanie modeli incremental

Zobacz `testing-standards.md` — sekcja "Incremental models".

Spot-check po run: czy liczba wierszy rośnie monotonicznie?
```sql
select event_date, count(*) as rows
from {{ ref('fct_orders') }}
group by 1
order by 1 desc
limit 7
```

---

## Backfill strategy

### Pełny rebuild (najprostszy)

```bash
dbt-op run --select fct_orders --full-refresh
```

Używaj gdy: schemat się zmienił, logika biznesowa się zmieniła, podejrzewasz corrupted data.

**Uwaga na BigQuery:** `--full-refresh` dropuje i odtwarza tabelę. Przy dużych tabelach może być kosztowny — sprawdź rozmiar przed uruchomieniem.

### Partial backfill (dla dużych tabel)

Gdy `--full-refresh` jest za drogi, przebuduj konkretne partycje:

```bash
# Nadpisz ostatnie 30 dni
dbt-op run --select fct_orders \
  --vars '{"start_date": "2024-01-01", "end_date": "2024-01-31"}'
```

W modelu:

```sql
{% if is_incremental() %}
where event_date >= cast(var('start_date', date_sub(current_date(), interval 3 day)) as date)
  and event_date <= cast(var('end_date', current_date()) as date)
{% endif %}
```

### Kiedy używać którego

| Sytuacja | Strategia |
|---|---|
| Zmiana schematu | `--full-refresh` |
| Zmiana logiki biznesowej | `--full-refresh` lub partial backfill |
| Corrupted data w konkretnym okresie | partial backfill |
| Late-arriving data z ostatnich dni | automatyczny merge z oknem 3 dni |
| Nowy model, pierwsza inicjalizacja | `--full-refresh` (pierwszy run) |

---

## Pełny przykład

```sql
{{
  config(
    materialized = 'incremental',
    incremental_strategy = 'merge',
    unique_key = 'order_sk',
    partition_by = {
      'field': 'order_date',
      'data_type': 'date',
      'granularity': 'day'
    },
    cluster_by = ['customer_id'],
    on_schema_change = 'append_new_columns'
  )
}}

with

source_orders as (
    select * from {{ ref('stg_source__orders') }}

    {% if is_incremental() %}
    where order_date >= date_sub(current_date(), interval 3 day)
    {% endif %}
),

dim_customers as (
    select * from {{ ref('dim_customers') }}
),

final as (
    select
        {{ dbt_utils.generate_surrogate_key([
            'cast(source_orders.order_id as string)'
        ]) }} as order_sk,

        source_orders.order_id,

        coalesce(dim_customers.customer_sk, '-1') as customer_sk,
        source_orders.customer_id,

        source_orders.order_status,
        source_orders.total_amount,

        date(source_orders.created_at) as order_date,
        source_orders.created_at

    from source_orders

    left join dim_customers
        on source_orders.customer_id = dim_customers.customer_id
)

select * from final
```
