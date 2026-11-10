# 🚚 E-Commerce Logistics Audit & Supply Chain Optimization (Olist Dataset)

## 📌 Project Overview

This project presents an end-to-end data engineering and analytics pipeline designed to audit logistics performance, evaluate fulfillment risk, and uncover supply chain bottlenecks for **Olist** (the largest e-commerce marketplace in Brazil). 

By transforming raw transaction records into a **Gold Master Mart**, this audit quantifies delivery over-buffering, seller performance risks, and spatial/category constraints to drive actionable logistics strategies.

---

## 🎯 Business Context & Key Hypotheses

### Business Problem
E-commerce marketplaces face significant margin leaks due to inaccurate delivery estimates, seller delays, and regional distance inefficiencies. Over-promising delivery dates leads to poor customer retention, while overly conservative estimates (over-buffering) degrade conversion rates.

### Core Hypotheses Tested
1. **Delivery Over-Buffering:** Estimated delivery times are systematically inflated compared to actual delivery times, skewing customer expectations.
2. **Seller Performance Bottlenecks:** A minority of underperforming sellers account for the majority of delivery delay incidents.
3. **Geographic & Category Friction:** Order fulfillment times increase disproportionately when orders span long Haversine distances or involve heavy/complex product categories.

---

## 🛠️ Architecture & Pipeline Strategy

The project follows a modular ETL and Data Mart construction architecture:

1. **Source Layer:** Raw transaction data in CSV format (excluded from Git via `.gitignore`).
2. **Bronze / Silver Layer (Notebooks 01–07):** Modular cleaning pipelines responsible for schema standardization, outlier handling, datetime parsing, and outputting lightweight, compressed **Apache Parquet** files.
3. **Gold Layer (Notebook 08):** Merging cleaned layers into a unified analytical data mart with computed spatial metrics (Haversine distance) and logistics KPIs.

---

## 📊 Dataset & Requirements

