# PepsiCo Capstone Project — Brand Momentum & Whitespace Analytics

A multi-workstream analytics framework built for PepsiCo to identify high-momentum beverage brands, emerging flavor territories, and innovation/renovation opportunities in the U.S. Water and Carbonated Soft Drinks (CSD) categories, using Circana's 2021–2025 syndicated dataset.

**Course:** OPIM 5770 — Advanced Business Analytics and Project Management, UConn School of Business
**Instructors:** Kelly Fisher, Eric McKean
**Team:** Vivek Pant, Christopher Jacob, Srijanani Achuthan, Adriana Dorris
**Term:** Fall 2025

---

## Executive Summary

This project explores long-term growth patterns in the U.S. Carbonated Soft Drinks and Water categories using a five-year syndicated dataset (2021–2025), with the goal of identifying sustained brand momentum and strategic opportunities for PepsiCo as consumer preferences shift toward multicultural flavors, functional benefits, and premium positioning. The project combines a multi-year momentum scoring framework, K-Means brand segmentation, an emerging-brand predictor with PCA-based health feature reduction, and a still-vs-sparkling water performance comparison — producing concrete, actionable lists of "Whitespace Insurgent" brands (innovation/acquisition targets) and "Pepsi Renovate" brands (underperformers needing strategic review).

---

## Problem Statement

PepsiCo asked a graduate consulting team to develop a structured, analytics-based approach for detecting early and sustained growth signals in the beverage landscape. Traditional category reporting is descriptive but doesn't reliably surface emerging competitors, white-space opportunities, or brands likely to sustain multi-year performance. The project goal was to build a scalable, reproducible framework that:

- Identifies brands with strong early in-market signals or sustained multi-year growth
- Distinguishes emerging competitors from established leaders quantitatively
- Surfaces actionable flavor, claim, and positioning territories aligned with shifting consumer behavior
- Delivers insights PepsiCo can use immediately for innovation, portfolio, and scouting decisions

---

## Methodology

### 1. Still Water vs. Sparkling Water Analysis
- **Objective:** Determine whether Still Water outperforms Sparkling Water using a standardized **Composite Success Score** = 0.5 × 1YR CAGR + 0.5 × 1YR Acceleration
- **Hypothesis test:** One-sided t-test comparing mean Composite Success Scores (H1: µ_Still > µ_Sparkling)
- **Emerging brand identification:** Products launched within 24 months, not owned by major incumbents (PepsiCo, Coca-Cola, Nestlé, Keurig Dr Pepper, Danone), with above-median momentum, rising distribution, and improving velocity

### 2. Water Model — Predictive Framework
- **Hypothesis 1 (Early Pull):** Higher `log_sales_per_acv` during a brand's first 12 months predicts breakout success (top 10% efficiency)
- **Hypothesis 2 (Momentum > Distribution):** Growth momentum (CAGR) is more predictive of breakout success than distribution scale (ACV) alone
- **Targets:** Continuous `log_sales_per_acv = log(1 + TTM_Sales / ACV)`; binary `SUCCESS_FLAG` defined as the top 10% (90th percentile) of `log_sales_per_acv`
- **Models:** Logistic Regression (breakout classifier) and Gradient Boosting Regressor (efficiency regressor)

### 3. Multi-Year Momentum Analysis (CSD)
- Aggregated SKU-level data to brand level (1-, 2-, and 3-year dollar-sales CAGR)
- **Momentum Score** = 0.6 × CAGR(1Y) + 0.3 × CAGR(2Y) + 0.1 × CAGR(3Y), with missing multi-year CAGRs imputed from CAGR(1Y) for newer brands
- Ranked all brands to identify the Top 10 fastest-growing CSD brands over multiple horizons

### 4. Brand Segmentation via K-Means Clustering
- **Preprocessing:** Converted timestamps to determine launch dates, coerced numeric types, scaled distribution to 0–1, winsorized velocity at 1st/99th percentiles
- **Feature engineering (first 6 months post-launch):** Early Pull Brand (log), Early Velocity Mean (log), Early Distribution Mean, Velocity Slope (6M), Distribution Growth (6M)
- **Success Flag:** 1 if Max TTM Sales ≥ $25M
- **Classification:** Logistic Regression (balanced class weights), 70/30 stratified train/test split
- **Clustering:** Features standardized (StandardScaler), missing values imputed with column means; K = 3 selected via Elbow Method

### 5. Carbonated Drinks Emerging Brand Predictor
- **Data:** `CSD_DATA_ALL.csv`, aggregated from product-level to 266 brands (summed TTM $ Sales; averaged Distribution, Energy, Sugars, Velocity)
- **Feature engineering:** `Sales_per_ACV = TTM_$_SALES / (DISTRIBUTION + ε)`; `SUCCESS_FLAG` = top 10% of Sales_per_ACV; keyword-based Zero-Sugar and Energy flags
- **PCA:** Applied to standardized Energy (kcal/100g/mL) and Sugars (g/100g/mL) — PC1 explains 98.48% of variance (overall "caloric/sugar richness"); PC2 explains 1.52%
- **Models:**
  - **Innovation Model** (excludes Distribution) — Features: PC1, PC2, Zero_Sugar_Flag, Energy_Flag, one-hot SUB_CATEGORY
  - **Hybrid Model** (includes Distribution) — same features + Distribution
  - Classification via Logistic Regression (balanced); regression via Gradient Boosting Regressor
  - 80/20 train/test split, `random_state=42`, stratified for classification

---

## Key Results

### Water Model Performance
- **Breakout Classifier (Logistic Regression):** AUC = 0.759. Top PepsiCo water brands by breakout probability: LIFE WTR (0.900), Bubly (0.826), Aquafina (0.736)
- **Efficiency Regressor (Gradient Boosting):** R² = 0.304, MAE = 1.478 (log units). Top brands by predicted sales efficiency: LIFE WTR, Aquafina, Bubly

