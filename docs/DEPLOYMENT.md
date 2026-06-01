# DEPLOYMENT.md — Hosting & Distribution Guide

## Pre-Deployment Checklist

- [ ] HTML report generated and validated (no broken links)
- [ ] All embedded JSON is valid (test in browser console)
- [ ] Charts render correctly in target browsers (Chrome, Firefox, Safari)
- [ ] Plotly CDN is accessible (test with offline mode disabled)
- [ ] No console errors (check DevTools → Console tab)
- [ ] Report file size is acceptable (<100 MB)
- [ ] Data is anonymized (no PII embedded)
- [ ] README and documentation complete
- [ ] Git repository initialized and clean

---

## Deployment Options

### Option 1: GitHub Pages (Recommended for Teams)

**Advantages**:
- Free hosting
- Integrated with version control
- HTTPS by default
- Automatic deployment on Git push

**Steps**:

```bash
# 1. Ensure Git repo is initialized
cd ~/DecodeLabs/p2
git init

# 2. Add all files
git add .
git commit -m "docs: Add EDA report with documentation"

# 3. Push to GitHub
git remote add origin https://github.com/Boukehamohamedakram/DecodeLabs-Internship-P2-EDA.git
git branch -M main
git push -u origin main

# 4. Enable GitHub Pages
# In GitHub repo settings:
#   - Settings → Pages
#   - Source: Deploy from a branch
#   - Branch: main / folder: /(root)
#   - Save

# 5. Access via: https://github.com/Boukehamohamedakram/DecodeLabs-Internship-P2-EDA/blob/main/data/outputs/EDA_Report_DecodeLabs.html
#    OR (if Pages enabled): https://boukehamohamedakram.github.io/DecodeLabs-Internship-P2-EDA/data/outputs/EDA_Report_DecodeLabs.html
```

**File URL Format**:
```
https://raw.githubusercontent.com/Boukehamohamedakram/DecodeLabs-Internship-P2-EDA/main/data/outputs/EDA_Report_DecodeLabs.html
```

---

### Option 2: Netlify Drop (Easiest for Static Files)

**Advantages**:
- Zero configuration
- Drag-and-drop deployment
- Free SSL/HTTPS
- Instant updates

**Steps**:

```bash
# 1. Visit https://app.netlify.com/drop
# 2. Drag data/outputs/ folder into Netlify
# 3. Get instant URL: https://[random-name].netlify.app/EDA_Report_DecodeLabs.html
# 4. Custom domain: Connect GitHub repo for auto-deployment
```

---

### Option 3: AWS S3 + CloudFront (Scalable)

**Advantages**:
- Highly scalable
- Global CDN (CloudFront)
- Fine-grained access control
- Suitable for enterprise

**Steps**:

```bash
# 1. Create S3 bucket
aws s3 mb s3://decodelabs-eda-reports

# 2. Upload HTML
aws s3 cp data/outputs/EDA_Report_DecodeLabs.html \
  s3://decodelabs-eda-reports/2026-06-02/EDA_Report.html \
  --acl public-read \
  --content-type text/html

# 3. Set up CloudFront distribution
# In AWS Console:
#   - CloudFront → Create Distribution
#   - Origin: S3 bucket
#   - Default cache TTL: 86400 (1 day)
#   - Enable HTTPS

# 4. Access via: https://d1234abc.cloudfront.net/2026-06-02/EDA_Report.html
```

**Cost Estimate**:
- S3 storage: ~$0.01 per report
- CloudFront: ~$0.085 per GB
- For 1,000 downloads: ~$85

---

### Option 4: Local HTTP Server (Internal Sharing)

**Advantages**:
- No external dependencies
- Works offline
- Ideal for internal reviews

**Steps**:

```bash
# Python 3
cd ~/DecodeLabs/p2/data/outputs
python -m http.server 8000

# Access: http://localhost:8000/EDA_Report_DecodeLabs.html
# Share: http://[your-internal-ip]:8000/EDA_Report_DecodeLabs.html

# Stop: Ctrl+C
```

---

## Performance Optimization

### 1. Enable Gzip Compression

```bash
# Most web servers (Netlify, GitHub Pages, S3 with CloudFront) auto-gzip

# Test: Check Content-Encoding header
curl -I https://your-report-url.html | grep Content-Encoding
# Expected: Content-Encoding: gzip
```

### 2. Optimize Plotly Bundle

**Current**: ~3 MB (CDN)
**Option A**: Keep CDN (fastest for cached users)
**Option B**: Inline Plotly (self-contained)

```html
<!-- Current approach (CDN) -->
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

<!-- Alternative: Inline (larger initial file) -->
<script>
  /* Plotly library embedded inline (~800 KB gzipped) */
</script>
```

### 3. Lazy Load Charts

