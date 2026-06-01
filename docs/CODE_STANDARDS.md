# CODE_STANDARDS.md — Documentation & Best Practices

## Philosophy: "Why" Over "What"

Comments should explain **business logic and assumptions**, not restate what the code obviously does.

### ✗ Anti-Pattern: Restating Code

```python
# BAD: Comment just repeats what code does
df['revenue'] = df['quantity'] * df['unit_price']  # Multiply quantity by unit price
```

### ✓ Best Practice: Explain Intent and Context

```python
"""
Calculate total order revenue for RFM segmentation.

BUSINESS RULE: Revenue excludes tax and shipping per data agreement.
ASSUMPTION: Unit prices are validated as positive during data ingestion.
EDGE CASE: Orders with quantity=0 are filtered pre-aggregation (duplicates).
INTERPRETATION: Used to compute "M" (Monetary value) in RFM analysis.

Args:
    df (DataFrame): Orders with 'quantity' and 'unit_price' columns
    
Returns:
    Series: Revenue in USD
"""
df['revenue'] = df['quantity'] * df['unit_price']  # Total transaction value
```

---

## Function Documentation Template

Every function must include a docstring explaining:

```python
def calculate_rfm_score(orders_df, reference_date=None):
    """
    Compute Recency, Frequency, Monetary (RFM) scores for customer segmentation.
    
    BUSINESS QUESTION: Which customer segments have highest lifetime value?
    METHODOLOGY: RFM clustering (K-means, k=3) on normalized scores.
    ASSUMPTIONS:
      - Orders with status='Completed' are valid transactions
      - Recency measured from reference_date (default: today)
      - Frequency counts all orders per customer
      - Monetary sums total revenue (USD)
    
    EDGE CASES:
      - Customers with no completed orders → excluded from analysis
      - Reference date in future → returns 0 recency
      - Negative revenue → filtered during data validation
    
    PERFORMANCE NOTE: Scales to 10M+ records; uses NumPy vectorization.
    
    Args:
        orders_df (DataFrame): Columns: [OrderID, CustomerID, Date, TotalPrice, Status]
        reference_date (str): ISO format date (YYYY-MM-DD). Default: today
    
    Returns:
        DataFrame: Columns: [CustomerID, Recency, Frequency, Monetary, RFM_Score]
        
    Example:
        >>> rfm = calculate_rfm_score(orders, reference_date='2026-06-02')
        >>> print(rfm.head())
        CustomerID  Recency  Frequency  Monetary  RFM_Score
        5042        12       8          9520      VIP
        5043        45       2          2180      Regular
    """
    # Implementation...
```

---

## Visualization Documentation

Every chart in the report should include metadata:

```python
{
    "chart_id": "chart_001",
    "title": "Revenue Distribution by Product",
    "chart_type": "box_plot",
    
    # BUSINESS CONTEXT
    "business_question": "Which products have most consistent revenue?",
    "purpose": "Identify revenue variance; flag high-variance products for review",
    
    # DATA SOURCE
    "data_source": "SELECT product, revenue FROM orders WHERE status='Completed'",
    "filters_applied": ["status = 'Completed'"],
    "record_count": 952,
    
    # VISUAL ENCODING
    "visual_encoding": {
        "x_axis": "Product",
        "y_axis": "Revenue (USD)",
        "color_by": "Product Category",
        "outlier_marker": "Diamond"
    },
    
    # INTERPRETATION
    "interpretation": (
        "High variance in Product A ($180K-$380K) suggests pricing/demand volatility. "
        "Recommend review of discount strategy or seasonal impact."
    ),
    "insights_from_chart": [
        "Product A has 2.1x revenue variance vs. average",
        "12 orders flagged as outliers (>Q3 + 1.5×IQR)"
    ],
    
    # NEXT STEPS
    "recommendation": "Investigate Product A demand drivers; A/B test pricing",
    "follow_up_analysis": "Correlation with marketing spend, seasonality"
}
```

---

## Python Script Structure

### Template: generate_eda_report.py

