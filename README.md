# Project-Grocery-Store-Sales-SQL

# Grocery Store Sales Analysis

## Descrizione del Progetto

Questo progetto di analisi dati è stato sviluppato per **FoodYum**, una catena di negozi di alimentari statunitense. L'obiettivo principale è garantire che il negozio mantenga un'offerta diversificata di prodotti in tutte le categorie e fasce di prezzo, per servire efficacemente una clientela ampia e variegata.

## Contesto

Con l'aumento dei costi alimentari, FoodYum vuole assicurarsi di avere prodotti che coprano diverse fasce di prezzo in tutte le categorie merceologiche (Produce, Meat, Dairy, Bakery, Snacks), garantendo accessibilità a tutti i clienti.

## Dataset

Il dataset contiene informazioni sui prodotti disponibili nel catalogo FoodYum:

| Colonna | Tipo | Descrizione |
|---------|------|-------------|
| `product_id` | Nominale | Identificatore univoco del prodotto |
| `product_type` | Nominale | Categoria del prodotto (Produce, Meat, Dairy, Bakery, Snacks) |
| `brand` | Nominale | Marca del prodotto (7 valori possibili) |
| `weight` | Continuo | Peso del prodotto in grammi |
| `price` | Continuo | Prezzo di vendita in dollari USA |
| `average_units_sold` | Discreto | Media delle unità vendute mensilmente |
| `year_added` | Nominale | Anno di introduzione del prodotto nel catalogo |
| `stock_location` | Nominale | Magazzino di origine (A, B, C, D) |

## Analisi Effettuate

### Task 1: Identificazione Valori Mancanti
Identificazione del numero di prodotti con `year_added` mancante, a causa di un bug nel sistema nel 2022.

**Query:**
```sql
SELECT COUNT(*) AS missing_year
FROM public.products
WHERE public.products.year_added IS NULL;
```

**Risultato:** 170 prodotti con anno mancante

---

### Task 2: Pulizia dei Dati
Implementazione di regole di pulizia per garantire la qualità del dataset:
- Sostituzione valori nulli in `product_type` e `brand` con "Unknown"
- Sostituzione valori nulli in `weight` e `price` con la mediana
- Sostituzione valori nulli in `average_units_sold` con 0
- Sostituzione valori nulli in `year_added` con 2022
- Normalizzazione di `stock_location` in maiuscolo

**Query:**
```sql
WITH medians AS (
    SELECT
        PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY 
            CAST(REPLACE(weight, ' grams', '') AS NUMERIC)
        ) AS median_weight,
        PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY price) AS median_price
    FROM public.products
)
SELECT 
    p.product_id,
    COALESCE(p.product_type, 'Unknown') AS product_type,
    CASE 
        WHEN p.brand IS NULL THEN 'Unknown'
        WHEN p.brand = '-' THEN 'Unknown'
        ELSE p.brand
    END AS brand,
    COALESCE(
        CAST(REPLACE(p.weight, ' grams', '') AS NUMERIC), 
        m.median_weight
    ) AS weight,
    COALESCE(p.price, m.median_price) AS price,
    COALESCE(p.average_units_sold, 0) AS average_units_sold,
    COALESCE(p.year_added, 2022) AS year_added,
    CASE 
        WHEN p.stock_location IS NULL THEN 'Unknown'
        WHEN p.stock_location = '-' THEN 'Unknown'
        ELSE UPPER(p.stock_location)
    END AS stock_location
FROM public.products p
CROSS JOIN medians m;
```

**Risultato:** Dataset pulito con 1700 righe

---

### Task 3: Analisi Range Prezzi
Calcolo del range di prezzi (minimo e massimo) per ciascuna categoria di prodotto.

**Query:**
```sql
SELECT 
    product_type,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM public.products
GROUP BY product_type;
```

**Risultati:**
- **Snacks**: $5.20 - $10.72
- **Produce**: $3.46 - $8.78
- **Dairy**: $8.33 - $13.97
- **Bakery**: $6.26 - $11.88
- **Meat**: $11.48 - $16.98

---

### Task 4: Analisi Prodotti Strategici
Identificazione di prodotti Meat e Dairy con vendite superiori a 10 unità mensili, per analisi approfondita.

**Query:**
```sql
SELECT
    public.products.product_id,
    public.products.price,
    public.products.average_units_sold
FROM public.products
WHERE public.products.product_type IN ('Meat', 'Dairy')
    AND public.products.average_units_sold > 10;
```

**Risultato:** 698 prodotti identificati

---

## Tecnologie Utilizzate

- **SQL (PostgreSQL)**: Query e analisi dati
- **DataCamp notebook (DataLab)**: Ambiente di sviluppo

## Insights Principali

1. **Qualità dei Dati**: Il 10% dei prodotti del 2022 aveva dati mancanti sull'anno di introduzione
2. **Distribuzione Prezzi**: Meat presenta i prezzi più alti ($11.48-$16.98), mentre Produce i più bassi ($3.46-$8.78)
3. **Prodotti Strategici**: 698 prodotti di Meat e Dairy mostrano vendite consistenti (>10 unità/mese)
4. **Copertura Categorie**: Tutte le 5 categorie merceologiche presentano un range di prezzi adeguato per servire diverse fasce di clientela

## Raccomandazioni

- Monitorare la qualità dei dati inseriti nel sistema per evitare problemi simili al bug del 2022
- Considerare l'espansione dell'offerta nella categoria Produce (range più ristretto)
- Focalizzare le strategie di marketing sui 698 prodotti ad alto volume in Meat e Dairy
