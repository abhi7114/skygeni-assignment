# Part 1 – Problem Framing

## 1. What is the Real Business Problem?

### The Surface Symptom:
"Win rate has dropped over the last two quarters, but pipeline volume looks healthy."

### The Real Business Problem:
This is a **revenue engine efficiency crisis** disguised as a volume success. The company is generating sufficient top-of-funnel activity, but deals are either:

- **Dying at specific funnel stages** (leakage)
- **Converting at lower rates** due to quality deterioration
- **Taking longer to close** (velocity degradation)
- **Being pursued in wrong segments** (ICP drift)

The core issue is **lack of diagnostic visibility**. The CRO cannot distinguish between:

- **Execution problems** (reps struggling at specific stages)
- **Process problems** (qualification criteria too loose)
- **Market problems** (competitive pressure, pricing misalignment)
- **Segment problems** (wrong industries/regions/products being prioritized)
- **Behavioral problems** (rep activity quality declining)

This creates **decision paralysis** — leadership doesn't know where to invest time and resources for maximum impact.

---

## 2. What Key Questions Should an AI System Answer?

The system should help the CRO answer these critical questions:

### A. Funnel Diagnostics

**Where are deals dying?**
- Which stage has the largest drop-off rate?
- Has stage-to-stage conversion velocity changed over time?
- Are deals getting stuck longer at specific stages?

**Is this a qualification problem or an execution problem?**
- Are early-stage deals (Qualified) progressing at normal rates?
- Is the quality of deals entering the funnel deteriorating?

### B. Segment-Level Risk

**Which customer segments are underperforming?**
- Win rate by industry vertical
- Win rate by region
- Win rate by deal size tier
- Win rate by product type

**Are certain GTM motions failing?**
- Lead source performance (Inbound vs Outbound vs Partner vs Referral)
- Product mix shifts impacting overall win rate

### C. Root Cause Isolation

**Is this a people problem, a process problem, or a market problem?**
- Are certain reps/teams underperforming (while avoiding blame culture)?
- Are there systematic process breakdowns?
- Has market competitiveness changed?

**What are the leading indicators of decline?**
- Early-stage volume quality deteriorating?
- Deal velocity slowing before win rate drops?
- Mix shift toward harder-to-close segments?

### D. Action Prioritization

**Where should leadership focus next quarter?**
- Which stages need enablement investment?
- Which segments should get more/less focus?
- Which lead sources need optimization?

**What's the expected impact of interventions?**
- If we fix stage X conversion by Y%, what's the revenue impact?
- If we reallocate from segment A to segment B, what's the upside?

---

## 3. What Metrics Matter Most for Diagnosing Win Rate Issues?

### Core Funnel Metrics

| Metric | Why It Matters | What It Reveals |
|--------|----------------|-----------------|
| Stage-to-Stage Conversion Rates | Shows exactly where deals die | Process or execution bottlenecks |
| Stage Velocity (Days per Stage) | Detects slowdowns before they become losses | Early warning of problems |
| Win Rate by Entry Stage | Tests if qualification is working | Quality of pipeline entering funnel |

### Segmented Performance Metrics

| Metric | Why It Matters | What It Reveals |
|--------|----------------|-----------------|
| Win Rate by Region | Geographic GTM health | Market fit and execution by geo |
| Win Rate by Industry | Vertical-specific product fit | ICP alignment |
| Win Rate by Product Type | SKU-level conversion | Pricing/packaging/complexity issues |
| Win Rate by Lead Source | Channel quality | Marketing and partner effectiveness |
| Win Rate by Deal Size Tier | Complexity and qualification rigor | Enterprise vs SMB motion quality |

### Custom Diagnostic Metrics (Novel Contributions)

#### 1. Stage Friction Index

**Formula:**
```
% of deals spending >2x median time in a given stage
```

**Why it matters:**
- Highlights where deals get "stuck" before they die
- Early indicator of future losses
- Pinpoints specific process/enablement gaps

**Example Action:**
If 40% of deals spend >2x median time at "Proposal," it suggests:
- Proposal quality/approval process issue
- Pricing negotiation friction
- Lack of decision-maker access

**CRO Action:** Investigate proposal review process, add deal coaching for stalled proposals

#### 2. Pipeline Quality Ratio (PQR)

**Formula:**
```
PQR = (Win Rate × Average Deal Value) / Median Time to Close
```
Calculated per segment (region × product × lead source)

**Why it matters:**
- Combines conversion, value, AND velocity
- Accounts for opportunity cost of long sales cycles
- Helps prioritize GTM resource allocation

**Example Insight:**
- Segment A: 60% win rate, $50K ACV, 90 days → PQR = 33.3
- Segment B: 40% win rate, $100K ACV, 120 days → PQR = 33.3
- → Despite lower win rate, Segment B delivers equal efficiency

**CRO Action:** Allocate resources based on PQR, not just win rate or deal size alone