```python
#!/usr/bin/env python3
"""
Module: generate_eda_report.py

PURPOSE: 
    Generate interactive HTML EDA report from raw order data.
    Transforms raw CSV → validation → aggregation → JSON → HTML rendering

OWNER: Data Engineering Team
CONTACT: data@decodelabs.com
LAST MODIFIED: 2026-06-02

USAGE:
    python generate_eda_report.py \
        --input data/raw/orders.csv \
        --output data/outputs/ \
        --remove-outliers=false \
        --log-level=INFO

DEPENDENCIES:
    - pandas >= 1.3.0
    - numpy >= 1.21.0
    - plotly >= 5.0.0
    - scipy >= 1.7.0

OUTPUT FILES:
    - data/outputs/EDA_Report_DecodeLabs.html (Main report)
    - data/outputs/eda_metadata.json (Analysis metadata)
"""

import argparse
import logging
from pathlib import Path
import json
from datetime import datetime

import pandas as pd
import numpy as np
from plotly import graph_objects as go


# ============================================================================
# LOGGING CONFIGURATION
# ============================================================================

def setup_logging(log_level):
    """
    Configure logging with timestamp and level indicators.
    
    ASSUMPTION: Logs written to stdout; errors to stderr
    
    Args:
        log_level (str): DEBUG, INFO, WARNING, ERROR, CRITICAL
    """
    logging.basicConfig(
        level=getattr(logging, log_level),
        format='[%(asctime)s] %(levelname)-8s %(message)s',
        handlers=[
            logging.StreamHandler(),
            logging.FileHandler('generate_eda_report.log')
        ]
    )
    return logging.getLogger(__name__)


# ============================================================================
# DATA LOADING & VALIDATION
# ============================================================================

def load_raw_data(input_path):
    """
    Load and type-convert raw order CSV.
    
    BUSINESS RULE: All orders must have OrderID, Date, TotalPrice (not nullable)
    EDGE CASES:
      - Missing CustomerID → impute with mode (most common customer)
      - Date parsing errors → skip record and log warning
      - Negative TotalPrice → flag as anomaly, retain for analysis
    
    Args:
        input_path (str): Path to CSV file
        
    Returns:
        DataFrame: Validated orders with proper dtypes
        
    Raises:
        FileNotFoundError: If input file does not exist
        ValueError: If critical columns missing
    """
    logger = logging.getLogger(__name__)
    logger.info(f"Loading data from {input_path}")
    
    # Load with type hints
    dtype_map = {
        'OrderID': 'int64',
        'CustomerID': 'Int64',  # Nullable integer
        'TotalPrice': 'float64',
        'Quantity': 'int64',
        'OrderStatus': 'string'
    }
    
    df = pd.read_csv(input_path, dtype=dtype_map)
    logger.info(f"Loaded {len(df)} records from {input_path}")
    
    return df


def validate_data(df):
    """
    Perform data quality checks.
    
    VALIDATION RULES:
      1. No null OrderID, Date, TotalPrice
      2. TotalPrice >= 0
      3. Date within operational window (2020-2030)
      4. OrderStatus in enum: ['Completed', 'Pending', 'Cancelled']
    
    ACTIONS:
      - Report: Log validation summary
      - Flag: Mark invalid records
      - Retain: Keep for analysis (do not remove)
    
    Args:
        df (DataFrame): Orders dataframe
        
    Returns:
        tuple: (valid_df, validation_report)
    """
    logger = logging.getLogger(__name__)
    
    validation_report = {
        'total_records': len(df),
        'issues': []
    }
    
    # Check for nulls in critical columns
    critical_cols = ['OrderID', 'Date', 'TotalPrice']
    for col in critical_cols:
        if df[col].isnull().any():
            count = df[col].isnull().sum()
            validation_report['issues'].append(
                f"{col}: {count} nulls ({100*count/len(df):.1f}%)"
            )
    
    logger.info(f"Validation: {json.dumps(validation_report, indent=2)}")
    return df, validation_report


# ============================================================================
# ANALYSIS & AGGREGATION
# ============================================================================

def compute_descriptive_statistics(df):
    """
    Calculate descriptive statistics for numeric columns.
    
    STATISTICS COMPUTED:
      - Mean, Median, Std Dev, Min, Max
      - Quartiles (Q1, Q2/Median, Q3)
      - Skewness, Kurtosis (distribution shape)
      - IQR (Interquartile Range) for outlier detection
    
    USES:
      - Outlier flagging
      - Distribution visualization
      - Summary reporting
    
    Args:
        df (DataFrame): Orders
        
    Returns:
        dict: Statistics keyed by column name
    """
    numeric_cols = df.select_dtypes(include=[np.number]).columns
    
    stats = {}
    for col in numeric_cols:
        col_data = df[col].dropna()
        
        q1, q3 = col_data.quantile([0.25, 0.75])
        iqr = q3 - q1
        
        stats[col] = {
            'count': len(col_data),
            'mean': float(col_data.mean()),
            'std': float(col_data.std()),
            'min': float(col_data.min()),
            'q1': float(q1),
            'median': float(col_data.median()),
            'q3': float(q3),
            'max': float(col_data.max()),
            'iqr': float(iqr),
            'skewness': float(col_data.skew()),
            'kurtosis': float(col_data.kurtosis())
        }
    
    return stats


# ============================================================================
# HTML GENERATION
# ============================================================================

def generate_html_report(data_dict, output_path):
    """
    Generate HTML report with embedded data and Plotly visualizations.
    
    STRUCTURE:
      1. <head>: Meta, CDN links (Plotly, Bootstrap)
      2. <body>: Sections (Overview, Statistics, Charts)
      3. <script>: Embedded JSON + Chart rendering
    
    Args:
        data_dict (dict): Aggregated analysis data
        output_path (str): Output HTML file path
    """
    html_content = f"""
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>EDA Report — DecodeLabs</title>
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    </head>
    <body>
        <div class="container mt-5">
            <h1>EDA Report — DecodeLabs</h1>
            <p>Generated: {datetime.now().isoformat()}</p>
            
            <div id="chart-container"></div>
        </div>
        
        <script>
            window.eda_data = {json.dumps(data_dict)};
        </script>
    </body>
    </html>
    """
    
    with open(output_path, 'w') as f:
        f.write(html_content)
    
    logging.info(f"Report generated: {output_path}")


# ============================================================================
# MAIN EXECUTION
# ============================================================================

def main():
    """Orchestrate full EDA pipeline."""
    parser = argparse.ArgumentParser(description='Generate EDA Report')
    parser.add_argument('--input', required=True, help='Input CSV path')
    parser.add_argument('--output', required=True, help='Output directory')
    parser.add_argument('--log-level', default='INFO', help='Logging level')
    
    args = parser.parse_args()
    logger = setup_logging(args.log_level)
    
    # Load & validate
    df = load_raw_data(args.input)
    df, validation = validate_data(df)
    
    # Analyze
    stats = compute_descriptive_statistics(df)
    
    # Export
    output_file = Path(args.output) / 'EDA_Report_DecodeLabs.html'
    generate_html_report({'statistics': stats}, str(output_file))
    
    logger.info("✓ EDA report generation complete")


if __name__ == '__main__':
    main()
```

