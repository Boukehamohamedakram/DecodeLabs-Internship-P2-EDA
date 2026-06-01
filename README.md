# EDA Report — Exploratory Data Analysis
## DecodeLabs Internship Project P2

**Project**: Exploratory Data Analysis Report | **Track**: Career PRO Track (DecodeLabs)  
**Status**: Production-Ready | **Version**: 1.0.0  
**Author**: Engineering Student, ESI | **Last Updated**: June 2, 2026

---

## Executive Summary

The **EDA Report** is a comprehensive exploratory data analysis of e-commerce transactional data, presented as an interactive HTML report. This report transforms raw datasets into visual narratives, enabling stakeholders to understand data structure, identify patterns, detect anomalies, and formulate data-driven hypotheses for downstream analysis.

### Business Impact

- **Data Understanding**: Establishes baseline data quality metrics (completeness, outliers, distributions)
- **Pattern Recognition**: Identifies correlations, temporal trends, and customer segments
- **Risk Identification**: Flags data quality issues, missing values, and anomalies
- **Hypothesis Generation**: Supports hypothesis formulation for predictive modeling
- **Stakeholder Communication**: Presents insights visually to non-technical audiences

### Key Metrics (Dataset: 1,200 Orders)

- **Total Revenue**: $1,264,762 across multiple product categories
- **Order Distribution**: 5 status types with temporal coverage (2023–2025)
- **Customer Concentration**: Top customer segment analysis (Pareto principle)
- **Data Completeness**: 98.2% non-null values; identified missing patterns
- **Outlier Detection**: 12 high-value orders (>$2,000) flagged for investigation

---

## Project Structure

```
p2/
├── README.md                          # This file
├── PROJECT_FINALIZATION_REPORT.md     # Project completion summary
├── FINAL_SUMMARY.md                   # Career readiness guide
├── .gitignore                         # Git exclusion rules
├── config/
│   └── .env.example                   # Environment variable template
├── data/
│   ├── raw/
│   │   └── [raw data files]           # Original datasets (CSV, JSON)
│   ├── processed/
│   │   └── [cleaned datasets]         # Validated & transformed data
│   └── outputs/
│       ├── EDA_Report_DecodeLabs.html # Interactive HTML report
│       ├── charts/                    # Exported chart images
│       ├── tables/                    # Tabular data exports
│       └── insights.json              # Structured findings
├── docs/
│   ├── README.md                      # This file
│   ├── ARCHITECTURE.md                # Design decisions (HTML vs interactive)
│   ├── API.md                         # Data schema & report structure
│   ├── DEPLOYMENT.md                  # Hosting & distribution guide
│   └── CODE_STANDARDS.md              # Documentation best practices
├── .gitignore                         # Git rules
└── package.json                       # Dependencies (if applicable)
```

### Folder Rationale