### Temporal/Trend Metrics

| Metric | Why It Matters |
|--------|----------------|
| Win Rate Δ (QoQ, MoM) | Trajectory, not just point-in-time |
| Stage Conversion Volatility | Process consistency |
| Weighted Pipeline Value at Risk | Revenue impact forecasting |
| Lead Source Mix Shift | Early detection of channel quality changes |

### Comparative Benchmarks

| Metric | Why It Matters |
|--------|----------------|
| Win Rate vs Historical Baseline | Magnitude of degradation |
| Segment Performance vs Company Avg | Outlier identification |
| This Quarter vs Same Quarter Last Year | Seasonality adjustment |

---

## 4. What Assumptions Are Being Made?

### Data Quality Assumptions

**CRM data is reasonably accurate and complete**
- Deal stages are updated consistently
- Close dates and outcomes are reliable
- Deal amounts reflect committed ACV

*Risk:* If reps don't maintain CRM hygiene, insights become misleading  
*Mitigation:* Add data quality scoring to the system

**Deal stages follow standardized definitions**
- "Qualified" means the same across all reps/regions
- Stage progression follows expected sequence

*Risk:* Regional or team-specific stage usage variations create noise  
*Mitigation:* Segment analysis by team to detect definition drift

**Historical patterns are directionally predictive**
- Past win rate drivers remain relevant for next quarter
- Market conditions haven't fundamentally shifted

*Risk:* Black swan events (new competitor, pricing change, economic shock)  
*Mitigation:* Include market context monitoring, not just internal metrics

### Business Process Assumptions

**Pipeline volume is directionally meaningful**
- More deals → more opportunity (if quality is stable)

*Risk:* Volume can mask quality erosion  
*Mitigation:* Track "quality-adjusted pipeline" using early-stage progression rates

**Deal outcomes reflect sales execution quality**
- Win/loss is partially controllable by sales team

*Risk:* Some losses are due to external factors (budget cuts, project canceled)  
*Mitigation:* Capture loss reasons in CRM, separate "controllable" from "uncontrollable" losses

**Reps are using stages consistently and honestly**
- Stages aren't being gamed to hit activity metrics

*Risk:* Stage inflation to appear busy  
*Mitigation:* Audit stage dwell times and regression patterns

### Analytical Assumptions

**Correlation ≠ Causation**
- Identified "drivers" are associations, not proven causes
- Hidden confounders may exist (rep skill, territory quality, product fit)

*Risk:* Acting on correlations that aren't causal  
*Mitigation:* A/B test interventions where possible, combine quant with qual

**Missing variables are important**
- Buyer intent, urgency, budget authority not captured in CRM
- Competitive dynamics not visible
- Pricing/discounting often not tracked
- Champion strength, exec sponsor presence missing

*Risk:* System will have limited predictive power  
*Mitigation:* Acknowledge limits, position as diagnostic tool, plan data enrichment

**Segment-level patterns are more stable than individual predictions**
- Patterns across 100 deals are meaningful
- Individual deal scoring is inherently noisy

*Risk:* Over-relying on individual deal scores  
*Mitigation:* Use for prioritization and risk-flagging, not deterministic forecasting

### Organizational Assumptions

**Leadership will act on insights**
- Identified issues will be addressed
- Resources can be reallocated

*Risk:* Analysis paralysis or political blockers  
*Mitigation:* Tie insights to clear, prioritized actions

**System is for decision support, not automation**
- Human judgment remains central
- Insights enhance, not replace, sales intuition

*Risk:* Over-automation leading to misuse  
*Mitigation:* Clear documentation of use cases and limitations

**Changes can be measured**
- Interventions are trackable
- Time-to-impact is reasonable

*Risk:* Too many simultaneous changes blur attribution  
*Mitigation:* Phased rollout, control groups where possible

---

## Summary: Problem Framing Philosophy

This problem is best approached as:

###  Decision Intelligence (not pure ML prediction)
- **Goal:** Help leaders make better decisions
- **Focus:** Diagnostic insights → Recommended actions
- **Success:** Improved processes and outcomes, not model accuracy

###  Process Diagnosis (not people blame)
- Identify systemic patterns
- Avoid rep-level scoring for punishment
- Guide resource allocation and enablement investment

###  Segment-Level Analysis (not individual deal scoring)
- Patterns across cohorts are stable
- Individual deals are inherently noisy
- Focus on "where" not "who"

###  Interpretable & Actionable (not black-box)
- CRO can explain findings to board
- Sales leaders know what to change
- Insights tie to concrete interventions

###  Honest About Limitations (not overselling AI)
- Acknowledge weak predictive signal from metadata alone
- Frame as diagnostic tool, not crystal ball
- Combine with qualitative sales knowledge
- Plan for data enrichment over time

---

## The Real Win
**Moving leadership from "we don't know what's wrong" to "here are the 3 highest-ROI places to intervene."**