---

## Variable Naming Conventions

| Pattern | Usage | Example |
|---------|-------|---------|
| `is_*, has_*, should_*` | Boolean variables | `is_outlier`, `has_missing_values` |
| Plural nouns | Collections | `outliers`, `segments`, `records` |
| `UPPER_SNAKE_CASE` | Constants | `MAX_PRICE = 10000`, `IQR_THRESHOLD = 1.5` |
| `_private_function()` | Internal helpers | `_normalize_prices()`, `_compute_iqr()` |
| `calculate_*, compute_*` | Computations | `calculate_rfm_score()`, `compute_statistics()` |

---

## Error Handling Pattern

```python
def load_and_validate(input_path):
    """
    Load data with comprehensive error handling.
    
    FAILURES HANDLED:
      - FileNotFoundError: Input file does not exist
      - ValueError: CSV has wrong columns or types
      - MemoryError: Dataset too large for available RAM
    
    STRATEGY: Fail fast with informative error; log full context
    """
    try:
        if not Path(input_path).exists():
            raise FileNotFoundError(f"Input file not found: {input_path}")
        
        df = pd.read_csv(input_path)
        
        required_cols = ['OrderID', 'Date', 'TotalPrice']
        if not all(col in df.columns for col in required_cols):
            missing = set(required_cols) - set(df.columns)
            raise ValueError(f"Missing required columns: {missing}")
        
        return df
        
    except FileNotFoundError as e:
        logging.error(f"Cannot load data: {e}")
        raise
    except ValueError as e:
        logging.error(f"Data validation failed: {e}")
        raise
    except MemoryError:
        logging.error("Dataset too large; consider chunking or downsampling")
        raise
```

---

## Code Review Checklist

Before committing analysis code:

- [ ] Function has docstring explaining "why" and assumptions
- [ ] Function parameters documented with types
- [ ] Return values clearly specified
- [ ] Edge cases listed (null values, negative numbers, etc.)
- [ ] Data source documented (SQL query or CSV reference)
- [ ] Comments explain business logic, not syntax
- [ ] Error handling present for file I/O and parsing
- [ ] Variables follow naming conventions
- [ ] Logging statements track progress and failures
- [ ] Script runs without errors: `python script.py --help`

---

## Testing Template

```python
import pytest

def test_calculate_rfm_score():
    """
    Test RFM scoring with known data.
    
    SCENARIO: 3 customers with known RFM values
    EXPECTED: RFM score matches manual calculation
    """
    # Arrange: Create test data
    test_data = pd.DataFrame({
        'OrderID': [1, 2, 3],
        'CustomerID': [100, 100, 101],
        'Date': ['2026-01-15', '2026-06-01', '2026-05-30'],
        'TotalPrice': [100, 200, 50],
        'Status': ['Completed', 'Completed', 'Completed']
    })
    
    # Act: Compute RFM
    rfm = calculate_rfm_score(test_data, reference_date='2026-06-02')
    
    # Assert: Verify results
    assert rfm.loc[rfm['CustomerID'] == 100, 'Frequency'].values[0] == 2
    assert rfm.loc[rfm['CustomerID'] == 100, 'Monetary'].values[0] == 300
```

---

## Documentation Maintenance

- Update docstrings when changing function behavior
- Add inline comments for non-obvious logic
- Document all assumptions (data format, business rules)
- Keep README.md in sync with project structure
- Version control all analysis code

---

## Quality Metrics

| Metric | Target | How to Measure |
|--------|--------|---|
| Code Coverage | ≥80% | `pytest --cov` |
| Documentation | 100% | `pydocstyle` |
| Type Hints | ≥80% | `mypy` |
| Linting | A grade | `pylint`, `flake8` |
| Docstring Completeness | All functions | Manual review |