- **Data Source:** [Kaggle — Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Local Data Setup:** To run the pipeline locally, download the raw dataset from Kaggle and place the CSV files into the `data/` directory.

---

## 📂 Repository Structure

```text
olist-logistics-analytics/
├── data/                       # Local data storage (Git-ignored CSVs + clean Parquet files)
│   ├── .gitkeep
│   ├── customers_clean.parquet
│   ├── geolocation_clean.parquet
│   ├── order_items_clean.parquet
│   ├── orders_clean.parquet
│   ├── products_clean.parquet
│   ├── reviews_clean.parquet
│   └── sellers_clean.parquet
├── notebooks/                  # Modular ETL & Analytical Pipelines
│   ├── 01_orders_cleaning.ipynb
│   ├── 02_customers_cleaning.ipynb
│   ├── 03_order_items_cleaning.ipynb
│   ├── 04_sellers_cleaning.ipynb
│   ├── 05_products_cleaning.ipynb
│   ├── 06_geolocation_cleaning.ipynb
│   ├── 07_order_reviews_cleaning.ipynb
│   └── 08_gold_layer_modeling.ipynb
├── .gitignore                  # Production-grade Git exclusion config
└── README.md                   # Project documentation & execution guide
```

---

## ⚙️ Solution Architecture & Methodology

```text
Raw Olist Relational CSVs (7 tables in /data/)
  ➔ Modular Cleaning Notebooks (01 to 07: Type casting, missing values, outlier detection, Parquet export)
  ➔ Gold Layer Modeling Notebook (08: Relational joins, Guardrails, Haversine Distance)
  ➔ Spatial & Temporal Feature Engineering (Transit & Handling Days, Distance Tiers)
  ➔ Macro Distance Tier Analysis (Hypothesis 1)
  ➔ Micro Vendor Audit & Risk Scatter Profiling (Hypothesis 2)
  ➔ Product Category Vulnerability Assessment (Hypothesis 3)
  ➔ Executive Recommendations (1–2 Week Operational Interventions)
```

---

## 🧹 Data Hygiene & Master Mart Assembly

To ensure high data integrity, the raw multi-table relational dataset underwent a structured cleaning pipeline across dedicated notebooks (`01` through `07`), saving intermediate results into optimized parquet formats inside the `data/` folder:

- **Modular Hygiene & Outlier Treatment:** Handled missing values, standardized string formats, cast strict data types, and filtered out statistical outliers (e.g., erroneous geolocation coordinates outside Brazilian borders, negative or extreme anomalous temporal deltas, and invalid product dimensions/prices).
- **Gold Layer Consolidation (`08_gold_layer_modeling.ipynb`):** Merged all cleaned tables and enforced strict analytical Guardrails:
  - Excluded non-completed orders (filtering strictly for `order_status == 'delivered'`).
  - Removed records with missing essential timestamp checkpoints or invalid coordinates.
- **Spatial Feature Engineering:** Computed direct geographical distances between customer and seller zip codes using the Haversine formula (converted to kilometers) and categorized into logical tiers: Local (<200km), Regional (200-500km), National (500-1000km), and Remote (>1000km).
- **Temporal Feature Engineering:**
  - `seller_handling_days`: Time elapsed from purchase to carrier handoff.
  - `carrier_transit_days`: Time elapsed from carrier dispatch to final customer delivery.
  - `is_underestimated`: Binary delay flag marking orders delivered after the promised estimate date.
  - `forecast_error_days`: Quantitative metric capturing exact variance between promised delivery date and actual delivery date.

---

## 📈 Hypothesis Testing & Analytical Findings

### 📍 Hypothesis 1: Macro Distance Impact & Forecast Over-Buffering
- **Distance Decay:** Delay rates predictably scale with distance — rising from 4.6% for Local shipments to 10.3% for Remote deliveries.
- **Systemic Over-Buffering Uncovered:** Despite distance delays, the forecasting engine exhibits extreme conservatism. Across all distance tiers, the median order arrives 10 to 14 days earlier than the promised estimate date. This artificial buffer inflates expected delivery times on product pages, causing severe checkout conversion loss (our primary Counter-Metric risk).

---

### 🏪 Hypothesis 2: Micro Seller Performance & Financial Risk (`gmv_at_risk`)
Auditing vendors with $\ge 30$ orders and delay rates $>15\%$ revealed that operational risk is concentrated in a tiny fraction of sellers exposing massive platform GMV to churn risk.

#### 💡 Root Cause Analysis: Handling Time vs. Delay Rate
By plotting Average Seller Handling Time against Underestimated / Delayed Order Rate (%) weighted by GMV at Risk (bubble size), sellers naturally clustered into two operational profiles:

1. **Cluster 1: Operational "Slowpokes" (Vendor Bottlenecks):**
   - **Profile:** Sellers taking 7 to 17.5+ days just to pack and hand off goods to the carrier.
   - **Diagnosis:** Internal warehouse inefficiency. High handling times directly cause late customer deliveries.
2. **Cluster 2: Carrier-Victim Fast Dispatches:**
   - **Profile:** Merchants packing goods rapidly (< 3 handling days), yet suffering from 20%–30%+ delay rates.
   - **Diagnosis:** Localized last-mile courier failures in specific merchant regions, NOT merchant incompetence.

---

### 📦 Hypothesis 3: Product Category Vulnerabilities
Filtering for categories with $\ge 100$ items isolated distinct product-level operational constraints:

- **Audio & Electronics (`audio`):** High delay rate (11.83%) driven by carrier transit times (9.97 days) despite rapid seller dispatch (2.1 days), indicating high sensitivity during transit.
- **Seasonal Items (`artigos_de_natal`):** High delay rate (10.07%) tied to peak transit times (12.0 days), reflecting holiday network congestion.
- **Heavy Freight & Furniture (`moveis_escritorio`):** Delay rate (8.05%) driven by extreme warehouse handling times (9.84 days), reflecting packaging and bulk freight assembly bottlenecks.

---

## 📝 Executive Summary & Strategic Recommendations

Our end-to-end audit reveals that delivery failures and churn are driven by specific operational bottlenecks rather than random variance. Addressing these issues directly reduces support complaint costs, financial compensation payouts, and customer attrition.

### Key Findings & Root Causes
1. **Macro-Level (Distance vs. Conversion Loss):** Delay rates scale from 4.6% (Local) to 10.3% (Remote). However, median deliveries arrive 10–14 days ahead of schedule due to severe forecast over-buffering, artificially hurting checkout conversion.
2. **Micro-Level (Vendor vs. Courier Responsibility):** Risk scatter analysis isolates merchant internal packaging delays (7–17+ days) from regional last-mile courier breakdowns.
3. **Category Constraints:** Heavy items (furniture) choke vendor packaging times, while sensitive items (audio) and seasonal goods choke courier transit networks.

### Immediate Actionable Recommendations (1–2 Week Implementation)
- ⚡ **Algorithmic Forecast Recalibration (Quick Win):** Aggressively trim estimated delivery buffers by 5–7 days for Local and Regional distance tiers. This instantly boosts checkout conversion while remaining safely within target SLA error rates.
- 🛡️ **Differentiated Merchant Policy & Search Demotions:**
  - Apply search ranking demotions and operational penalties strictly to **Cluster 1** sellers with high handling times ($> 5$ days).
  - Protect **Cluster 2** vendors (fast dispatches) from unfair penalties, shifting focus toward carrier accountability.
- 🚚 **Regional Carrier Contract Restructuring:** Re-route or renegotiate last-mile courier contracts in high-delay regional corridors to reduce late-delivery compensation costs and support ticket volume.

---

## 🚀 How to Run the Analysis Locally

### 1. Clone the Repository
```bash
git clone https://github.com/Yan-devt/olist-logistics-analytics.git
cd olist-logistics-analytics
```

### 2. Install Dependencies
```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install pandas numpy pyarrow matplotlib seaborn geopy
```

### 3. Add Data Source
Download the raw Olist dataset from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and place the CSV files inside the `data/` folder.

### 4. Execute Pipelines
- Execute cleaning notebooks sequentially from `notebooks/01_orders_cleaning.ipynb` to `notebooks/07_order_reviews_cleaning.ipynb` to build the `.parquet` layers.
- Open and run the master modeling notebook: `notebooks/08_gold_layer_modeling.ipynb`.