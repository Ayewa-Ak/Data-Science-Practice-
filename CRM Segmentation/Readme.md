# CRM Customer Segmentation — Italian Food Retail Subsidiary

## Project Overview

This project delivers a data-driven customer base segmentation for the Italian 
subsidiary of a leading international food retailer. The goal was to replace an 
outdated segmentation model with a refreshed, analytically rigorous framework 
that enables targeted communication campaigns across distinct customer groups.

The full pipeline covers data loading, quality checks, exploratory data analysis, 
feature engineering, k-means clustering, segment profiling, and campaign brief 
definition - all documented in a single Jupyter notebook.

---

## Business Objective

The Italian subsidiary wanted to renew its CRM communication programmes by 
identifying which types of customers exist in its loyalty base, how they behave 
differently, and how to communicate with each group effectively.

The expected output was an effective segmentation of the customer base 
highlighting significant dimensions that can be leveraged for targeted 
communication campaigns, along with a convincing performance measure to evaluate 
the segmentation over time.

---

## Data

Four datasets were provided covering approximately 2 years of transactional 
history:

| Dataset | Rows | Description |
|---|---|---|
| RETAIL_PRODUCT.csv | 2,800 | Product master - item ID, category ID, category description |
| RETAIL_REGISTRY.csv | 25,727 | Customer registry - ID, registration date, regional code, origin |
| RETAIL_SALES.csv | 489,967 | Transaction headers - customer ID, date, outlet, payment method, time, total points |
| RETAIL_SALES_DETAIL.csv | 2,047,073 | Receipt lines - customer ID, date, product ID, quantity, list price, special flag |

**Note:** RETAIL_PRODUCT.csv and RETAIL_SALES.csv use semicolon delimiters. 
RETAIL_REGISTRY.csv and RETAIL_SALES_DETAIL.csv use comma delimiters.

---

## Project Structure
CRM_Segmenntation_Akosua_Ayewa.ipynb   # Main analysis notebook

CRM_Performance_Measure_Framework.docx  # Formal performance measure document

CRM_Segmentation_Presentation.pptx      # 8-slide business presentation

README.md

---

## Methodology

### 1. Data Loading
Files are loaded from a directory using pandas, with separator handling applied 
per file. All four dataframes are assigned to named variables for clarity.

### 2. Data Exploration
Before any modelling, the raw data is explored across six dimensions:

- **Univariate distributions** - histograms and summary statistics for total 
points, quantity, list price, payment method, outlet, and day of week. Both raw 
and log-transformed versions are plotted to assess skewness.
- **Temporal patterns** - monthly transaction volume and points over the full 
2-year window, with a COVID reference marker. Day-of-week and hour-of-day 
distributions are also examined.
- **Customer-level aggregation** - the transaction log is collapsed to one row 
per customer with eight candidate metrics: recency, frequency, total points, 
average ticket, outlet HHI (Herfindahl index), average basket value, average 
categories per visit, and average promo share.
- **Correlation & redundancy** - a Spearman correlation matrix identifies 
overlapping metrics. Pairs with |r| > 0.7 are flagged for removal.
- **Outlier profiling** - top 1% spenders, one-time purchasers, and recently 
registered customers are profiled and assessed before exclusion decisions are made.
- **Segmentation signal** - bivariate scatter plots (raw and log-scale) and 
cross-tabulations test whether metrics discriminate between customer types before 
modelling begins.

### 2.7 Data Quality Checks
A formal quality audit was run across five categories:

- **Completeness** - null audit across all key fields. TOTAL_POINTS has 21,868 
nulls; LIST_PRICE has 21,704 nulls. Loyalty card attachment rate: 100%.
- **Consistency** - 467,638 transactions show a >1% mismatch between header 
totals and line item sums, likely due to basket-level discounts. Negative values 
found in TOTAL_POINTS (24,099), QUANTITY (343), and LIST_PRICE (20,385).
- **Linkage integrity** - 1 orphan customer ID (WO01027977) not found in 
registry. 2 orphan product IDs (232395, -12) not found in product master. 
0 duplicate transactions detected.
- **Temporal validity** - significant drop in transaction volume observed around 
March 2020 consistent with COVID-19 lockdowns.
- **Behavioural plausibility** - 26 customers in top 0.1% of spend flagged for 
review. 3 receipt lines with quantity > 500 identified as implausible.

### 3. Data Cleaning
Based on quality findings, the following exclusions were applied:

