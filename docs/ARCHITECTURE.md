# ARCHITECTURE.md — Design Decisions & Rationale

## Overview

The EDA Report is a **static HTML artifact** generated from exploratory data analysis. This document explains architectural choices, trade-offs considered, and rationale for the selected technology stack.

---

## Design Decision 1: Static HTML vs. Interactive Frameworks

### Decision: HTML5 + Plotly.js (Static with embedded JavaScript)

### Considered Alternatives

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| **Static HTML (Selected)** | Self-contained, no server, instant deployment | Limited interactivity, single export per analysis | ✅ **Chosen** |
| **React/Vue SPA** | Fully interactive, real-time filtering | Requires Node.js backend, complex deployment | ❌ Rejected |
| **Jupyter Notebook** | Great for exploration, version control | Overkill for static report, slow to load | ❌ Rejected |
| **Tableau/Power BI** | Professional, enterprise features | Expensive licensing, vendor lock-in | ❌ Rejected |
| **PDF Report** | Universal compatibility | No interactivity, large file size, no charts | ❌ Rejected |

### Rationale

**Static HTML was chosen because:**

1. **Deliverable Simplicity**: Single file artifact; no server infrastructure
2. **Accessibility**: Works offline; no runtime dependencies beyond browser
3. **Version Control**: HTML source code trackable in Git
4. **Deployment**: Distribute via email, Git, S3, or GitHub Pages instantly
5. **Stakeholder Communication**: Interactive charts without requiring software installation

### Trade-Offs

- **Lost**: Real-time data refresh (requires data pipeline trigger)
- **Lost**: Advanced filtering/slicing (static snapshots only)
- **Gained**: Zero infrastructure cost, instant sharing

---

## Design Decision 2: Data Processing Strategy

### Decision: Python (Pandas/NumPy) for Analysis; Export to JSON

### Rationale

1. **Python Ecosystem**: Pandas/NumPy are industry-standard for tabular data
2. **Reproducibility**: Scripts version-controlled; analysis fully auditable
3. **JSON Embedding**: Pre-computed metrics embedded in HTML avoid runtime calculations
4. **Scalability**: Processing separated from rendering (future: migrate to Spark)

### Data Pipeline

```
Raw CSV → Python (Pandas) → Validation → Aggregation → JSON Export → HTML Embedding
  (1)              (2)           (3)           (4)            (5)            (6)
```

1. **Load**: Read CSV with specified types
2. **Clean**: Handle nulls, type conversions, outlier flags
3. **Validate**: Schema checks, statistical bounds
4. **Compute**: Summary statistics, aggregations, RFM segments
5. **Export**: JSON with metrics, chart data, metadata
6. **Embed**: Insert JSON into HTML template

### Why Not SQL?

- SQLite is embedded in data, but Python provides richer statistical functions
- Future evolution: Could migrate to SQL if scaling to billions of records
- Current dataset (1,200 rows) benefits from Python's flexibility

---

## Design Decision 3: Visualization Library

### Decision: Plotly.js for Interactive Charts

### Considered Alternatives

| Option | Interactivity | Learning Curve | Bundle Size | Verdict |
|--------|---------------|---------------|----|---------|
| **Plotly (Selected)** | High (zoom, pan, hover) | Medium | 3 MB CDN | ✅ **Chosen** |
| **D3.js** | Very high (complete control) | Steep | Custom | ❌ Too complex |
| **Chart.js** | Low (basic tooltips) | Easy | Small | ❌ Limited interactivity |
| **Matplotlib (PNG export)** | None (static images) | Easy | N/A | ❌ No interactivity |
| **Apache ECharts** | High | Medium | 1.5 MB | ⚠️ Alternative |

### Rationale

1. **Interactivity**: Plotly provides zoom, pan, hover tooltips out-of-the-box
2. **No Server**: All visualization logic runs in browser (no Python backend needed)
3. **Export**: Charts downloadable as PNG for presentations
4. **Responsive**: Auto-scales to desktop/tablet without code changes
5. **Documentation**: Excellent documentation for rapid development

### Bundle Size Concern

- Plotly CDN: ~3 MB (gzipped: ~800 KB)
- Impact: First load ~2 seconds on 4G; acceptable for static report
- Mitigation: Could use ECharts (1.5 MB) or D3.js (custom, smaller) if size critical

---

## Design Decision 4: Data Storage Format

### Decision: Embedded JSON in HTML

### Rationale

1. **Self-Contained**: Report is single file; no external data dependencies
2. **Security**: No server queries; data never exposed to network
3. **Portability**: Works offline; can email as attachment
4. **Version Control**: All data changes visible in Git diff

### Embedded JSON Structure

```html
<script>
  window.eda_data = {
    "dataset_overview": { /* summary stats */ },
    "numeric_distributions": [ /* histogram data */ ],
    "categorical_distributions": [ /* bar chart data */ ],
    "correlation_matrix": [ /* heatmap data */ ],
    "outliers": [ /* flagged records */ ],
    "segments": [ /* RFM clusters */ ]
  };
</script>
```

### Drawback: Large Files

- Embedded JSON increases HTML size (~15 MB for 1M+ rows)
- **Mitigation**: Compress with gzip; use external JSON file for very large datasets

---

## Design Decision 5: Styling & Theme

### Decision: Bootstrap + Custom CSS (Dark Theme)

### Rationale

1. **Professional Appearance**: Bootstrap ensures consistent, modern layout
2. **Accessibility**: Dark theme reduces eye strain; high contrast ratios (WCAG AA)
3. **Responsive**: Mobile-friendly without custom breakpoints
4. **Customizable**: Easy theme switching (light/dark toggle possible)

