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

## Next Steps

- Remove keywords when searching on Vinted
- Apply reranking using visual similarity (`FashionCLIP` + Vector DB)