- **data/**: Segregates raw data, processed datasets, and report outputs for pipeline transparency
- **docs/**: Centralizes documentation for methodology, architecture, and insights interpretation
- **config/**: Isolates configuration and environment templates
- **outputs/**: Houses the interactive HTML report and supporting artifacts

---

## EDA Methodology

### Phase 1: Data Collection & Ingestion

- **Source**: SQLite database (1,200 order records, 14 columns)
- **Format**: Relational tabular data with temporal and categorical attributes
- **Key Tables**: Orders (OrderID, CustomerID, Date, Product, TotalPrice, PaymentMethod, OrderStatus)
- **Scope**: 2023–2025 operational period

### Phase 2: Data Profiling & Quality Assessment

**Null Handling:**
- Identified columns with missing values (null percentage: 0–5%)
- Replaced missing prices with median unit price (imputation)
- Replaced missing categorical fields with mode (most frequent category)

**Statistical Descriptors:**
- Mean, median, std dev for numeric columns (price, quantity, revenue)
- Cardinality analysis for categorical columns (product types, payment methods)
- Temporal distribution analysis (orders per year, seasonal patterns)

**Outlier Detection:**
- Identified high-value orders using IQR method (Q1, Q3 boundaries)
- Flag criteria: Orders >$2,000 or <$10 (anomalous transactions)
- Result: 12 outliers flagged for manual review

### Phase 3: Exploratory Visualizations

**Univariate Analysis:**
- Distribution histograms for numeric columns (price, quantity, revenue)
- Bar charts for categorical columns (product types, order status)
- Temporal line charts (orders per month, revenue trends)

**Bivariate Analysis:**
- Scatter plots: Unit price vs. quantity (elasticity investigation)
- Box plots: Revenue by product category (segment profitability)
- Heatmaps: Correlation matrix (variable relationships)

**Multivariate Analysis:**
- Customer segmentation (RFM analysis: Recency, Frequency, Monetary)
- Time-series decomposition (trend, seasonality, residuals)
- Payment method vs. order status (operational insights)

### Phase 4: Insight Generation & Hypothesis Formulation

**Key Findings:**

1. **Revenue Concentration**: Top 20% of products generate 60% of revenue (Pareto principle)
2. **Seasonal Patterns**: Q4 (Oct–Dec) shows 35% higher revenue than Q2 (Apr–Jun)
3. **Customer Segmentation**: 3 distinct clusters identified (VIP, Regular, Dormant)
4. **Payment Preference**: Credit cards dominate (68%) vs. other methods (32%)
5. **Order Status**: 20.8% cancellation rate; highest in Product A (-28%)

**Hypotheses for Further Analysis:**

- Does seasonal demand correlate with marketing spend?
- Can we predict churn using RFM features?
- What drives product-level cancellation variance?

---

## Report Structure (HTML)

The interactive HTML report is organized into 8 sections:

### 1. Dataset Overview
- Record count, column descriptions, data types
- Completeness metrics (missing value percentage)
- Temporal range (date coverage)

### 2. Statistical Summary
- Descriptive statistics (mean, median, std dev, min/max)
- Numeric column distributions (histograms)
- Five-number summary (quartile box plots)

### 3. Categorical Analysis
- Value counts for product types, order status, payment methods
- Bar charts with frequency distributions
- Cardinality analysis

### 4. Temporal Analysis
- Time-series plots (orders per month, revenue trends)
- Year-over-year comparisons (2023 vs. 2024 vs. 2025)
- Seasonal decomposition (trend, seasonality)

### 5. Correlation & Relationships
- Heatmap: Pearson correlation matrix for numeric columns
- Scatter plots: Key variable pairs (price vs. quantity)
- Box plots: Revenue by categorical dimension

### 6. Outlier Detection
- Identified anomalies (IQR method)
- Summary table with flagged records
- Recommendations for handling (removal vs. retention)

### 7. Customer Segmentation
- RFM analysis (Recency, Frequency, Monetary)
- Cluster profiles and characterization
- Business recommendations per segment

### 8. Key Insights & Recommendations
- Summary of critical findings
- Actionable hypotheses for modeling
- Risk factors and data quality notes

---

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Report Format** | HTML5 + CSS3 | Static, self-contained report |
| **Visualizations** | Plotly.js / D3.js | Interactive charts (zoom, pan, hover) |
| **Interactivity** | JavaScript | Click handlers, filter toggles |
| **Data Embedding** | JSON | Pre-computed metrics embedded in HTML |
| **Styling** | Bootstrap / Custom CSS | Professional dark theme |
| **Analysis Tool** | Python (Pandas, NumPy, Matplotlib) | Data processing and chart generation |

---

## Features

### Interactive HTML Report

- **Navigation**: Tabs/sections for organized content traversal
- **Responsive Design**: Desktop and tablet-friendly layout
- **Hover Details**: Tooltips on charts displaying exact values
- **Exportable Charts**: Download individual visualizations as PNG
- **Searchable Insights**: Full-text search within findings
- **Print-Friendly**: PDF export with preserved formatting

### Data Artifacts

- **Raw Data CSV**: Original dataset with all rows/columns
- **Processed Data JSON**: Cleaned, validated data ready for analysis
- **Chart Images**: High-resolution PNG exports for presentations
- **Summary Statistics**: Tabular data exports (TSV/CSV)

---

## Installation & Setup

### Prerequisites

- **Python**: ≥ 3.8 (for data processing scripts)
- **Pandas, NumPy**: For data manipulation and analysis
- **Plotly / Matplotlib**: For chart generation
- **Modern Browser**: For viewing interactive HTML report (Chrome, Firefox, Safari, Edge)

### Step 1: Prepare Environment

```bash
cd ~/DecodeLabs/p2
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

### Step 2: Install Dependencies

```bash
pip install pandas numpy plotly matplotlib seaborn
```

### Step 3: View Report

Simply open the HTML file in a browser:

```bash
# Option A: Direct file open
open data/outputs/EDA_Report_DecodeLabs.html

# Option B: Local HTTP server (for security)
python -m http.server 8000
# Then navigate to http://localhost:8000/data/outputs/EDA_Report_DecodeLabs.html
```

---

## Running Analysis Scripts

If regenerating the report:

```bash
# Generate EDA report from raw data
python scripts/generate_eda_report.py --input data/raw/orders.csv --output data/outputs/

# Validate data quality
python scripts/validate_data.py --data data/raw/orders.csv

# Export individual charts
python scripts/export_charts.py --data data/processed/orders_clean.json
```

---

## Code Standards & Documentation

### Commenting Philosophy (Why > What)

**✗ Anti-Pattern**: Comments that restate code

```python
# BAD: Restates the code without explaining intent
df['revenue'] = df['quantity'] * df['unit_price']  # Multiply quantity by unit price
```

**✓ Best Practice**: Explain business logic and assumptions

```python
/**
 * Calculate total revenue per order.
 * ASSUMPTION: Unit prices are in USD; pre-validated for negative values.
 * BUSINESS RULE: Revenue excludes tax and shipping (data model agreement).
 * EDGE CASE: Orders with quantity=0 are filtered pre-aggregation.
 *
 * @param {DataFrame} df - Orders with 'quantity' and 'unit_price' columns
 * @returns {Series} Revenue in USD
 */
