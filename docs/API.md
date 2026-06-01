# API.md — Data Schema & Report Structure

## Report Structure

### HTML Organization

```html
<div class="eda-report">
  <header>
    <h1>EDA Report — DecodeLabs</h1>
    <p>Analysis Date, Dataset Info</p>
  </header>
  
  <nav class="tabs">
    <button class="tab" data-section="overview">Overview</button>
    <button class="tab" data-section="statistics">Statistics</button>
    <button class="tab" data-section="distributions">Distributions</button>
    ...
  </nav>
  
  <main>
    <section id="overview"><!-- Dataset summary --></section>
    <section id="statistics"><!-- Descriptive stats --></section>
    <section id="distributions"><!-- Histograms, bar charts --></section>
    ...
  </main>
  
  <script>
    window.eda_data = { /* Embedded JSON */ };
  </script>
</div>
```

---

## Embedded JSON Schema

### Root Structure

```json
{
  "metadata": { /* Analysis metadata */ },
  "dataset_overview": { /* Record count, columns */ },
  "numeric_analysis": { /* Distributions, statistics */ },
  "categorical_analysis": { /* Value counts */ },
  "temporal_analysis": { /* Time-series data */ },
  "correlation": { /* Numeric correlation matrix */ },
  "outliers": { /* Flagged anomalies */ },
  "segments": { /* Customer segmentation */ },
  "insights": { /* Key findings */ }
}
```

### metadata Object

```json
{
  "metadata": {
    "report_date": "2026-06-02T14:30:00Z",
    "dataset_name": "Orders Database",
    "record_count": 1200,
    "total_columns": 14,
    "date_range": {
      "min": "2023-01-01",
      "max": "2025-12-31"
    },
    "analyst": "Data Team",
    "version": "1.0.0"
  }
}
```

### dataset_overview Object

```json
{
  "dataset_overview": {
    "total_records": 1200,
    "columns": [
      {
        "name": "OrderID",
        "type": "integer",
        "non_null_count": 1200,
        "null_percentage": 0.0,
        "description": "Unique order identifier"
      },
      {
        "name": "CustomerID",
        "type": "integer",
        "non_null_count": 1198,
        "null_percentage": 0.167,
        "description": "Customer identifier"
      },
      {
        "name": "TotalPrice",
        "type": "float",
        "non_null_count": 1200,
        "null_percentage": 0.0,
        "description": "Order total in USD",
        "min": 9.99,
        "max": 4299.99,
        "mean": 1054.0
      }
    ],
    "null_summary": {
      "total_nulls": 18,
      "null_percentage": 1.5
    }
  }
}
```

---

## Data Source: Orders Table

### Column Definitions

| Column | Type | Nullable | Description | Example |
|--------|------|----------|-------------|---------|
| OrderID | INT | No | Unique order identifier | 1001 |
| CustomerID | INT | Yes | Customer reference | 5042 |
| Date | DATE | No | Order date | 2024-03-15 |
| Product | VARCHAR | No | Product name | "Laptop" |
| Quantity | INT | No | Units ordered | 2 |
| UnitPrice | DECIMAL | No | Price per unit (USD) | 899.99 |
| TotalPrice | DECIMAL | No | Quantity × UnitPrice (USD) | 1799.98 |
| PaymentMethod | VARCHAR | No | Payment type | "CreditCard" / "PayPal" / "BankTransfer" |
| OrderStatus | VARCHAR | No | Fulfillment status | "Completed" / "Pending" / "Cancelled" |
| ReferralSource | VARCHAR | Yes | Customer acquisition channel | "Google" / "Direct" / "Newsletter" |
| **[7+ custom fields]** | ... | ... | Additional business context | ... |

---

## numeric_analysis Object

### Structure

