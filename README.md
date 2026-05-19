# Bank Customer Churn Analysis — Power BI

A Power BI dashboard analysing customer attrition across **10,000 banking customers** in France, Germany, and Spain. Built on the [Bank Customer Churn Prediction](https://www.kaggle.com/datasets/gauravtopre/bank-customer-churn-dataset) dataset, the report surfaces the key demographic, behavioural, and product drivers of churn — and translates them into actionable retention strategies.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Key Findings](#key-findings)
3. [Dashboard Pages](#dashboard-pages)
4. [Data Model](#data-model)
5. [DAX Measures](#dax-measures)
6. [Power Query Transformations](#power-query-transformations)
7. [Relationships](#relationships)
8. [Strategic Recommendations](#strategic-recommendations)
9. [File Structure](#file-structure)

## Project Overview

| Attribute | Detail |
|---|---|
| **Tool** | Power BI Desktop |
| **Dataset** | Bank Customer Churn Prediction (CSV) |
| **Scope** | France, Germany, Spain |
| **Customers** | 10,000 |
| **Overall Churn Rate** | **20.4%** — roughly 1 in 5 customers |
| **Customers Lost** | 2,037 |
| **Customers Retained** | 7,963 |

The dashboard enables executives to quantify churn risk across demographics, products, and regions — and align marketing and product strategy with data-driven insights rather than assumptions.

## Key Findings

### 1. Geography
| Country | Churn Rate |
|---|---|
| Germany | **32.4%** |
| Spain | 16.7% |
| France | 16.2% |

Germany's churn rate is nearly **double** that of France and Spain, pointing to regional gaps in product relevance, service quality, or competitive positioning.

### 2. Credit Score
Customers with a credit score **below 400** have a **100% churn rate**. Above 400, the rate stabilises around 19–21% across all higher bands. Low creditworthiness strongly correlates with attrition — likely linked to financial stress or unmet product needs.

### 3. Product Holding
| Product | Churn Rate |
|---|---|
| Product 4 | **82.7%** |
| Product 3 | 27.7% |
| Product 1 | ~8% |
| Product 2 | ~7.6% |

Products 1 and 2 dominate the portfolio (~96% of customers) and exhibit healthy retention. Product 4 requires urgent review — its churn rate suggests it either fails to meet expectations or attracts short-term users.

### 4. Activity Status
| Status | Churn Rate |
|---|---|
| Inactive | **26.9%** |
| Active | 14.3% |

Inactive members are nearly **2× more likely** to churn. Engagement level is one of the strongest controllable levers for retention.

### 5. Gender
| Gender | Churn Rate |
|---|---|
| Female | **25.1%** |
| Male | 16.5% |

Female customers are **52% more likely** to churn relative to males — suggesting possible differences in product relevance or service experience.

### 6. Age
Churn rises steeply from age 30, peaks at **71.4% at age 56**, then declines after 60. Mid-career customers (30–60) are the highest-risk cohort — likely seeking better rates, digital convenience, or wealth advisory services.

### 7. Customer Distribution Snapshot
| Metric | Value |
|---|---|
| Total customers | 10,000 |
| Female customers | 54.6% |
| Inactive members | 51.5% |
| Credit card holders | 70% |
| Holding Product 1 or 2 | ~96% |

Inactivity at 51.5% and heavy product concentration in P1 & P2 signal limited cross-sell diversity and a significant engagement risk.

## Dashboard Pages

The `.pbix` file contains the following report pages:

**Dashboard** — Headline KPIs with churn rate, customers lost, and retained figures. Entry point for executives.

**Churn Analysis** — Deep-dive visuals: churn by country, gender, credit score band, product holding, activity status, age band, and account balance. Slicers for interactive filtering across all dimensions.

## Data Model

The model is built on **5 tables**:

| Table | Role |
|---|---|
| `Customer Data` | Core fact table — one row per customer |
| `Age Groups` | Dimension for age band sort order |
| `Acc Bal Groups` | Dimension for account balance band sort order |
| `Credit Score Groups` | Dimension for credit score band sort order |
| `Country Groups` | Dimension for country lookup |

The four dimension tables exist purely for **sort-order control** on categorical chart axes. Without them, Power BI would sort bands alphabetically (e.g. `>71` before `<20`).

## DAX Measures

All four measures live in the `Customer Data` table.

### No of Customers

```dax
No of Customers = COUNT('Customer Data'[Customer ID])
```

Total count of customers in the current filter context. Used as the denominator in churn rate calculations and as a headline KPI card.

### Customers Lost

```dax
Customers Lost =
CALCULATE(
    COUNT('Customer Data'[Churn]),
    'Customer Data'[Churn] = "Churned"
)
```

Counts only rows where `Churn = "Churned"`. `CALCULATE` modifies the filter context to isolate churned customers from the full dataset.

### Churn Rate

```dax
Churn Rate =
'Customer Data'[Customers Lost] / 'Customer Data'[No of Customers]
```

The primary KPI — churned customers divided by total customers. Format as **Percentage** in report settings. This measure is the most cross-referenced metric in the report; slicer changes to any dimension (Country, Age Group, Gender, etc.) automatically flow through `Customers Lost` and `No of Customers`.

### Country List

```dax
Country List =
CALCULATE(
    CONCATENATEX(
        VALUES('Customer Data'[Country]),
        'Customer Data'[Country],
        ","
    )
)

Returns a comma-separated string of distinct countries in the current filter context.

- `VALUES` — extracts a unique list of countries
- `CONCATENATEX` — iterates over that list and joins values with a `,` delimiter
- `CALCULATE` — ensures evaluation happens in a clean filter context

Used in tooltip and text card visuals to display which countries are currently in scope.

> **Note:** All DAX measures use implicit filter inheritance from report slicers. No explicit `ALL()` or `REMOVEFILTERS()` overrides are needed since the measures are designed to reflect the user's current selection.

## Power Query Transformations

All bucketing and label encoding is handled in **Power Query (M)**, not in DAX.

### Customer Data (Main Table)

Source: `Bank Customer Churn Prediction.csv`

| Step | Transformation |
|---|---|
| Promoted Headers | First row used as column names |
| Changed Type | IDs and scores cast to `Int64`, balance to `Currency` |
| Removed Columns | `estimated_salary` dropped |
| Renamed Columns | Snake_case → readable names (e.g. `credit_score` → `Credit Score`) |
| Products | `products_number` merged into label: `"Product 1"`, `"Product 2"`, etc. |
| Credit Card Status | `1` → `"Owned"`, `0` → `"Not Owned"` |
| Activity Status | `1` → `"Active"`, `0` → `"Inactive"` |
| Churn | `1` → `"Churned"`, `0` → `"Not Churned"` |
| Age Groups | Conditional column: `≤20` → `"<20"`, `≤30` → `"21-30"`, ..., `≥71` → `">71"` |
| Credit Scores | Conditional column: `<400` → `"<=400"`, `≤500` → `"401-500"`, ..., `≥801` → `">=800"` |
| Acc Balance | Conditional column: `0` → `0`, `≤1k` → `"1-1k"`, ..., `>200k` → `">200k"` |
| Min Churn Rate | Static column — value `0` (gauge visual minimum) |
| Max Churn Rate | Static column — value `1` (gauge visual maximum) |
| Target Churn Rate | Static column — value `0.15` (gauge target / benchmark line) |

### Dimension Tables

Each dimension is derived from `Customer Data` with duplicates removed and a sort ID column added.

| Table | Sort Column | Range |
|---|---|---|
| `Age Groups` | `Age Group ID` | 1 (`<20`) → 7 (`>71`) |
| `Acc Bal Groups` | `Acc Bal ID` | 1 (`0`) → 5 (`>200k`) |
| `Credit Score Groups` | `Credit Score ID` | 1 (`<=400`) → 6 (`>=800`) |
| `Country Groups` | `Country ID` | Auto-incremented index |

## Relationships

All relationships are **Many-to-One** from `Customer Data` to the respective dimension table, with **Single** cross-filter direction.

| From (`Customer Data`) | To (Dimension Table) | Join Column |
|---|---|---|
| `Age Groups` | `Age Groups` | `Age Groups` |
| `Acc Balance` | `Acc Bal Groups` | `Acc Balance` |
| `Credit Scores` | `Credit Score Groups` | `Credit Scores` |
| `Country` | `Country Groups` | `Country` |

## Strategic Recommendations

| Priority | Area | Action |
|---|---|---|
| Urgent | **Germany** | Launch localised retention campaigns. Investigate service gaps vs. France and Spain. |
| Urgent | **Product 4** | Urgently reassess value proposition, pricing, and customer experience to arrest 82.7% churn. |
| High | **Credit Score < 500** | Offer financial coaching, flexible credit products, and budgeting tools. |
| High | **Inactive Members** | Implement loyalty programmes and digital nudges for 4,849+ inactive customers. |
| High | **Age 30–50** | Target mid-career customers with competitive rates, digital convenience, and wealth advisory. |
| Medium | **Female Customers** | Explore dedicated relationship managers and gender-relevant product bundles. |

> A **5% reduction in churn** ≈ 500 customers retained — a material improvement in deposits, fee revenue, and NPS.

## File Structure

```
├── Customer_Churn_Analysis.pbix        # Power BI report file
├── Bank_Customer_Churn_Prediction.csv  # Source dataset
└── README.md                           # This file
```

---

## Getting Started

1. Download and open `Customer_Churn_Analysis.pbix` in [Power BI Desktop](https://powerbi.microsoft.com/desktop/)
2. If prompted, update the data source path to point to `Bank_Customer_Churn_Prediction.csv` on your machine
3. Click **Refresh** to reload the data
4. Use the slicers on the **Churn Analysis** page to filter by Country, Gender, Activity Status, and more