### CSS Variables for Theming

```css
:root {
  --bg-primary: #0f1117;      /* Page background */
  --bg-secondary: #181c27;    /* Cards, sections */
  --text-primary: #e6edf3;    /* Main text */
  --text-secondary: #8b949e;  /* Secondary text */
  --accent: #4a9edd;          /* Links, highlights */
  --warning: #d1632d;         /* Anomalies */
  --success: #238636;         /* Positive findings */
}
```

---

## Design Decision 6: Hypothesis Generation Approach

### Decision: Include "Next Steps" Section with Actionable Hypotheses

### Rationale

1. **Bridge Gap**: EDA is exploratory; report should suggest modeling directions
2. **Stakeholder Communication**: Translates findings into business questions
3. **Reproducibility**: Hypotheses are explicitly documented for future team members

### Hypothesis Examples

- "Does seasonal demand correlate with marketing spend?" → Requires: Marketing data
- "Can we predict churn using RFM features?" → Requires: Churn labels, historical data
- "What drives product-level cancellation variance?" → Requires: Product attributes, A/B test results

---

## Design Decision 7: Outlier Handling

### Decision: Flag & Retain (Not Remove)

### Rationale

1. **Domain Knowledge**: High-value orders ($2,000+) are legitimate transactions, not errors
2. **Bias Prevention**: Removing outliers without investigation introduces bias
3. **Business Value**: Outlier analysis reveals customer segments (VIP customers)
4. **Auditability**: Explicit documentation of outlier treatment

### Outlier Definition

- **Price Outliers**: IQR method (Q1 - 1.5×IQR, Q3 + 1.5×IQR)
- **Quantity Outliers**: Orders with qty > 100 (bulk purchases)
- **Revenue Outliers**: Orders > 95th percentile ($3,500+)

---

## Design Decision 8: Reproducibility & Documentation

### Decision: Include Python Scripts; Document All Assumptions

### Rationale

1. **Auditability**: Future analysts can verify calculations
2. **Maintenance**: Original author's reasoning preserved in comments
3. **Scalability**: Scripts adaptable to new datasets
4. **Best Practice**: Professional data science standards

### Script Organization

```
scripts/
├── generate_eda_report.py    # Main pipeline
├── validate_data.py          # Quality checks
├── compute_statistics.py     # Aggregations
└── export_charts.py          # Visualization generation
```

Each script includes:
- Docstring explaining purpose
- Inline comments for complex logic
- Error handling with informative messages
- Logging for debugging

---

## Future Evolution (v2.0)

### Enhancement 1: Real-Time Data Refresh

- Current: Report is static snapshot
- Future: Connect to live database; regenerate on schedule
- Technology: GitHub Actions (automated report generation)

### Enhancement 2: Interactive Filtering

- Current: Fixed visualizations
- Future: Click filters to slice data (requires backend API)
- Technology: React + Node.js backend

### Enhancement 3: Automated Anomaly Detection

- Current: Manual IQR-based outlier flagging
- Future: Machine learning (Isolation Forest, LOF) for sophisticated detection
- Technology: Scikit-learn, AutoML

### Enhancement 4: Mobile-Optimized Report

- Current: Works on tablet; suboptimal on mobile
- Future: Responsive chart sizing; mobile-specific layouts
- Technology: CSS media queries, Plotly responsive sizing

---

## Performance Considerations

### Report File Size

| Component | Size | Gzipped |
|-----------|------|---------|
| HTML Structure | 2 KB | 1 KB |
| Bootstrap CSS | 150 KB | 35 KB |
| Plotly.js CDN | 3 MB | 800 KB |
| Embedded JSON | 28 KB | 8 KB |
| **Total** | ~3.18 MB | ~844 KB |

### Load Time Optimization

1. **CDN Delivery**: Plotly served from CDN (cached globally)
2. **Gzip Compression**: Browser auto-decompresses (.gzip negotiation)
3. **Lazy Loading**: Charts render on tab-click (not all at once)
4. **Bundling**: Consider bundling Plotly locally for offline reliability

---

## Security Considerations

### Data Privacy

- **Embedded Data**: All data visible in HTML source (not encrypted)
- **Implication**: Don't embed PII; anonymize customer data (use IDs, not names)
- **Mitigation**: Store sensitive data in separate protected JSON; reference by key

### Injection Attacks

- **Risk**: Plotly allows custom HTML in chart annotations
- **Mitigation**: Sanitize all user inputs; use Plotly's built-in escaping

### CORS (Cross-Origin Resource Sharing)

- **Issue**: External CDN dependencies
- **Mitigation**: Inline Plotly library locally if CDN is unreliable

---

## Accessibility (WCAG 2.1 AA)

### Color Contrast

- Dark theme (#0f1117 bg, #e6edf3 text) = 15:1 contrast ratio ✅ (exceeds 7:1 AA standard)
- Warning color (#d1632d) = 5.2:1 against background ✅

### Keyboard Navigation

- All interactive elements (tabs, buttons) are keyboard-accessible
- Tab order follows visual flow

### Alt Text for Charts

- Each chart includes SVG title and description
- Screen readers read: "Chart: Revenue Distribution by Product. Box plot shows revenue variance across 7 product categories. Q1 median is $180K, Q3 is $320K."

---

## Conclusion

The architectural choices prioritize:

1. **Simplicity**: Single-file deliverable
2. **Accessibility**: Works offline, no dependencies
3. **Professionalism**: Interactive visualizations, polished design
4. **Auditability**: Scripts document methodology
5. **Scalability**: Foundation for future enhancements

This report is production-ready for static analysis sharing; future evolution can add interactivity as business needs evolve.