### Still Water vs. Sparkling Water
- Still Water Composite Success Score (0.1395) significantly exceeds Sparkling Water (0.0996), per t-test — a structural performance advantage, not solely distribution-driven
- **Still Water growth territories:** Functional Hydration (Lemon Perfect, Karma, BodyArmor), Premium Alkaline & Mineral Waters (Alkaline88, Flow, Hawaiian Springs), Sustainable Packaging (Pathwater, Flow)
- **Sparkling Water whitespace:** Botanical/herbal flavors (hibiscus, rose, mojito, lavender), unflavored sparkling water, and "clean label"/natural mineral-electrolyte claims — areas where PepsiCo is underrepresented

### CSD Multi-Year Momentum
- Top 10 momentum brands include international entrants (Ramune, Barrilitos, Shirakiku, A Siciliana — Japanese, Mexican, Italian, Caribbean) and functional sodas (Poppi, Olipop, Culture Pop, United Sodas of America)
- Five identified innovation gaps: global/multicultural flavors, botanical craft sodas, low-sugar/clean-label alternatives, functional soda territory, and premium packaging

### K-Means Brand Segmentation
- Balanced Logistic Regression achieved **ROC-AUC = 0.784** (Precision = 0.27, Recall = 0.75, F1 = 0.40 for the success class)
- **Top predictors:** Distribution EarlyMean (1.016), Velocity EarlyMean (log) (0.730), Early Pull Brand (log) (0.329)
- **Three brand archetypes (K=3):**
  - **Cluster 0 (183 brands):** Low distribution/velocity, modest early momentum
  - **Cluster 1 (5 brands):** Highest growth acceleration — "breakout potential" (e.g., DR PEPPER)
  - **Cluster 2 (89 brands):** Strong initial performance but early deceleration (potential saturation)

### CSD Emerging Brand Predictor
- **Innovation Model:** AUC = 0.592, R² = −0.491 (regression underperforms the mean)
- **Hybrid Model:** AUC = 0.616, R² = −26.830 — severe regression underperformance, but residuals revealed strategic signal
- **Whitespace Insurgents** (non-Pepsi brands far exceeding predicted sales): Canada Dry, Sprite, Faygo, Sunkist, Dr Pepper
- **Pepsi Renovate** (Pepsi brands far below predicted sales): Mist Twst, Sierra Mist, Manzanita Sol, Starry, Pepsi (Sugar/Calorie Reduced)

---

## Strategic Recommendations

1. **Invest in high-potential water brands** — LIFE WTR, Bubly, and Aquafina warrant increased marketing/distribution and product line extensions
2. **Monitor emerging water brands** — Be Light and select Crisp N Clear SKUs show high breakout probability despite low current distribution
3. **Review low-efficiency brands** — evaluate pricing, promotion, and supply chain for persistently underperforming brands; consider portfolio rationalization
4. **Build a global/multicultural flavor pipeline** (yuzu, tamarind, hibiscus, tropical citrus) to capture rising CSD momentum
5. **Develop a premium botanical craft soda platform** — entering via innovation, partnership, or acquisition
6. **Segmented investment by cluster** — aggressive investment in Cluster 1 (breakout) brands, sustaining initiatives for Cluster 2, iterative testing for Cluster 0
7. **Conduct deep-dive research on Whitespace Insurgents** (Canada Dry, Sprite, Faygo, Sunkist, Dr Pepper) for acquisition/emulation insights
8. **Strategic review for Pepsi Renovate brands** (Mist Twst, Sierra Mist, Manzanita Sol, Starry, Pepsi Sugar/Calorie Reduced) covering reformulation, repositioning, and pricing

---

## Limitations & Future Work

- The fixed TTM Sales threshold ($25M) for `SUCCESS_FLAG` may not capture nuances like profitability or niche market share
- Regression models for the CSD Emerging Brand Predictor showed poor fit (negative R²), indicating the current feature set is insufficient for precise quantitative forecasting
- Analysis excludes external factors: marketing spend, competitive activity, and macroeconomic indicators
- Future enhancements: enriched feature sets (marketing spend, digital engagement, competitor pricing), dynamic success definitions incorporating market size/lifecycle stage, advanced models (gradient boosting, neural networks, survival analysis), and qualitative research to complement quantitative findings

---

## Tools & Technologies

- **Python** — pandas, NumPy, scikit-learn (Logistic Regression, Gradient Boosting, K-Means, StandardScaler, PCA), matplotlib, seaborn
- **Statistical Methods** — Hypothesis testing (t-test), PCA, K-Means clustering (Elbow Method), logistic & gradient boosting regression
- **Data Source** — Circana 5-year syndicated beverage dataset (2021–2025), `CSD_DATA_ALL.csv`, `soft_drinks_5yr_COMBINED.csv`

---

## References

1. Pedregosa, F., et al. (2011). *Scikit-learn: Machine Learning in Python*. Journal of Machine Learning Research, 12, 2825–2830.
2. The pandas development team. (2020). *pandas-dev/pandas: Pandas*. Zenodo.
3. Hunter, J. D. (2007). *Matplotlib: A 2D Graphics Environment*. Computing in Science & Engineering, 9(3), 90–95.
4. Waskom, M. L. (2021). *seaborn: statistical data visualization*. Journal of Open Source Software, 6(60), 3021.

---

## Repository Contents

- `Team1_FinalPaper.pdf` — Full written report with methodology, results, and appendix code
- `Team1_Final_Presentation.pptx` — Final presentation slide deck
- Appendix A (in report) — Python code for multi-year momentum score calculation and Top 10 brand ranking
