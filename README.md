# Unpacking Affordability & Demand — NYC Housing (Capstone)

> **Why New York?** New York City is a compact, globally integrated, and locally constrained housing market. High incomes and wage floors create demand capacity while strict land-use, zoning, and supply frictions make prices highly sensitive to small inventory changes. This repository contains the cleaned data, notebooks, charts, and reproducible analysis that explain *what drives price, what affects affordability, and where opportunity/risks lie* across NYC neighborhoods.

---

## Table of contents

- [Project Overview](#project-overview)  
- [Key research questions](#key-research-questions)  
- [Data sources (provenance)](#data-sources-provenance)  
- [What I produced (deliverables)](#what-i-produced-deliverables)  
- [Data cleaning & methodology (summary)](#data-cleaning--methodology-summary)  
- [Data dictionary (summary)](#data-dictionary-summary)  
- [Main analyses & visualizations](#main-analyses--visualizations)  
- [Key findings & interpretation (deep insights)](#key-findings--interpretation-deep-insights)  
- [Actionable recommendations](#actionable-recommendations)  
- [How to reproduce / run notebooks](#how-to-reproduce--run-notebooks)  

---

## Project overview

This project performs a data-driven analysis of the New York housing market using cleaned listing-level transactions and Redfin yearly market aggregates. The goal is to identify the dominant drivers of price and affordability, segment neighborhoods into interpretable market types (stable luxury, volatile opportunity, mass-market), and produce clear recommendations for renters, buyers, investors and policymakers.

**Primary data inputs**
- Transaction-level listing data (prices, sqft, beds, baths, addresses, broker, coordinates)
- Yearly market aggregates (median list/sale price, inventory, months-of-supply)
- Long-run income & minimum wage series (US and NY)

**High-level outcome**
- A set of visual and statistical analyses that explain price variation across space and time, identify leading market indicators (inventory, months_of_supply), and quantify the role of usable space (bedrooms & bathrooms) vs location.

---

## Project website
URL: https://nyc-housing-market-analysis.lovable.app

---

## Key research questions

1. Which property attributes (sqft, beds, baths, property type) most strongly predict sale price?  
2. How have median sale & list prices and inventory changed over 2012–2025, and what does that imply about supply/demand cycles?  
3. How does value density (price per sqft) vary spatially across NYC neighborhoods?  
4. Can neighborhoods be clustered into meaningful market types with different investment/policy implications?  
5. How do supply-side signals (inventory, months_of_supply, new_listings) relate to price dynamics, and do they lead price changes?  
6. What actionable recommendations follow for renters, buyers, investors, and policymakers?

---

## Data sources (provenance)

All data used in this project were downloaded and stored locally in the repo (or linked to via raw URLs). Key sources:

- **NY Houses Sold Data (listing-level)** — Kaggle: *New York Housing Market* (cleaned listings).  
  URL: https://www.kaggle.com/datasets/nelgiriyewithana/new-york-housing-market

- **NY Annual Housing Market Data (aggregated)** — Redfin Data Center (yearly & monthly market metrics).  
  URL: https://www.redfin.com/news/data-center/  
  *Notes:* Redfin provides aggregated metrics such as median list/sale price, inventory, homes sold, and months-of-supply.

- **Median income series (US & Northeast)** — St. Louis FRED:  
  - Northeast median household income: `MEFAINUSNEA672N`.  
    https://fred.stlouisfed.org/series/MEFAINUSNEA672N  
  - US median household income: `MEFAINUSA672N`.  
    https://fred.stlouisfed.org/series/MEFAINUSA672N

- **Minimum wage (US & NY)** — FRED series:  
  - US minimum wage: `STTMINWGFG`.  
    https://fred.stlouisfed.org/series/STTMINWGFG  
  - NY minimum wage: `STTMINWGNY`.  
    https://fred.stlouisfed.org/series/STTMINWGNY

**Context comparison (Kenya)** — sources used in the site’s context sidebar (non-analytical contextual comparison only):  
- World Bank — Kenya country data / GDP per capita.  
  https://data.worldbank.org/country/kenya  
- Kenya National Bureau of Statistics — 2023/24 Kenya Housing Survey (basic report).  
  https://www.knbs.or.ke/reports/2023-24-kenya-housing-survey-basic-report/  
- Knight Frank Kenya — market notes & Nairobi prime residential trends.  
  https://content.knightfrank.com/research

> All external data sources are credited and logged in the notebook headers with download dates.

---

## What I produced (deliverables)

- Cleaned data artifacts:
  - `NY-House-Dataset.csv` (cleaned listing-level CSV)  
  - `market_aggregated_yearly_cleaned.csv` (Redfin yearly market cleaned export — also derived from `New York Housing Market Data.xlsx`)
- Notebooks:
  - `Cleaning.ipynb` — primary cleaning pipeline and heuristics (price unit fixes, dup removal, outlier handling, exports)  
  - `Cleaning No.2.ipynb` — additional validation and QA checks
- Visual assets (high-resolution and interactive):
  - `Dashboard 2.png` — Income & minimum wage comparison visuals  
  - `Dashboard 1 (3).png`, `Dashboard 1 (2).png` — main analytics dashboards (price vs sqft, bedroom/bathroom tiers, price-per-sqft bubbles, histograms, inventory vs price, list vs sale)  
  - `eda_outputs/` — intermediate PNGs and Plotly HTMLs (histogram, boxplot by neighborhood, correlation heatmap, interactive scatter)
- Presentation files:
  - `Presentation and Analysis.docx`  
  - `NY Annual Market Data Analysis.pptx`  
  - `NY House Sales Analysis Charts.pptx`

---

## Data cleaning & methodology (summary)

Key cleaning steps (fully reproducible in `Cleaning.ipynb`):

1. **Column normalization** — standardized column names to `snake_case`.  
2. **Price parsing & unit correction** — detected price values entered in thousands (e.g., `315` to mean `$315,000`) and applied a conservative scaling heuristic (values below 10,000 scaled when evidence of shorthand exists). All price fields cast to numeric (integers).  
3. **Sqft & numeric coercion** — coerced area, bed and bath counts to numeric; flagged implausible values (0, negatives, extreme outliers).  
4. **Address-based deduplication** — removed duplicates using `address_full`, `price`, `beds`, and `sqft` where appropriate.  
5. **Outlier handling** — for visuals I capped extreme values at the 99th percentile (or used log scales) to prevent a few luxury deals from distorting charts; for modelling, winsorization and log-transforms were used where appropriate.  
6. **Date parsing & bucketing** — normalized period dates in the Redfin file and created `year` keys for joining yearly market context to listing-level aggregates.  
7. **Exports** — final cleaned CSVs and generated visual files saved to `eda_outputs/` and `/mnt/data/`.

**Important reproducibility notes**
- All steps, conservative heuristics, and row counts changed by each transformation are logged inside the notebooks.  
- Spot-checks were performed after the thousands-scaling heuristic to ensure no systematic mis-scaling.

---

## Data dictionary (summary)

Below are the primary fields used from the two main datasets (abridged — full dictionary in the repo / notebooks).

### A — `NY-House-Dataset.csv` (key columns)
- `price` — numeric, sale/list price in USD (cleaned)  
- `price_in_k` — original price reported in thousands (if present) — used for unit-fix heuristics  
- `beds` — integer, bedroom count  
- `baths` — float, bathroom count (important predictor)  
- `sqft` — float, living area square footage  
- `address_full` — string, combined address used for dedupe  
- `sub_locality` — string, neighborhood alias (used for spatial grouping)  
- `broker` — string, listing broker/agent  
- `lat`, `lon` — float, coordinates (where present)

### B — `market_aggregated_yearly_cleaned.csv` (Redfin-derived)
- `period_year` — date / year-of-record  
- `region` — string (e.g., New York)  
- `property_type` — string (Condo/Co-op, Townhouse, etc.)  
- `median_list_price` — numeric  
- `median_sale_price` — numeric  
- `inventory` — integer (active listings)  
- `new_listings` — integer  
- `homes_sold` — integer  
- `months_of_supply` — float — key supply-tightness indicator

(Full, column-level dictionary is included in `/docs/data_dictionary.md` in this repo.)

---

## Main analyses & visualizations

**Primary charts (files in repo / `images/` or `/mnt/data`)**:

1. **Income & Wage Context** — `Dashboard 2.png`  
   - Long-run minimum wage (US vs NY) and median household income (US vs Northeast). Used in the intro to motivate NYC selection.

2. **Market trends & inventory** — `Dashboard 1 (3).png`  
   - Median list vs sale price (2012–2025) and inventory/months_of_supply trend. Used to demonstrate supply-led price dynamics.

3. **Bedroom & Bathroom tiers** — `Dashboard 1 (3).png` (top-left panel)  
   - Median price by bedroom count with bathroom tiers. This panel shows how both beds and baths together create step-wise price tiers.

4. **Price vs Square Footage & Price-per-sqft bubble** — `Dashboard 1 (3).png` (top-right / bubble)  
   - Price vs sqft scatter, borough-level average sqft & price; bubble chart of price_per_sqft by county/neighborhood.

5. **Distribution of sale prices (histogram)** — `Dashboard 1 (2).png` (bottom-right)  
   - Shows long-right tail and modal clusters.

6. **Top 10 / neighborhood snapshots / scatter interactives** — `eda_outputs/scatter_price_vs_sqft.html` & `eda_outputs/top10_by_price.csv`; interactive Plotly scatter for exploratory use.

**Each chart is accompanied by a short interpretive caption and recommended talking points in `Presentation and Analysis.docx`.**

---

## Key findings & interpretation (deep insights)

> *Note:* The observations below are written as interpretive insights based on the cleaned data and charts produced in this repo. The language is deliberately careful — numeric effect sizes are available in the notebooks if you want to cite precise coefficients.

1. **Income & wages justify NYC as a high-demand market, but income growth alone does not drive short-run price spikes.**  
   - Higher median incomes in NYC create demand capacity; however, supply frictions and concentrated demand create the sharp accelerations that cause affordability stress.

2. **The market is two-tiered: a mass-market where most transactions occur, and a luxury tail that strongly influences means.**  
   - Use medians and percentiles for realistic affordability statements — means are biased by infrequent, extremely high-value transactions.

3. **Location (neighborhood / sub_locality) explains more cross-sectional price variance than individual micro-features, but micro-features (beds & baths) determine household-level usability and resale value.**  
   - Price-per-sqft is the clearest compact metric of local scarcity and premium.

4. **Bedrooms + bathrooms are structural determinants of price (beds give functional scale; baths multiply utility).**  
   - Adding a bathroom to a multi-person-targeted unit often produces a larger percent price uplift than an equivalent sqft increase in the same neighborhood.

5. **Inventory and months-of-supply are effective short-run leading indicators.**  
   - Falling inventory predicts near-term price accelerations; rising inventory predicts stabilization or declines.

6. **Neighborhood volatility & mean price-per-sqft together identify different market types useful for investors and policymakers.**  
   - Stable high price-per-sqft areas behave differently than volatile low-base areas; each has distinct policy and investment implications.

---

## Actionable recommendations (short)

- **For renters:** Track neighborhood inventory & months_of_supply. When inventory tightens, act quickly.  
- **For buyers:** Prioritize neighborhoods with stable price-per-sqft and functional living features (beds/baths) over cosmetic upgrades.  
- **For investors:** Target neighborhoods with rising incomes but lower current price-per-sqft (early opportunity), and use months_of_supply as a tactical signal.  
- **For policymakers:** Use medians & percentiles to measure affordability; target supply increases where incomes lag prices.

---

## How to reproduce / run notebooks

### Requirements

Create a virtual environment (recommended) and install requirements:

```bash
# using venv
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