```json
{
  "numeric_analysis": {
    "TotalPrice": {
      "count": 1200,
      "mean": 1054.0,
      "std": 847.32,
      "min": 9.99,
      "25%": 249.99,
      "50%": 749.99,
      "75%": 1599.99,
      "max": 4299.99,
      "skewness": 1.23,
      "kurtosis": 2.45,
      "distribution_type": "right_skewed",
      "histogram_data": [
        { "bin": "[0, 500)", "count": 312, "percentage": 26.0 },
        { "bin": "[500, 1000)", "count": 287, "percentage": 23.9 },
        ...
      ]
    },
    "Quantity": { /* Similar structure */ }
  }
}
```

### Chart Mapping

```javascript
// For each numeric column, Plotly receives:
{
  x: [histogram_bin_labels],
  y: [histogram_counts],
  type: 'histogram',
  marker: { color: '#4a9edd' }
}
```

---

## categorical_analysis Object

### Structure

```json
{
  "categorical_analysis": {
    "Product": {
      "unique_count": 7,
      "value_counts": [
        { "value": "Laptop", "count": 245, "percentage": 20.4 },
        { "value": "Monitor", "count": 198, "percentage": 16.5 },
        { "value": "Keyboard", "count": 187, "percentage": 15.6 }
      ],
      "chart_type": "bar"
    },
    "PaymentMethod": {
      "unique_count": 4,
      "value_counts": [
        { "value": "CreditCard", "count": 816, "percentage": 68.0 },
        { "value": "PayPal", "count": 256, "percentage": 21.3 }
      ]
    },
    "OrderStatus": {
      "unique_count": 5,
      "value_counts": [
        { "value": "Completed", "count": 952, "percentage": 79.3 },
        { "value": "Cancelled", "count": 250, "percentage": 20.8 }
      ]
    }
  }
}
```

---

## temporal_analysis Object

### Structure

```json
{
  "temporal_analysis": {
    "monthly_orders": [
      { "month": "2023-01", "count": 42, "revenue": 44280 },
      { "month": "2023-02", "count": 38, "revenue": 40652 },
      ...
    ],
    "yearly_summary": [
      { "year": 2023, "orders": 542, "revenue": 552100 },
      { "year": 2024, "orders": 489, "revenue": 487200 },
      { "year": 2025, "orders": 169, "revenue": 225462 }
    ],
    "seasonal_decomposition": {
      "trend": [ /* trend values */ ],
      "seasonal": [ /* seasonal component */ ],
      "residual": [ /* residuals */ ]
    }
  }
}
```

### Chart Mapping for Plotly

```javascript
{
  x: ["2023-01", "2023-02", ...],
  y: [42, 38, ...],
  type: 'scatter',
  mode: 'lines+markers',
  name: 'Orders per Month'
}
```

---

## correlation Object

### Structure

```json
{
  "correlation": {
    "correlation_matrix": [
      [1.0, 0.45, 0.82],          // TotalPrice vs all
      [0.45, 1.0, 0.12],          // Quantity vs all
      [0.82, 0.12, 1.0]           // UnitPrice vs all
    ],
    "column_names": ["TotalPrice", "Quantity", "UnitPrice"],
    "significant_pairs": [
      {
        "var1": "TotalPrice",
        "var2": "Quantity",
        "correlation": 0.45,
        "p_value": 0.001,
        "interpretation": "Moderate positive correlation; higher quantity → higher total price"
      }
    ]
  }
}
```

### Heatmap Encoding

```javascript
{
  z: correlation_matrix,
  x: column_names,
  y: column_names,
  type: 'heatmap',
  colorscale: 'Viridis',
  zmin: -1,
  zmax: 1
}
```

---

## outliers Object

### Structure

```json
{
  "outliers": {
    "detection_method": "IQR (Interquartile Range)",
    "flagged_records": [
      {
        "row_id": 427,
        "OrderID": 5427,
        "TotalPrice": 3999.99,
        "Quantity": 4,
        "reason": "Price exceeds Q3 + 1.5×IQR",
        "severity": "high",
        "recommendation": "Investigate; likely legitimate VIP order"
      },
      {
        "row_id": 892,
        "OrderID": 5892,
        "TotalPrice": 15.99,
        "Quantity": 1,
        "reason": "Price below Q1 - 1.5×IQR",
        "severity": "low",
        "recommendation": "Possible clearance sale; verify pricing"
      }
    ],
    "outlier_summary": {
      "total_flagged": 12,
      "percentage": 1.0,
      "action": "retained"
    }
  }
}
```