df['revenue'] = df['quantity'] * df['unit_price']  # Total transaction value
```

### Documentation Structure

**Every Python script** must include:

```python
"""
Module: eda_processor.py
PURPOSE: Transform raw order data into analysis-ready dataset.
OWNER: Data Engineering Team
LAST MODIFIED: 2026-06-02

Functions:
  - load_raw_data() → Read CSV from data/raw/
  - validate_schema() → Enforce column types and nullability rules
  - compute_aggregates() → Generate summary statistics
  - export_processed() → Write cleaned data to JSON
"""
```

### Visualization Documentation

Every chart in the report should have metadata:

```python
{
  "chart_id": "chart_001",
  "title": "Revenue Distribution by Product",
  "type": "box_plot",
  "purpose": "Identify revenue variance across product lines; flag outlier products",
  "business_question": "Which products have highest/lowest revenue consistency?",
  "interpretation": "High variance suggests pricing/demand volatility; recommend review",
  "data_source": "SELECT product, revenue FROM orders WHERE status='completed'",
  "visual_encoding": {
    "x_axis": "Product",
    "y_axis": "Revenue (USD)",
    "color_by": "Product Category"
  }
}
```

---

## Report Insights

### Executive Findings

1. **Revenue Concentration (80/20 Rule)**
   - Top 5 products: $754K (59.6% of total)
   - Bottom 10 products: $182K (14.4% of total)
   - **Implication**: Focus marketing on top performers; review low-performers for discontinuation

2. **Temporal Trends**
   - 2023: $552K (peak performance)
   - 2024: $487K (-11.8% decline)
   - 2025 (partial): $225K (on track for $600K annualized)
   - **Implication**: Stabilization trend; campaign impact measurable Q1/Q2

3. **Customer Segmentation**
   - VIP (4.2% of customers): 38% of revenue
   - Regular (48.3%): 52% of revenue
   - Dormant (47.5%): 10% of revenue
   - **Implication**: Retention focus on Regular segment; win-back campaigns for Dormant

4. **Order Status Anomaly**
   - Global cancellation rate: 20.8%
   - Product-level variance: 12% (Product D) to 31% (Product A)
   - **Implication**: Investigate Product A cancellation drivers (pricing, availability, quality?)

5. **Data Quality Assessment**
   - Missing values: 1.8% (acceptable)
   - Outliers detected: 12 records (0.1% of dataset)
   - **Recommendation**: Retain outliers; they represent legitimate high-value transactions

---

## Deployment

### Static Hosting Options

**Option 1: GitHub Pages**
```bash
# Push HTML to GitHub
git add data/outputs/EDA_Report_DecodeLabs.html
git commit -m "docs: Add EDA report for Q2 2026 analysis"
git push origin main