| Decision | Rationale |
|---|---|
| Exclude customer WO01027977 | Orphan ID - not in registry |
| Exclude product IDs 232395 and -12 | Not in product master |
| Exclude lines with quantity > 500 | Implausible, likely errors |
| Filter to positive TOTAL_POINTS and LIST_PRICE | Negatives are returns |
| Trim to trailing 12 months | Avoids COVID-era distortion |

After cleaning: **299,861 transaction headers** and **1,281,003 receipt lines**.

### 4. Feature Engineering
The customer metric table is prepared for clustering:

- **Log-transformation** - total points, frequency, and average ticket are 
right-skewed and are log-transformed using np.log1p
- **Redundancy removal** - highly correlated metrics are dropped based on the 
Spearman correlation analysis
- **Scaling** - all features are standardised using StandardScaler

Final feature set: `recency`, `outlet_hhi`, `avg_promo_share`, 
`total_points_log`, `frequency_log`, `avg_ticket_log`

### 5. Segmentation Modelling
K-means clustering is applied to the scaled customer feature table of 25,193 
customers. The optimal number of clusters is determined using:

- **Elbow method** - WCSS plotted against k from 2 to 10
- **Silhouette score** - cluster separation quality measured per k

**Result: k = 5 selected as the optimal number of segments.**

### 6. Segment Profiling
Each cluster is profiled using mean metric values, a radar chart, a heatmap, 
box plots, and a revenue share analysis comparing customer count % vs spend %.

---

## Results

Five customer segments were identified:

| Segment | Cluster | Customers | Revenue Share | Key Characteristic |
|---|---|---|---|---|
| Champions | 3 | 35% | 62% | High frequency, high spend, recent |
| High Value Occasionals | 4 | 3% | 4% | Rare visits but highest avg ticket |
| Engaged Mid-Tier | 2 | 17% | 15% | Solid behaviour, upsell potential |
| Light Buyers | 0 | 36% | 17% | Regular visits but low basket size |
| Lapsed / At-Risk | 1 | 9% | 2% | ~6 months inactive, drifting away |

**Critical insight:** Champions represent just 35% of the customer base but 
generate 62% of total revenue. Protecting this group is the single 
highest-priority CRM action.

---

## Campaign Briefs

| Segment | Priority | Objective | Offer Type | Frequency |
|---|---|---|---|---|
| Champions | Highest | Retain | VIP rewards, early access, personalised thank-you | 2x / month |
| High Value Occasionals | High | Frequency | Time-limited, personalised category incentives | 1x / month |
| Engaged Mid-Tier | High | Upsell | Bundle deals, cross-category promos, points multiplier | 2x / month |
| Light Buyers | Medium | Basket building | Category intro offers, multi-buy promotions | 1x / month |
| Lapsed / At-Risk | Medium | Win-back | High-value coupon, win-back sequence over 4 weeks | 2-3x over 4 weeks |

---

## Performance Measure

Three KPIs are defined to track segmentation effectiveness on an ongoing basis:

**KPI 1 - Segment Migration Rate**
Share of customers moving to a higher-value segment each month.
- Formula: Migrants to higher segment ÷ origin segment size × 100
- Target: 5% of Light Buyers upgrade to Engaged Mid-Tier within 3 months

**KPI 2 - Revenue Per Segment**
Total points attributable to each segment tracked monthly.
- Formula: Segment points ÷ total points × 100
- Target: Champions revenue share maintained at or above 60%

**KPI 3 - Campaign Uplift**
Incremental purchase rate of targeted segment vs a 10-15% holdout control group.
- Formula: (Targeted rate − Control rate) ÷ Control rate × 100
- Target: Minimum 10% uplift per campaign across all segments

Measurement cadence: KPI 1 and 2 reviewed monthly. KPI 3 reviewed per campaign. 
Full re-segmentation run quarterly.

---

## Deliverables

- **Jupyter Notebook** - full documented analytical pipeline
- **Performance Measure Framework** - formal Word document covering KPI 
definitions, formulas, targets, measurement cadence, governance, and limitations
- **Business Presentation** - 8-slide PowerPoint covering business challenge, 
methodology, headline results, segment personalities, campaign briefs, 
performance measure, and next steps

---

## Libraries Used

```python
pandas
numpy
matplotlib
seaborn
scipy
sklearn (StandardScaler, KMeans, silhouette_score)
```

---

## Next Steps

- Refresh segmentation monthly as new transaction data arrives
- Track segment migration quarterly to measure campaign effectiveness
- Run A/B uplift analysis per segment after the first 3-month campaign cycle
- Add geographic dimension (REGIONAL_CODE) for outlet-level targeting in 
the next iteration
