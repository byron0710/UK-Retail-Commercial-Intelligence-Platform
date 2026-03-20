

https://github.com/user-attachments/assets/01bda85f-fcda-4bdd-9cab-b90810efa20b

[UK Retail Commercial Intelligence Platform 2026-03-20 11_15.pdf](https://github.com/user-attachments/files/26139606/UK.Retail.Commercial.Intelligence.Platform.2026-03-20.11_15.pdf)
[UK Retail Commercial Intelligence Platform 2026-03-20 11_17.pdf](https://github.com/user-attachments/files/26139603/UK.Retail.Commercial.Intelligence.Platform.2026-03-20.11_17.pdf)
[UK Retail Commercial Intelligence Platform 2026-03-20 11_18.pdf](https://github.com/user-attachments/files/26139602/UK.Retail.Commercial.Intelligence.Platform.2026-03-20.11_18.pdf)
[UK Retail Commercial Intelligence Platform 2026-03-20 11_19.pdf](https://github.com/user-attachments/files/26139600/UK.Retail.Commercial.Intelligence.Platform.2026-03-20.11_19.pdf)
[UK Retail Commercial Intelligence Platform 2026-03-20 11_21.pdf](https://github.com/user-attachments/files/26139598/UK.Retail.Commercial.Intelligence.Platform.2026-03-20.11_21.pdf)
# 🛒 UK Retail Commercial Intelligence Platform

> **End-to-end commercial analytics platform built on Databricks** — modelling contribution margin, promotional ROI, time-series forecasting, price elasticity, and auto-generated recommendations for a UK nursery retail brand.

**Built by:** Byron Wanjau | Data Analyst | Nairobi, Kenya
**Platform:** Databricks (Free Edition) | Unity Catalog | Serverless Compute
**Live Dashboard:** (https://dbc-8ed91520-e60a.cloud.databricks.com/dashboardsv3/01f122cf1b8916c7b671632ec602c85b/published?o=7474656367840140)

---

## 📊 Commercial Headlines

| Metric | Value |
|---|---|
| Gross Revenue Analysed | £17,878,978 |
| Net Revenue (post-deductions) | £15,019,197 |
| Contribution Margin | £8,600,404 |
| Avg Contribution Margin % | 45% |
| Margin Leakage Identified | £2,851,781 |
| Unique SKUs | 4,906 |
| Unique Customers | 5,350 |
| HIGH Priority Actions Raised | 34 |

---

## 🎯 Business Context

This project simulates the full analytical mandate of a **Commercial Data Analyst** for a design-led UK nursery brand distributing through 8 major retail partners (Boots, John Lewis, Asda, Amazon UK, Mothercare, Smyths Toys, Very.co.uk) and a growing DTC ecommerce channel.

The brand operates in a margin-sensitive, seasonally driven category where:
- Trade investment is significant and must be justified by ROI
- Retailer funding structures are commercially complex
- Promotional discipline directly impacts profitability
- Seasonal trading materially affects stock and revenue
- Portfolio performance must be actively optimised

The platform was built to answer 6 commercial questions that raw data systems fail to address:

1. **Where is margin leaking?** — Gross to Net to Contribution Margin waterfall by retailer
2. **Which promotions are destroying value?** — Incrementality and ROI per promotion event
3. **What will revenue look like next quarter?** — Prophet forecasting with 3 scenarios
4. **Which categories can absorb a price increase?** — OLS price elasticity regression
5. **Where are we missing distribution?** — UK regional whitespace vs population weight
6. **What should the business do about all of it?** — Auto-generated recommendations engine

---

## 🏗️ Architecture — Medallion Layers on Databricks

```
┌─────────────────────────────────────────────────────────────────┐
│                        BRONZE LAYER                             │
│  Raw landing — 8 source files ingested as-is into Delta tables  │
│  online_retail_II.xlsx │ ONS retail index │ Google Trends UK    │
│  ONS CPI inflation │ Synthetic ERP master │ Retailer funding    │
└─────────────────────┬───────────────────────────────────────────┘
                      │ PySpark cleaning & enrichment
┌─────────────────────▼───────────────────────────────────────────┐
│                        SILVER LAYER                             │
│  Cleaned, typed, enriched — partitioned by year/month           │
│  silver_pos_cleaned │ silver_erp_product_master                  │
│  silver_retailer_funding │ silver_promo_calendar                 │
│  silver_uk_region_lookup                                         │
└─────────────────────┬───────────────────────────────────────────┘
                      │ Commercial modelling & ML
┌─────────────────────▼───────────────────────────────────────────┐
│                         GOLD LAYER                              │
│  18 business-ready KPI tables — AI/BI dashboard ready           │
│  P&L │ Promo ROI │ Forecasts │ Elasticity │ Recommendations      │
└─────────────────────┬───────────────────────────────────────────┘
                      │ DirectQuery
┌─────────────────────▼───────────────────────────────────────────┐
│                   AI/BI DASHBOARD (7 pages)                     │
│  Published │ Global filters │ Stakeholder-ready                  │
└─────────────────────────────────────────────────────────────────┘
```
---

## 📦 Project Modules

### Module 7 — Bronze Ingestion & ERP Foundation
**Notebook:** `Module_07_Bronze_Ingestion.py`

- Ingests all 8 source files into Bronze Delta tables
- Generates synthetic ERP Product Master (40 SKUs, 8 categories)
- Generates synthetic Retailer Funding table (8 retailers with rebate %, settlement discounts, scan allowances)
- Generates synthetic Promotional Calendar (12 events)
- Tags every transaction with UK Region and Channel (Retail vs DTC)

**Output tables:** `bronze_pos_raw`, `bronze_ons_retail_index`, `bronze_trends_baby_timeseries`, `silver_erp_product_master`, `silver_retailer_funding`, `silver_promo_calendar`, `silver_uk_region_lookup`

---

### Module 1 — Silver Cleaning & Commercial P&L
**Notebook:** `Module_01_Silver_Commercial_PL.py`

- Cleans and types 1,067,371 raw POS transactions
- Removes cancellations, negatives, nulls, and duplicates
- Joins ERP product master, retailer funding, and UK region lookup
- Builds full **Gross → Rebates → Settlement → Scan → Net → COGS → Contribution Margin** waterfall
- Produces SKU profitability ranking with HERO / SOLID / WATCH / AT RISK flags
- Produces regional performance table with RAG status

**Key findings:**
- £2.85M margin leakage identified across portfolio
- 45% average contribution margin
- DTC margin: 26.8% vs Retail margin: 63.3%

**Output tables:** `silver_pos_cleaned`, `gold_retailer_pl`, `gold_sku_profitability`, `gold_regional_performance`, `gold_margin_waterfall`

---

### Module 2 — Trade Spend & Promotional ROI
**Notebook:** `Module_02_Trade_Spend_Promo_ROI.py`

- Tags transactions as promotional or baseline using promo calendar
- Calculates **rolling 4-week pre-promotion baseline** per retailer
- Calculates **promotional lift** (actual minus baseline volume)
- Calculates **incremental margin** from the lift
- Deducts trade spend cost to calculate **true Promotional ROI**
- Ranks all 12 promotions — flags STRONG POSITIVE / POSITIVE / VALUE DESTROYING
- Builds trade spend efficiency table: spend per incremental unit by retailer

**Output tables:** `gold_promo_events`, `gold_trade_spend_efficiency`, `gold_promo_roi_ranking`

---

### Module 3 — Forecasting, Sell-Through & Seasonal Risk
**Notebook:** `Module_03_Forecasting_SellThrough.py`

- Trains **Facebook Prophet** time-series model per channel
- Generates 3 scenarios: Base, Upside (+15%), Downside (-20%)
- Logs all models and metrics (MAPE, bias) to **MLflow**
- Calculates **sell-through rate** per SKU per week vs 12-week rolling max
- Flags FAST MOVER / SLOW MOVER / AT RISK SKUs
- Builds **seasonal risk index** by category and quarter

**Output tables:** `gold_forecast_scenarios`, `gold_forecast_accuracy`, `gold_sell_through`, `gold_seasonal_risk`

---

### Module 4 — Pricing Elasticity & Market Intelligence
**Notebook:** `Module_04_Pricing_Elasticity.py`

- Calculates **price elasticity of demand** using OLS regression on log(price) vs log(units)
- Classifies categories: HIGHLY ELASTIC / ELASTIC / MILDLY ELASTIC / INELASTIC
- Builds competitor pricing index vs 5 simulated UK nursery competitors
- Identifies HIGH / MODERATE / LOW competitive risk by category
- Calculates UK **regional whitespace**: revenue share vs population weight gap
- London identified as MAJOR WHITESPACE (0% revenue vs 13.4% population weight)

**Output tables:** `gold_price_elasticity`, `gold_competitor_pricing`, `gold_whitespace`, `gold_pricing_recommendations`

---

### Module 6 — Actionable Recommendations Engine
**Notebook:** `Module_06_Recommendations_Engine.py`

Applies **10 business rules** across all Gold tables to auto-generate commercial recommendations:

| Rule | What It Flags |
|---|---|
| Rule 1 | Retailer margin below 30% threshold |
| Rule 2 | AT RISK SKUs in the portfolio |
| Rule 3 | Value-destroying promotions (negative ROI) |
| Rule 4 | Inefficient trade spend (>£5 per incremental unit) |
| Rule 5 | Poor forecast accuracy (MAPE >30%) |
| Rule 6 | Slow-moving SKUs (4+ consecutive weeks below 40% sell-through) |
| Rule 7 | Recurring seasonal troughs (2+ periods below index 0.7) |
| Rule 8 | Price increase opportunities (inelastic categories) |
| Rule 9 | Major regional whitespace (>5% gap vs population weight) |
| Rule 10 | Competitive pricing risk (HIGH RISK flag) |

**Result:** 34 HIGH priority + 6 MEDIUM priority recommendations generated

**Output tables:** `gold_recommendations`, `gold_weekly_briefing`

---

### Module 5 — AI/BI Dashboard
**Platform:** Databricks AI/BI Dashboards (7 pages, published, global filters)

| Page | Content |
|---|---|
| Commercial Overview | KPI cards, revenue trend, margin waterfall, retailer table |
| Retailer Profitability | Retailer P&L, margin bridge, SKU portfolio health |
| DTC vs Retail | Channel margin comparison (26.8% vs 63.3%), monthly trend |
| Promo Performance | ROI ranking, actual vs baseline, trade spend efficiency |
| Forecast & Risk | Scenario comparison, sell-through health, seasonal risk |
| Pricing & Market Intel | Elasticity matrix, competitor price gap, whitespace map |
| Recommendations | Action register — 34 HIGH priority items with evidence |

**[View Live Dashboard →](https://dbc-8ed91520-e60a.cloud.databricks.com/dashboardsv3/01f122cf1b8916c7b671632ec602c85b/published?o=7474656367840140)**

---

## 🛠️ Tech Stack

| Tool | Usage |
|---|---|
| **PySpark** | All data ingestion, transformation, Bronze→Silver→Gold |
| **Delta Lake** | Medallion architecture storage, ACID transactions |
| **Python** (pandas, scipy, statsmodels) | Elasticity modelling, synthetic data generation |
| **Facebook Prophet** | Time-series forecasting per channel |
| **scikit-learn** | OLS regression for price elasticity |
| **MLflow** | Experiment tracking, model versioning, forecast registry |
| **Databricks SQL** | Gold layer KPI aggregations, recommendations rules engine |
| **Databricks AI/BI** | 7-page executive dashboard with global filters |
| **Unity Catalog** | Data governance and table management |

---

## 📁 Repository Structure

```
uk-retail-commercial-intelligence-platform/
│
├── notebooks/
│   ├── Module_07_Bronze_Ingestion.py
│   ├── Module_01_Silver_Commercial_PL.py
│   ├── Module_02_Trade_Spend_Promo_ROI.py
│   ├── Module_03_Forecasting_SellThrough.py
│   ├── Module_04_Pricing_Elasticity.py
│   └── Module_06_Recommendations_Engine.py
│
├── docs/
│   ├── Project_Blueprint.docx
│   ├── Project_Conclusion_and_Solution.docx
│   └── Analytical_Thinking_Framework_Audit.docx
│
├── data/
│   └── README_data_sources.md
│
└── README.md
```

---

## 📥 Data Sources

| Dataset | Source | Type |
|---|---|---|
| Online Retail II (UK transactions 2009–2011) | [UCI ML Repository](https://archive.ics.uci.edu/dataset/502/online+retail+ii) | Real / Public |
| ONS Retail Sales Index | [ONS](https://www.ons.gov.uk/businessindustryandtrade/retailindustry) | Real / Public |
| ONS CPI Inflation | [ONS](https://www.ons.gov.uk/economy/inflationandpriceindices) | Real / Public |
| Google Trends (baby products UK) | [Google Trends](https://trends.google.com) | Real / Public |
| ERP Product Master | Python-generated (synthetic) | Synthetic |
| Retailer Funding Structures | Python-generated (synthetic) | Synthetic |
| Competitor Pricing | Python-generated (synthetic) | Synthetic |

---

## ⚠️ Data Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| UCI data is 2009–2011 historical snapshot | May not reflect current UK retail dynamics | Used for structural pattern analysis only |
| Synthetic ERP uses category average costs | SKU-level margins are estimates not actuals | Real project would use actual ERP cost data |
| Retailer assignment by Customer ID range | Simulated account segmentation not real | Real project would have retailer dimension table |
| Competitor pricing is synthetic | Directional only — not real market data | Real project would use Nielsen or Kantar data |
| Google Trends is a search proxy for demand | Correlation not causation with actual sales | Used as supplementary market context only |

---

## 🔒 Data Privacy & GDPR

- No personally identifiable information (PII) was used in this project
- Customer IDs from the UCI dataset are anonymous numeric identifiers only
- Customer IDs were used exclusively for aggregation — no individual customer profiling
- No names, addresses, emails, or contact details exist in any dataset
- All synthetic data was generated programmatically with no real individual data

---


This project was audited against a professional analytical thinking framework covering 23 questions across 4 areas:

| Area | Questions | YES | PARTIAL | GAP |
|---|---|---|---|---|
| The Ask — Purpose & Context | 6 | 6 | 0 | 0 |
| Prepare & Process — Data Quality | 7 | 6 | 1 | 0 |
| Analyse & Share — Audience & Ethics | 7 | 6 | 1 | 0 |
| Self-Review Checklist | 4 | 3 | 1 | 0 |
| **Total** | **23** | **21** | **2** | **0** |

Full audit document available in `docs/Analytical_Thinking_Framework_Audit.docx`

---

## 👤 About the Author

**Byron Wanjau**
Data Analyst | Nairobi, Kenya
Currently working at Kenya Commerce Exchange Service Bureau Ltd.
Open to UK and global data analytics opportunities

- LinkedIn: www.linkedin.com/in/byron-wanjau
- Email: byronwanjau28@gmail.com

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

*Built with Databricks Free Edition · Inspired by [Data With Baraa](https://www.youtube.com/@DataWithBaraa)*
