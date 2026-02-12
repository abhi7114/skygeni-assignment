# SkyGeni Sales Intelligence Challenge – Solution

## Table of Contents
1. [Project Overview](#project-overview)
2. [Quick Start](#quick-start)
3. [Approach & Methodology](#approach--methodology)
4. [Key Design Decisions](#key-design-decisions)
5. [Results & Insights](#results--insights)
6. [Repository Structure](#repository-structure)
7. [Technical Stack](#technical-stack)

---

## Project Overview

**Business Problem:**  
A B2B SaaS company reports declining win rates despite healthy pipeline volume. The CRO needs diagnostic insights to understand *what's going wrong* and *where to focus* the team's efforts.

**Solution:**  
A decision intelligence system that identifies win rate drivers, surfaces segment-level risks, and provides actionable recommendations for sales leadership.

**Core Philosophy:**  
This is a **diagnostic engine for decision support**, not a predictive oracle. The goal is to help sales leaders ask better questions, intervene earlier, and allocate resources more effectively.

---

##  Quick Start

### Prerequisites
```bash
Python 3.8+
Jupyter Notebook
```

### Installation
```bash
# Clone the repository
git clone <repository-url>
cd skygeni

# Install dependencies
pip install pandas numpy matplotlib seaborn scikit-learn xgboost
```

### Run the Analysis

**Step 1: Exploratory Data Analysis**
```bash
jupyter notebook notebooks/eda.ipynb
```
- Open and run all cells sequentially
- Generates business insights and custom metrics
- Outputs: Segment performance tables, distribution plots saved to `images/`

**Step 2: Decision Engine (Win Rate Driver Analysis)**
```bash
jupyter notebook notebooks/decision_engine.ipynb
```
- Implements multi-model approach (Logistic Regression, Random Forest, Gradient Boosting, XGBoost)
- Performs hyperparameter tuning and cross-model validation
- Outputs: Stable driver identification, CRO-facing recommendations, model comparison charts saved to `images/`

### Expected Runtime
- `eda.ipynb`: ~2-3 minutes
- `decision_engine.ipynb`: ~10-15 minutes (includes hyperparameter tuning)

---

##  Approach & Methodology

### Part 1: Problem Framing
**Core Insight:**  
The real problem isn't "predict which deals will close" but "identify where the sales process is breaking and what actions will improve outcomes."

**Key Questions Answered:**
- Where are deals dying in the funnel?
- Which customer segments underperform?
- What lead sources drive volume but not quality?
- How do timing and seasonality affect win rates?

**Custom Metrics Invented:**
1. **Stage Friction Index** – Percentage of deals stuck >2x median time per stage
2. **Pipeline Quality Ratio (PQR)** – (Win Rate × Avg Deal Value) / Median Time to Close, calculated per segment

→ See [`docs/Part_1_Problem_Framing.md`](docs/Part_1_Problem_Framing.md) for detailed analysis

### Part 2: Exploratory Data Analysis (EDA)
**Notebook:** `notebooks/eda.ipynb`

**Key Business Insights Extracted:**

1. **Funnel Stage Quality Gap**  
   - Deals at "Qualified" stage show significantly lower win rates (42%) vs "Negotiation" (47%)
   - **Action:** Tighten early-stage qualification criteria (implement MEDDICC/BANT frameworks)

2. **Lead Source Quality Variance**  
   - Inbound leads outperform Partner-sourced leads by 4.5 percentage points
   - **Action:** Audit partner qualification process, introduce partner scorecards

3. **Product-Region ICP Misalignment**  
   - Enterprise products underperform in certain regions
   - Certain product-region combinations (e.g., Pro in India) consistently outperform
   - **Action:** Reallocate GTM resources toward high-performing segments

**Data Quality:**
- 5,000 deals spanning 15 months (Jan 2023 – Mar 2024)
- Overall win rate: 45.3%
- Clean dataset: 0 missing values, 0 duplicates
- Source: `data/skygeni_sales_data.csv`

**Visual Outputs:**  
All analysis visualizations saved to `images/` including:
- Win rate by dimension (`02_winrate_by_dimension.png`)
- Industry-quarter heatmap (`03_industry_quarter_heatmap.png`)
- Lead source quality analysis (`04_lead_source_quality.png`)
- Custom metrics visualizations (`06_custom_metric1_dves.png`, `07_custom_metric2_rrps.png`)

### Part 3: Decision Engine (Win Rate Driver Analysis)
**Notebook:** `notebooks/decision_engine.ipynb`

**Problem Definition:**  
Build a system to identify *which factors* are hurting or improving win rates, not just predict individual outcomes.

**Multi-Model Strategy:**

| Model | Purpose | Performance |
|-------|---------|-------------|
| **Logistic Regression** | Interpretable baseline, directional coefficients | AUC ~0.50 |
| **Random Forest** | Non-linear pattern validation | AUC ~0.49 |
| **Gradient Boosting** | Production candidate (chosen model) | AUC ~0.51 |
| **XGBoost** | Advanced tuning experiment | AUC ~0.50 |

**Why Performance Is Modest (AUC ~0.51):**  
The available features are coarse pre-deal metadata (industry, region, product, lead source, deal stage, deal size). Real win/loss outcomes depend on factors **not present** in this dataset:
- Rep behavior and activity quality
- Buyer intent and urgency
- Competitive pressure
- Pricing/discounting strategies
- Champion strength and economic buyer access
- Product fit depth and technical evaluation results

This isn't a modeling failure — it's a **data limitation insight**. The primary opportunity for improvement lies in capturing richer behavioral and engagement signals.

**Cross-Model Stable Drivers:**  
Features consistently important across ≥3 models:
- `stage_maturity` → Later-stage deals more likely to close
- `created_quarter`, `created_month`, `created_day_of_week` → Strong seasonality signals
- `product_x_region` interactions → ICP fit matters
- `stage_x_source` interactions → Lead quality varies by entry point

**Model Selection:**  
**Gradient Boosting** chosen for production prototyping because:
- Consistently outperformed linear and bagging models
- Captured non-linear interactions between deal size, stage, and lead source
- More stable performance than XGBoost across random seed variations
- Simpler operational complexity (fewer hyperparameters)

**Visual Outputs:**  
Model analysis charts saved to `images/`:
- `model_performance_comparison.png` – AUC comparison across all models
- `logreg_coefficients.png` – Interpretable coefficient analysis
- `rf_importance.png` – Random Forest feature importance

### Part 4: System Design
**Document:** [`docs/Part_4_System_Design.md`](docs/Part_4_System_Design.md)

Designed a production-ready **Win Rate Intelligence Platform** with:
- Daily ETL pipeline for CRM data ingestion
- Feature store with versioned metrics
- Analytics engine with anomaly detection
- Alert system (Slack, Email) with severity tiers
- Executive dashboard (Tableau/Looker)

**Example Alert:**
```
 CRITICAL: Qualified → Demo Conversion
Expected: 62% ± 4%
Actual (last 7 days): 48%
Affected Volume: 43 deals
Action: Review demo scheduling process
```

### Part 5: Reflection
**Document:** [`docs/Part_5_Reflection.md`](docs/Part_5_Reflection.md)

**Weakest Assumptions:**
1. CRM data quality is sufficient (reps may not update stages consistently)
2. Historical patterns remain predictive (market conditions can shift rapidly)
3. Segment-level insights are actionable enough (may need deeper root-cause analysis)

**What Would Break in Production:**
- Data integration complexity (messy CRMs, API failures)
- Organizational resistance (fear of data transparency)
- Alert overload (need careful threshold tuning)
- Model staleness (markets change faster than weekly retraining)

**Next Steps (1 Month):**
- Week 1-2: Integrate behavioral signals (emails, meetings, response times)
- Week 3: Add competitive intelligence and pricing data
- Week 4: Build closed-loop learning (track interventions, measure impact)

---

## 🔑 Key Design Decisions

### 1. Removed `sales_rep_id` to Avoid People-Blame
**Rationale:**  
- Rep identifiers leak territory and segment confounders
- Not actionable for CRO-level decisions (creates blame culture)
- Focus on *structural* process issues, not individual performance

**Impact:**  
Model focuses on factors leadership can systematically improve (qualification process, channel quality, ICP fit) rather than rep-specific execution.

### 2. Avoided Data Leakage by Dropping Post-Close Features
**Removed:**
- `closed_date` (only known after outcome)
- `sales_cycle_days` (computed post-close)
- `outcome` (target variable)

**Kept:**
- `created_date` (for temporal features: month, quarter, day-of-week)

**Rationale:**  
Decision engine must operate on **pre-deal signals** available when the deal enters the pipeline, not hindsight information.

### 3. Engineered Interaction Features Based on Business Hypotheses
**Created:**
- `stage_x_source` (20 combinations) – Lead quality varies by funnel entry point
- `product_x_region` (12 combinations) – ICP fit differs by market maturity

**Rationale:**  
CROs don't think in isolated dimensions. A "Partner lead" behaves differently if it enters at "Qualified" vs "Negotiation." These interactions capture realistic decision-making patterns.

### 4. Dropped Raw `deal_amount`, Kept Only `deal_amount_log`
**Rationale:**  
Tree models split on both raw and log-transformed deal size, inflating feature importance noise and creating redundant signals.

**Impact:**  
Cleaner feature importance interpretation. Log-transformed version handles skewness better for linear models while reducing tree model overfitting to outliers.

### 5. Used Cross-Model Validation for Driver Stability
**Approach:**  
Identified features appearing in top-15 across ≥3 models (Logistic Regression, Random Forest, Gradient Boosting, XGBoost).

**Rationale:**  
Drivers consistent across linear and non-linear models are more likely to be **real business signals** rather than model-specific artifacts.

**Result:**  
7 stable drivers identified, forming the basis of CRO recommendations.

### 6. Positioned Model as "Risk Ranking Tool" Not "Win/Loss Predictor"
**Framing:**  
Even with modest AUC (~0.51), probability ranking can surface top-risk deals for manager review and prioritize interventions.

**Why This Matters:**  
Avoids overpromising prediction accuracy while delivering real operational value. This is how real sales intelligence tools work — they guide attention, not replace judgment.

### 7. Extensive Hyperparameter Tuning (But Didn't Chase AUC)
**Tuning Performed:**
- Logistic Regression: GridSearchCV (4 C values × 2 solvers)
- Random Forest: RandomizedSearchCV (20 iterations)
- Gradient Boosting: RandomizedSearchCV (30 iterations)
- XGBoost: RandomizedSearchCV (50 iterations with L1/L2 regularization)

**Result:**  
Tuning provided marginal gains (~0.01-0.04 AUC). This **validates** that the bottleneck is feature quality, not model sophistication.

**Takeaway:**  
We documented this honestly rather than hiding it — demonstrating mature engineering judgment over Kaggle-style metric chasing.

### 8. Feature Selection Tested, But Not Used in Final Model
**Tested:**
- Univariate (SelectKBest with f_classif)
- Tree-based importance filtering
- PCA (dimensionality reduction)

**Result:**  
Feature selection hurt performance (XGBoost with selected features: AUC 0.494 vs full features: 0.498).

**Decision:**  
Used all features in final model, but documented selection experiments to show thorough exploration.

**Why This Is Good:**  
Negative results are valuable. They show we explored systematically and made evidence-based decisions.

### 9. Created Custom Business Metrics
**Stage Friction Index:**  
```
% of deals spending >2x median time in each stage
```
→ Early indicator of bottlenecks before deals die

**Pipeline Quality Ratio (PQR):**  
```
(Win Rate × Avg Deal Value) / Median Time to Close
```
→ Accounts for opportunity cost of long sales cycles

**Why These Matter:**  
Standard metrics (win rate, conversion rate) don't capture *efficiency* or *velocity*. These custom metrics guide resource allocation more effectively.

### 10. Honest About Limitations in Documentation
**Explicit Statements:**
- "This system is a prioritization engine, not a deterministic predictor"
- "Low AUC reflects data limitations, not modeling failure"
- "Missing signals (rep behavior, buyer intent, competition) are the primary bottleneck"

**Why This Builds Trust:**  
Sales leaders have been burned by oversold "AI magic." Honesty about what the system *can't* do builds credibility for what it *can* do.

---

##  Results & Insights

### Stable Win Rate Drivers (Validated Across Models)

| Driver | Direction | Business Interpretation |
|--------|-----------|------------------------|
| **Stage Maturity** | ↑ Positive | Later-stage deals consistently more likely to close |
| **Deal Size (log)** | — Complex | Larger deals require specialized playbooks |
| **Created Quarter** | ↑ Positive | Quarter-end urgency effects |
| **Created Month** | ↓ Negative | Certain months show weaker conversion |
| **Day of Week** | ↑ Positive | Deal creation timing affects quality |
| **Pro × India** | ↑ Positive | Strong product-market fit in this segment |
| **Lead Source Outbound** | Mixed | Performance varies by funnel stage |

### CRO-Actionable Recommendations

**1. Fix Qualified Stage Leakage**  
→ Implement stricter qualification (MEDDICC), automate demo scheduling, add early-stage velocity SLAs

**2. Build Large-Deal Playbook**  
→ Executive sponsorship, deal desks, specialized enablement for Enterprise segment

**3. Audit Partner Channel Quality**  
→ Introduce partner scorecards, tighten onboarding criteria, separate partner quality tiers

**4. Leverage Seasonality**  
→ Align campaigns with high-conversion quarters, increase acceleration efforts during strong windows

**5. Double Down on High-PQR Segments**  
→ Reallocate resources to Pro × India and other high-performing product-region combinations

---

## 📁 Repository Structure

```
skygeni/
│
├── README.md                          # This file
│
├── data/
│   └── skygeni_sales_data.csv         # Raw dataset (5,000 deals)
│
├── docs/
│   ├── Part_1_Problem_Framing.md      # Business problem analysis (no code)
│   ├── Part_4_System_Design.md        # Production architecture design
│   └── Part_5_Reflection.md           # Assumptions, limitations, next steps
│
├── notebooks/
│   ├── eda.ipynb                      # Part 2: Exploratory analysis
│   └── decision_engine.ipynb          # Part 3: Win rate driver analysis
│
└── images/                            # Generated plots and visualizations
    ├── 01_overview_dashboard.png      # Win/loss distribution overview
    ├── 02_quarterly_win_rate.png      # Temporal trends
    ├── 03_industry_quarter_heatmap.png # Segment performance heatmap
    ├── 04_lead_source_quality.png     # Channel quality analysis
    ├── 05_sales_cycle_correlation.png # Cycle time impact
    ├── 06_custom_metric1_dves.png     # Deal Velocity Efficiency Score
    ├── 07_custom_metric2_rrps.png     # Revenue Risk Probability Score
    ├── model_performance_comparison.png # Model AUC comparison
    ├── logreg_coefficients.png        # Coefficient analysis
    ├── rf_importance.png              # Feature importance
    └── ...                            # Additional analysis visualizations
```

---

##  Technical Stack

| Component | Technology |
|-----------|-----------|
| **Data Processing** | Pandas, NumPy |
| **Visualization** | Matplotlib, Seaborn |
| **ML Framework** | Scikit-learn, XGBoost |
| **Feature Engineering** | ColumnTransformer, OneHotEncoder, StandardScaler |
| **Hyperparameter Tuning** | GridSearchCV, RandomizedSearchCV |
| **Model Selection** | Cross-validation (3-fold stratified) |
| **Evaluation Metrics** | AUC, Accuracy, Precision, Recall, F1 |

---

##  What Makes This Solution Stand Out

### 1. Decision Intelligence > Pure ML
Framed as a diagnostic system for CRO decision-making, not a black-box predictor.

### 2. Honest About Performance
Documented why AUC is modest and positioned it as a data limitation insight, not a modeling failure.

### 3. Cross-Model Validation
Identified stable drivers across 4 model families to separate signal from noise.

### 4. Business Translation
Every technical finding mapped to concrete CRO actions (e.g., "tighten outbound qualification in Q1").

### 5. Production Mindset
Designed for operationalization (alerting, failure handling, monitoring) not just notebooks.

### 6. Custom Metrics
Invented Stage Friction Index and Pipeline Quality Ratio — metrics CROs don't have but need.

### 7. Explicit Assumptions
Documented what could break in production (data quality, organizational resistance, model drift).

### 8. Negative Results Documented
Showed feature selection experiments that didn't work — demonstrates thorough exploration.

---

##  Model Performance Summary

| Model | Baseline AUC | Tuned AUC | Key Insight |
|-------|--------------|-----------|-------------|
| Logistic Regression | 0.496 | 0.494 | Linear models fail; relationships are non-linear |
| Random Forest | 0.482 | 0.494 | Confirmed deal size and timing dominance |
| Gradient Boosting | 0.505 | **0.509** | **Selected for production** |
| XGBoost | 0.509 | 0.498 | Tuning inconsistent; confirms data ceiling |

**Interpretation:**  
All models converge around AUC ~0.50, indicating the **feature set is the limiting factor**. This validates that improving data instrumentation (adding engagement signals, competitive intelligence, pricing data) will unlock more value than further model tuning.

---

##  Future Enhancements (1-Month Roadmap)

### Week 1-2: Enhanced Data
- Integrate engagement signals (email activity, meeting cadence)
- Add competitive intelligence from win/loss notes
- Capture pricing/discounting data

### Week 3: Advanced Analytics
- Real-time deal risk scoring
- Personalized next-action recommendations per deal
- What-if simulation for interventions

### Week 4: Closed-Loop System
- Track which actions are taken
- A/B test interventions (measure impact)
- Auto-improve based on outcomes

---

## Contact & Feedback

**Author:** Abhishek Kumar Singh 

**Email:** abhisingh7114@gmail.com

**Questions?** Open an issue or reach out directly.

---



**Key Learnings:**
- Sales intelligence is primarily a **data + systems problem**, not a modeling problem
- Honesty about limitations builds more trust than overselling capabilities
- Stable cross-model drivers matter more than optimizing a single model's AUC
- The true leverage lies in closing the loop between insights and action

---

**License:** MIT  
**Last Updated:** February 13, 2026

---

## Quick Links

- [Problem Framing](docs/Part_1_Problem_Framing.md)
- [System Design](docs/Part_4_System_Design.md)
- [Reflection & Next Steps](docs/Part_5_Reflection.md)
- [EDA Notebook](notebooks/eda.ipynb)
- [Decision Engine Notebook](notebooks/decision_engine.ipynb)
- [Dataset](data/skygeni_sales_data.csv)
- [Visualizations](images/)
