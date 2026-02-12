# Part 4 – System Design: Sales Insight & Alert System

## System Overview

A production-ready **Win Rate Intelligence Platform** that transforms analytical insights into operational decision support for CROs and sales leadership.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       DATA SOURCES                              │
├─────────────────────────────────────────────────────────────────┤
│  • CRM (Salesforce/HubSpot) - Deal data, stages, outcomes       │
│  • Data Warehouse (Snowflake/BigQuery) - Historical analytics   │
│  • Engagement Data (optional) - Emails, calls, meetings         │
└──────────────────┬──────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                  INGESTION & VALIDATION                         │
├─────────────────────────────────────────────────────────────────┤
│  • Daily ETL (Airflow/Prefect)                                  │
│  • Schema validation & type checking                            │
│  • Data quality checks (completeness, consistency)              │
│  • Deduplication & cleansing                                    │
└──────────────────┬──────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                     FEATURE STORE                               │
├─────────────────────────────────────────────────────────────────┤
│  • Deal-level features (stage, size, source, region)            │
│  • Engineered features (stage maturity, deal size log)          │
│  • Aggregated metrics (win rates by segment)                    │
│  • Time-series features (trends, rolling averages)              │
│  • Custom metrics (Stage Friction Index, PQR)                   │
└──────────────────┬──────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────────┐
│               ANALYTICS & DECISION ENGINE                       │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐    │
│  │  Win Rate    │  │   Segment    │  │   Anomaly           │    │
│  │  Driver      │  │   Performance│  │   Detection         │    │
│  │  Analysis    │  │   Scoring    │  │                     │    │
│  └──────────────┘  └──────────────┘  └─────────────────────┘    │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────────┐    │
│  │  Stage       │  │   Trend      │  │   Alert             │    │
│  │  Conversion  │  │   Detection  │  │   Engine            │    │ 
│  │  Analysis    │  │              │  │                     │    │
│  └──────────────┘  └──────────────┘  └─────────────────────┘    │
└──────────────────┬──────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                      INSIGHT API                                │
├─────────────────────────────────────────────────────────────────┤
│  • REST API endpoints                                           │
│  • Authentication & rate limiting                               │
│  • Caching layer (Redis)                                        │
│  • Versioned endpoints                                          │
└──────────────────┬──────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                  DELIVERY & INTERFACE                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌────────────┐  ┌──────────────┐   │
│  │Dashboard │  │  Email   │  │   Slack    │  │ Salesforce   │   │
│  │(Tableau/ │  │  Digest  │  │   Alerts   │  │   Widget     │   │
│  │ Looker)  │  │          │  │            │  │              │   │
│  └──────────┘  └──────────┘  └────────────┘  └──────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Detailed Component Design

### 1. Ingestion & Validation Layer

**Purpose:** Ensure clean, consistent data before analysis

**Daily ETL Pipeline:**
- Extract from CRM via API
- Validate schema (required fields, types, nulls)
- Run quality checks (no duplicates, valid date ranges, positive amounts, valid stage values)
- Enrich features
- Load to feature store

**Failure Handling:**
- **On validation failure:** Alert data eng team, use last known good dataset
- **On quality check failure:** Log issues, flag affected deals, continue with clean subset
- **On API timeout:** Exponential backoff retry (3 attempts), then alert

---

### 2. Feature Store

**Technology:** PostgreSQL with versioned feature definitions in Git

**Feature Categories:**

**Base Features:**
- CRM fields: stage, deal_amount, industry, region, product_type, lead_source
- Temporal: created_month, created_quarter, created_year, day_of_week

**Engineered Features:**
- stage_maturity: Ordinal encoding of funnel progression
- deal_amount_log: Log-transformed deal size
- is_month_end: Binary flag for deals created in last week of month
- Interaction features: stage_x_source, product_x_region

**Aggregated Metrics (Updated Weekly):**
- Win rate by: industry, region, product, lead source, stage
- Stage-to-stage conversion rates
- Median time in each stage

**Custom Metrics:**
- Stage Friction Index: % deals stuck >2x median time per stage
- Pipeline Quality Ratio: (Win Rate × Avg Deal Size) / Median Time to Close, per segment

**Rolling Features:**
- 7-day, 30-day, 90-day win rate trends
- QoQ and MoM deltas
- Moving averages for smoothing

---

### 3. Analytics & Decision Engine

#### A. Win Rate Driver Analysis Module

**Function:** Identify factors hurting/improving win rate  
**Model:** Tuned Gradient Boosting  
**Refresh:** Weekly (Sundays at midnight)

**Output Example:**
```json
{
  "analysis_date": "2024-02-18",
  "model_version": "v2.3",
  "drivers": [
    {
      "factor": "stage_maturity",
      "importance": 0.22,
      "direction": "positive",
      "confidence": "high",
      "business_meaning": "Deals further in funnel consistently more likely to close"
    },
    {
      "factor": "lead_source=Outbound, product=Enterprise",
      "importance": 0.08,
      "direction": "negative",
      "confidence": "medium",
      "business_meaning": "Outbound enterprise deals underperform baseline"
    }
  ],
  "model_performance": {
    "note": "Modest AUC reflects data limitations; use for directional insights"
  }
}
```

