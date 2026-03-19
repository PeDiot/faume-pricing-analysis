# Faume Pricing Analysis

MVP to analyze the pricing position of second-hand [Sœur](https://secondhand.soeur.fr/products?page=1) items by comparing them with similar listings published on Vinted.

## Install

```bash
git clone https://github.com/PeDiot/faume-pricing-analysis.git
cd faume-pricing-analysis
uv sync
```

## Data

Create a local `data/` directory:

```bash
mkdir data
```

Download the following files into `data/`:

* [`items.json`](https://storage.googleapis.com/faume/soeur/items.json) — raw items extracted from the Faume API
* [`dataset.json`](https://storage.googleapis.com/faume/soeur/dataset.json) — Faume items enriched with Vinted comparables
* [`vinted_comparables.jsonl`](https://storage.googleapis.com/faume/soeur/vinted_comparables.jsonl) — preprocessed comparable-level dataset ready for BigQuery upload

## Workflow

### 1. Load Faume items

Load raw items from the Faume API export stored in `items.json`.

### 2. Map Faume items to Vinted catalogs

Map each Faume item to one or more Vinted catalog IDs based on the Faume item type, using [`catalog_mapping.json`](mapping/catalog_mapping.json).

### 3. Retrieve Vinted comparables

Search the Vinted database using `VintedClient`, combining:

* Vinted catalog IDs
* the item model provided by Faume as keyword input

This step produces a comparable set of Vinted listings for each Faume item.

### 4. Postprocess data for analytics

Postprocess Faume items and Vinted comparables into a flat comparable-level dataset formatted for BigQuery ingestion.

Output: [`vinted_comparables.jsonl`](https://storage.googleapis.com/faume/soeur/vinted_comparables.jsonl)

## Output

The final output is designed to feed a BigQuery table and power a [Looker Studio dashboard](https://lookerstudio.google.com/reporting/a768b056-e43f-49ba-8f5a-dcfff4e7f89b) with:

* portfolio-level pricing analysis
* item-level exploration
* category-level breakdowns

## Metrics

### Absolute Price Difference

```
price_diff_abs = price_faume - price_vinted_median
```

where `price_vinted_median` is the median price for comparable Vinted items.

### Relative Price Difference

```
price_diff_rel = price_faume - price_vinted_median / price_vinted_median
```

#### Interpretation

* A **positive** value means the Faume item is priced **above** the Vinted median benchmark
* A **negative** value means the Faume item is priced **below** the Vinted median benchmark
* A value close to **0** means the Faume item is priced **close to market**

### Confidence score

The confidence score is a heuristic indicator designed to reflect how reliable the Vinted comparison is for a given Faume item.

```
confidence_score = 0.6 × n_score + 0.4 × dispersion_score
````

#### `n_score`

`n_score` depends on the number of Vinted comparables found for the item (`n`).

Scoring rules:

* `n` < 3 → `n_score` = 0
* 3 <= `n` < 5 → `n_score` = 0.4
* 5 <= `n` < 10 → `n_score` = 0.7
* 10 <= `n` < 20 → `n_score` = 0.9
* `n` >= 20 → `n_score` = 1

#### `dispersion_score`

`dispersion_score` depends on how tightly Vinted comparable prices are clustered around the median (`dispersion`).

```
dispersion = (q3 - q1) / q2
```

where:

* `q1` = first quartile of Vinted prices
* `q2` = median of Vinted prices
* `q3` = third quartile of Vinted prices

Scoring rules:

* `dispersion` <= 0.20 → `dispersion_score`= 1
* 0.20 < `dispersion` <= 0.35 → `dispersion_score` = 0.8
* 0.35 < `dispersion` <= 0.50 → `dispersion_score` = 0.6
* 0.50 < `dispersion` <= 0.75 → `dispersion_score` = 0.35
* `dispersion` > 0.75 → `dispersion_score` = 0.1

#### Interpretation

The confidence score increases when:

* more Vinted comparables are available
* Vinted prices are more tightly clustered

This score is heuristic and meant to provide a simple, interpretable proxy for comparison quality.

## Next Steps

- Remove keywords when searching on Vinted
- Apply reranking using visual similarity (`FashionCLIP` + Vector DB)