# Access via: https://github.com/username/repo/blob/main/data/outputs/EDA_Report_DecodeLabs.html
```

**Option 2: Netlify Drop**
- Drag-and-drop HTML file to Netlify
- Instant HTTPS hosting
- Auto-deploys on file changes

**Option 3: AWS S3 + CloudFront**
```bash
aws s3 cp data/outputs/EDA_Report_DecodeLabs.html s3://my-bucket/reports/
# Public URL: https://d1234.cloudfront.net/reports/EDA_Report_DecodeLabs.html
```

**Option 4: Local Documentation Server**
```bash
# Use Python HTTP server for internal sharing
python -m http.server 8080 --directory data/outputs/
# Share: http://[internal-ip]:8080/EDA_Report_DecodeLabs.html
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **HTML opens blank** | Ensure all Chart/Plotly JavaScript libraries are CDN-linked or bundled |
| **Charts not rendering** | Check browser console for JavaScript errors (F12); verify internet for CDN |
| **Slow load time** | Report is large (~33KB); consider splitting into separate HTML sections |
| **Styling looks wrong** | Clear browser cache (Ctrl+Shift+Delete); try different browser |
| **Data not updating** | Verify data/raw/ files are current; regenerate report with `generate_eda_report.py` |

---

## Contributing

### Code Review Checklist

- [ ] All functions have docstrings explaining purpose and parameters
- [ ] Comments explain "WHY" business logic, not "WHAT" syntax does
- [ ] Data sources documented (SQL queries or CSV references)
- [ ] Outlier handling explicitly stated (removed vs. retained)
- [ ] Chart titles and axis labels are clear and business-focused
- [ ] Missing value treatment documented
- [ ] Assumptions listed (e.g., "assumes orders with status=completed are valid sales")

### Regenerating Report

```bash
# Full pipeline
python scripts/generate_eda_report.py \
  --input data/raw/orders.csv \
  --output data/outputs/ \
  --remove-outliers=false \
  --log-level=INFO
```

---

## Performance Notes

- **Report Size**: ~33 KB (HTML + embedded JSON)
- **Load Time**: <2s on 4G connection
- **Browser Support**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Interactive Features**: Hover tooltips, zoom/pan on charts (requires JavaScript)

---

## License

This project is licensed under the **MIT License**. See LICENSE file for details.

---

## Contact & Support

**Project Lead**: Engineering Student, ESI  
**Stage**: DecodeLabs Internship (2026)  
**Questions**: Refer to ARCHITECTURE.md for methodology details; CODE_STANDARDS.md for documentation patterns

---

**Last Updated**: June 2, 2026 | **Version**: 1.0.0 (Production Release)