---

## segments Object (RFM Analysis)

### Structure

```json
{
  "segments": {
    "method": "RFM Clustering (K-means, k=3)",
    "clusters": [
      {
        "segment_id": 1,
        "name": "VIP",
        "size": 50,
        "percentage": 4.2,
        "characteristics": {
          "avg_recency_days": 12,
          "avg_frequency": 8.4,
          "avg_monetary_value": 9520
        },
        "business_profile": "High-value, frequent customers; strong retention focus",
        "recommendations": ["VIP support", "Exclusive offers", "Loyalty rewards"]
      },
      {
        "segment_id": 2,
        "name": "Regular",
        "size": 580,
        "percentage": 48.3,
        "characteristics": {
          "avg_recency_days": 45,
          "avg_frequency": 2.1,
          "avg_monetary_value": 2180
        }
      },
      {
        "segment_id": 3,
        "name": "Dormant",
        "size": 570,
        "percentage": 47.5,
        "characteristics": {
          "avg_recency_days": 180,
          "avg_frequency": 0.8,
          "avg_monetary_value": 650
        }
      }
    ]
  }
}
```

---

## insights Object

### Structure

```json
{
  "insights": [
    {
      "finding_id": "F001",
      "title": "Revenue Concentration (80/20 Rule)",
      "description": "Top 5 products generate 59.6% of revenue",
      "data_support": {
        "metric": "Revenue Distribution",
        "value": "59.6%",
        "threshold": "> 50%"
      },
      "business_impact": "high",
      "recommendation": "Focus marketing budget on top performers; review low-performers for discontinuation"
    },
    {
      "finding_id": "F002",
      "title": "Elevated Cancellation Rate",
      "description": "Product A has 31% cancellation rate vs. global average 20.8%",
      "severity": "warning",
      "action": "Investigate Product A stock levels, pricing, or quality issues"
    }
  ]
}
```

---

## Example: Rendering a Plotly Chart from JSON

```javascript
// Extract data from embedded JSON
const data = window.eda_data.numeric_analysis.TotalPrice.histogram_data;

// Transform for Plotly
const trace = {
  x: data.map(d => d.bin),
  y: data.map(d => d.count),
  type: 'bar',
  marker: { color: '#4a9edd' }
};

// Render
Plotly.newPlot('chart-container', [trace], {
  title: 'Revenue Distribution',
  xaxis: { title: 'Revenue (USD)' },
  yaxis: { title: 'Order Count' }
});
```

---

## Data Export Formats

### CSV Export (for spreadsheet analysis)

```
OrderID,CustomerID,Date,Product,Quantity,UnitPrice,TotalPrice,PaymentMethod,OrderStatus
1001,5042,2024-03-15,Laptop,2,899.99,1799.98,CreditCard,Completed
1002,5043,2024-03-15,Monitor,1,349.99,349.99,PayPal,Completed
```

### JSON Export (for programmatic analysis)

```json
[
  {
    "OrderID": 1001,
    "CustomerID": 5042,
    "Date": "2024-03-15",
    "Product": "Laptop",
    ...
  }
]
```

---

## API Endpoints (if future backend added)

```
GET /api/eda/report         → Full report JSON
GET /api/eda/statistics     → Numeric statistics only
GET /api/eda/segments       → RFM segments
GET /api/eda/outliers       → Flagged records
POST /api/eda/export        → Export in requested format (CSV/JSON/Excel)
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-06-02 | Initial release; 8 sections, RFM segmentation |
| 0.9.0 | 2026-05-28 | Beta; core visualizations complete |