#### B. Segment Performance Scoring

**Function:** Rank segments by health, flag underperformers

**Methodology:**
- Compute win rate, conversion rate, avg deal size, velocity per segment
- Compare to historical baselines (rolling 6-month avg)
- Flag segments with >10% degradation or significant absolute drop

**Example Output:**
```json
{
  "segment_scores": [
    {
      "segment": "APAC + Enterprise",
      "current_win_rate": 0.38,
      "baseline_win_rate": 0.47,
      "delta": -0.09,
      "volume": 127,
      "status": "critical",
      "recommendation": "Audit qualification & enablement for APAC Enterprise"
    }
  ]
}
```

#### C. Anomaly Detection Module

**Signals Monitored:**
- Sudden win rate drops (>5% WoW)
- Stage conversion rate changes (>10% from baseline)
- Volume spikes without proportional wins
- Deal velocity degradation

**Method:**
- Statistical process control (3-sigma bands)
- Time-series decomposition (trend vs seasonality)

**Example Alert:**
```
⚠️ ANOMALY: Qualified → Demo Conversion
Expected: 62% ± 4%
Actual (last 7 days): 48%
Affected Volume: 43 deals
Severity: Warning
```

#### D. Trend Detection Module

**Metrics Tracked:**
- Win rate trajectory (improving/declining/stable)
- Stage velocity trends
- Deal size mix shifts

**Alert Logic:**
- 2 consecutive weeks decline → "Watch"
- 3 consecutive weeks + statistical significance → "Warning"
- 4 consecutive weeks → "Critical"

---

### 4. Insight API

**Technology:** REST API

**Key Endpoints:**
- `GET /api/v1/win-rate-drivers` → Returns current driver analysis
- `GET /api/v1/segment-performance` → Returns specific segment metrics
- `GET /api/v1/alerts` → Returns recent alerts
- `GET /api/v1/dashboard-summary` → Returns executive dashboard data
- `POST /api/v1/refresh-analysis` → Triggers on-demand recomputation

**Authentication:** OAuth 2.0  
**Rate Limiting:** 100 requests/min per user  
**Caching:** 5-min TTL on dashboard queries

---

### 5. Delivery & Interface Layer

#### A. Executive Dashboard (Tableau/Looker)

**Primary View:**
- Current win rate KPI (vs baseline, vs last quarter)
- Stage-by-stage funnel with conversion rates
- Top 3 risk drivers and top 3 opportunities
- Segment performance heatmap (region × product)

**Drill-Downs:**
- Win rate by lead source (stacked area chart)
- Stage Friction Index by stage (bar chart)
- Pipeline Quality Ratio by segment (scatter plot)

**Alert Panel:**
- Recent critical/warning alerts
- Recommended actions

**Refresh:** Real-time (pulls from API)

#### B. Weekly Email Digest

**Recipients:** CRO, VP Sales, Sales Ops

**Content Example:**
```
📊 SkyGeni Weekly Win Rate Intelligence – Feb 18, 2024

SUMMARY
Win Rate: 43.2% (▼ 2.1% WoW, ▼ 4.8% QoQ)
Pipeline Volume: 284 deals (▲ 12% WoW)
⚠️ Volume up, conversion down — quality issue suspected

TOP DRIVERS THIS WEEK
🟢 Deals reaching Negotiation +4.2% vs baseline
🔴 Outbound + Enterprise -6.7% vs baseline
🔴 Qualified stage friction increased (48% stuck >7 days)

RECOMMENDED ACTIONS
1. Audit outbound enterprise qualification process
2. Add stage conversion SLAs for Qualified → Demo
3. Review enablement for mid-funnel stages

FULL DASHBOARD: [Link]
```

#### C. Slack Alerts

**Channels:**
- `#sales-intelligence-critical` → Critical alerts
- `#sales-ops` → Warnings and trends

**Format:**
```
🚨 CRITICAL ALERT – Win Rate Drop Detected

Segment: Enterprise + Europe
Win Rate: 31% (was 48% last month)
Affected Deals: 23 losses this month

Root Cause (AI Analysis):
Stage leakage at Proposal → Negotiation
Possible factors: Pricing friction, decision-maker access

Recommended Action:
- Review pricing approval process for Europe Enterprise
- Add deal coaching for stalled proposals

View Details: [Dashboard Link]
Acknowledge: [Button]
```

---

## Data Flow (End-to-End)

### Daily Pipeline
```
06:00 UTC  ETL pulls closed deals from CRM
06:15 UTC  Validation & cleansing
06:30 UTC  Feature computation
07:00 UTC  Aggregated metrics update
07:30 UTC  Anomaly detection runs
07:45 UTC  Critical alerts sent (if triggered)
08:00 UTC  Dashboard cache refresh
```

### Weekly Pipeline
```
Sunday 00:00  Win rate driver model retraining
Sunday 01:00  Segment performance scoring
Sunday 02:00  Trend analysis update
Sunday 03:00  Weekly digest generation
Monday 08:00  Email digest sent
```