```javascript
// Load charts only when section becomes visible
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      renderChart(entry.target.id);
      observer.unobserve(entry.target);
    }
  });
});

document.querySelectorAll('.chart-container').forEach(el => {
  observer.observe(el);
});
```

---

## Monitoring & Analytics

### Google Analytics (Optional)

```html
<!-- Add to <head> -->
<script async src="https://www.googletagmanager.com/gtag/js?id=GA_MEASUREMENT_ID"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'GA_MEASUREMENT_ID');
</script>
```

**Tracked Events**:
- Page views
- Chart interactions (zoom, hover)
- Export clicks
- Section tab navigation

---

## Update Workflow

### When Data Changes

```bash
# 1. Update source data in data/raw/
cp ~/new_orders_data.csv data/raw/orders.csv

# 2. Regenerate report
python scripts/generate_eda_report.py \
  --input data/raw/orders.csv \
  --output data/outputs/

# 3. Commit changes
git add -A
git commit -m "docs: Update EDA report with Q2 2026 data"
git push origin main

# 4. Verify deployment
# GitHub Pages: Auto-updates within 1 minute
# Netlify: Auto-updates on Git push
# S3 + CloudFront: Invalidate cache
#   aws cloudfront create-invalidation --distribution-id [ID] --paths "/*"
```

### Versioning Reports

```
data/outputs/
├── 2026-06-02-EDA_Report_Q2.html          # v1.0
├── 2026-06-02-EDA_Report_Q2_v2.html       # v2.0 (corrected)
├── archive/
│   ├── 2026-03-01-EDA_Report_Q1.html
│   └── 2025-12-01-EDA_Report_Q4.html
```

---

## Troubleshooting Deployment Issues

| Issue | Solution |
|-------|----------|
| **Charts not rendering** | Check browser console for CDN errors; fallback to local Plotly |
| **Slow load time** | Enable gzip; use CDN with CloudFront; measure with WebPageTest |
| **GitHub Pages 404** | Verify branch is set to "main" in Settings → Pages; check file path |
| **Netlify shows directory listing** | Add `_redirects` file: `/index.html 200` |
| **CloudFront old version served** | Invalidate cache: `aws cloudfront create-invalidation` |
| **Mixed content (HTTP/HTTPS)** | Update CDN URLs to use `https://` |

---

## Disaster Recovery

### Backup Strategy

```bash
# 1. Local backup
cp -r ~/DecodeLabs/p2 ~/Backups/DecodeLabs_P2_$(date +%Y%m%d).tar.gz

# 2. GitHub backup (automatic)
# All commits preserved in Git history

# 3. S3 backup (if using AWS)
aws s3 sync ~/DecodeLabs/p2 s3://decodelabs-backup/p2/
```

### Rollback Procedure

```bash
# If deployment breaks, rollback to previous commit
git log --oneline | head
# Output: 3a2f1e9 docs: Update EDA report...
#         7b8c9d2 docs: Add EDA report with documentation

git revert 3a2f1e9
git push origin main
```

---

## Security Checklist

- [ ] No sensitive data (API keys, passwords) in HTML
- [ ] No PII (names, emails, phone numbers) in embedded JSON
- [ ] HTTPS enabled (automatic on Netlify, GitHub Pages)
- [ ] CORS headers configured (if serving via API later)
- [ ] Content Security Policy (CSP) headers set:
  ```
  Content-Security-Policy: 
    default-src 'self'; 
    script-src 'self' https://cdn.plot.ly; 
    style-src 'self' 'unsafe-inline'
  ```

---

## Production Checklist

```bash
# Before going live:

# 1. Validate HTML
npm install -g html-validator
html-validator --filename=data/outputs/EDA_Report_DecodeLabs.html

# 2. Check for broken links
npm install -g broken-link-checker
blc https://your-deployment-url

# 3. Performance audit
lighthouse https://your-deployment-url

# 4. Accessibility audit
npm install -g axe-core
# Run in browser console: axe.run()

# 5. Test in multiple browsers
# Chrome, Firefox, Safari, Edge

# 6. Test on mobile
# Use Chrome DevTools → Device Toolbar

# 7. Final commit
git add -A
git commit -m "chore: Production deployment checklist complete"
git push origin main
```

---

## Post-Deployment Monitoring

### Health Checks

```bash
# Weekly: Verify URL accessibility
curl -I https://your-report-url.html | grep HTTP

# Monthly: Check for JavaScript errors
# Use Sentry, LogRocket, or browser error tracking

# Quarterly: Performance audit
lighthouse https://your-report-url.html
```

---

## Support

For deployment issues:
1. Check GitHub Issues: https://github.com/Boukehamohamedakram/DecodeLabs-Internship-P2-EDA/issues
2. Review logs (Netlify: https://app.netlify.com/teams/[team]/sites → Deploys)
3. Contact: Refer to README.md for support contact