### On-Demand
```
User requests dashboard → API pulls from cache (5-min TTL)
→ If cache miss, compute and cache → Return to user
```

---

## Example Alerts & Insights

### Alert 1: Stage Leakage
```
📉 Stage Conversion Alert – Qualified → Demo

Current: 52% (was 68% last month)
Volume: 89 deals stuck at Qualified
Impact: ~$1.2M ARR at risk

Potential Causes:
- Demo scheduling bottleneck
- Qualification criteria too loose
- Seasonal buyer unavailability

Recommended Actions:
1. Automate demo scheduling
2. Review MEDDICC qualification
3. Add nurture sequence for delayed demos
```

### Alert 2: Channel Quality
```
⚠️ Lead Source Quality Degradation – Partner

Partner Lead Win Rate: 28% (down from 42%)
Volume: 127 deals this quarter
Lost Revenue: ~$800K ARR

Analysis:
- New partner onboarded in Jan with 18% win rate
- Existing partners stable at 44%

Recommended Actions:
1. Implement partner scorecard
2. Tighten new partner qualification
3. Create partner enablement program
```

---

## Execution Frequency

| Component | Frequency | Rationale |
|-----------|-----------|-----------|
| Data Ingestion | Daily (6 AM UTC) | Fresh data for ops |
| Feature Computation | Daily | Support real-time dashboard |
| Win Rate Driver Analysis | Weekly (Sunday) | Stable patterns need time |
| Segment Scoring | Weekly | Enough volume for significance |
| Anomaly Detection | Daily | Early warning critical |
| Critical Alerts | Real-time | Immediate action needed |
| Email Digest | Weekly (Mon 8 AM) | Manageable cadence |
| Dashboard Refresh | On-demand + cache | Balance freshness/performance |

---

## Failure Cases & Limitations

### Data Quality Failures
**Issue:** CRM data incomplete/inconsistent  
**Impact:** Misleading insights, false alerts  
**Mitigation:**
- Strict validation rules
- Fallback to last known good
- Data quality dashboard
- Monthly CRM hygiene audits

### Statistical Limitations
**Issue:** Low sample sizes in segments  
**Impact:** Unstable metrics, high variance  
**Mitigation:**
- Minimum n=30 threshold for reporting
- Confidence intervals on metrics
- Suppress alerts if sample too small
- Combine segments when appropriate

### Model Drift
**Issue:** Win rate drivers change over time  
**Impact:** Stale insights  
**Mitigation:**
- Weekly model retraining
- Performance monitoring
- Quarterly model review
- A/B test new versions

### Alert Fatigue
**Issue:** Too many alerts → ignored  
**Impact:** Critical signals missed  
**Mitigation:**
- Strict severity thresholds
- Daily digest for non-critical
- User-configurable preferences
- Snooze/resolve options

### Integration Failures
**Issue:** CRM API downtime  
**Impact:** Stale data  
**Mitigation:**
- Retry with exponential backoff
- Graceful degradation (cached data)
- ETL failure monitoring
- Manual override capability

---

## Production Readiness Checklist

-  Data Pipeline: Automated, monitored, validated
-  Feature Store: Versioned, tested, documented
-  Models: Cross-validated, tracked, explainable
-  API: Authenticated, rate-limited, cached
-  Alerts: Severity-tiered, routed, actionable
-  Dashboard: Responsive, accessible
-  Monitoring: System health, data quality, drift
-  Documentation: API docs, runbooks, user guides
-  Security: Encryption, access controls, audit logs
-  Testing: Unit, integration, UAT

---

## Future Enhancements (1-Month Roadmap)

### Phase 1 (Weeks 1-2): Enhanced Data
- Integrate engagement signals (emails, calls, meetings)
- Add competitive intelligence (win/loss notes)
- Capture pricing/discounting data
- Connect calendar/email for activity tracking

### Phase 2 (Weeks 3-4): Advanced Analytics
- Deal Risk Scoring: Real-time loss probability
- Recommended Actions: Personalized next steps per deal
- Revenue Forecasting: Predictive pipeline value
- What-If Simulation: Impact modeling for interventions

### Phase 3 (Month 2): Closed-Loop System
- Intervention Tracking: Log when actions taken
- A/B Testing: Measure impact of changes
- Feedback Collection: Rate insight quality
- Automated Learning: System improves based on outcomes

---

## Summary

This system design balances:
- **Analytical rigor** (robust driver analysis)
- **Operational reality** (daily workflows, failure handling)
- **User experience** (dashboards, alerts, digests)
- **Production readiness** (scalability, monitoring, security)

It's designed to be:
-  **Actionable:** Insights → Clear next steps
-  **Scalable:** Handles growth in data/users
-  **Reliable:** Graceful failure handling
-  **Interpretable:** Non-technical stakeholders understand
-  **Iterative:** Improves based on feedback

---

## The Real Value
**Shifting CROs from reactive firefighting to proactive risk management.**
