# Well Health Monitor
Is this well dying or just resting? A statistical investigation of production decline across 894 Bakken oil wells.

![](https://github.com/oluwagbemiga01/Well-Health-Monitor/blob/main/Well%20Health%20dashboard.png)

---
## Agenda
1. Problem Statement
2. Data Overview
3. Analysis & Findings
4. Hypothesis Testing
5. Recommendations

---
## Problem Statement
An operations team needs to know which oil wells are in genuine long term decline versus temporary underperformance, so they can prioritise intervention resources correctly. Getting this wrong costs money in both directions: wasting resources fixing a well that is fine, or ignoring a well that needed urgent attention.

## Four specific questions this analysis sets out to answer:
1. Which wells show statistically significant production decline over 12 months?
2. At current decline rates, which wells will drop below the economic threshold within 6 months?
3. Is there a statistically significant difference in decline rates between oil fields?
4. What is the 90% confidence interval for production in the next quarter?

---

## Data Overview
**Source**: Real Bakken formation production data - North Dakota OIl & Gas Division

| Attribute      | Detail           |
|:--------------:|:----------------:|
|Raw dataset size| 26,041 rows      |
|Wells in raw data| 894             |
|Date range      |Jan 2016-Jan 2019 |
|Fiels covered   |209 oil fields    |
|Companies       |92 operators      |

**After cleaning and filtering:**

| Attribute                       | Detail           |
|:-------------------------------:|:----------------:|
|Active wells (12+ months)        | 748              |
|Wells after reliability filter   | 633              |
|Final wells analysed             | 532(in fields with 3+ members)|
|Shut in months flagges           | 3,318 (Days Produces < 15)|

**Cleaning decisions:**
- Months where the well produced fewer than 15 days were flagged as shut-in and excluded from curve fitting
- Wells with fewer than 12 active months were excluded (insufficient data for reliable curve fitting)
- Wells where curve fitting returned R² < 0.1 or D < 0.001 were excluded as unreliable fits

**Four analytical datasets derived from the single raw file:**

| Dataset                         | Rows           | Purpose             |
|:-------------------------------:|:----------------:|:----------------:|
| Master Results                  | 532            | Fitted decline parameters per well |
| Forecast Results                | 532            | 6-month production projections |
| Confidence Intervals            | 532            | 90% CI bounds for next quarter |
| Recovery Analysis               | 801            | Post shutdown recovery events |

---

## Analysis & Findings

**Decline Curve Fitting**
Two models were fitted to every well using scipy.optimize.curve_fit:

**Exponential Decline:**
q(t) = q₀ × e^(−D×t)

**Hyperbolic Decline:**
q(t) = q₀ / (1 + b×D×t)^(1/b)
The best fitting model per well was selected based on R² goodness of fit.

| Model       | Wells  | Percentage |
|:-----------:|:------:|:-----------|
|Hyperbolic   | 491    | 65.6%      |
|Exponential  | 257    | 34.4%      |

Hyperbolic decline dominates which is consistent with known **Bakken shale** well behaviour where decline rate slows over time as natural fractures help maintain reservoir pressure.

**Decline Rate Summary (Best Model per Well):**
| Metric | Value  |
|:------:|:------:|
|Mean D  | 0.148  |
|Median D| 0.095  |
|25th percentile | 0.047 |
|75th percentile | 0.204 |

## Production Forecasting
Economic threshold set at **300 barrels/month** - this is the minimum viable production rate for onshore US wells.

| Risk Category          | Wells  | Percentage |
|:----------------------:|:------:|:-----------|
|At risk within 6 months | 49     | 9.2%       |
|Critical within 3 months| 40     | 7.5%       |

**Most urgent well:** HOMER FEDERAL 14X-32F — currently producing 2,567 bbl/month but forecast to crash to 150 bbl/month by month 3 due to a D value of 0.196.

---

## Confidence Intervals (90%)
| Metric                   | Value          |
|:------------------------:|:--------------:|
|Median point forecast     | 1,260 bbl/month|
|Median CI lower bound     | 200 bbl/month  |
|Median CI upper bound     | 2,284 bbl/month|
|Median CI width           | 1,819 bbl/month|

Wide confidence intervals reflect genuine uncertainty in individual well behaviour, i think this is a realistic and honest result for shale wells.

---

## Field Comparison (ANOVA)
One way ANOVA tested whether decline rates differ significantly across 45 fields (a minimum of 5 wells per field)

| Metric                   | Value          |
|:------------------------:|:--------------:|
| F-statistic              | 4.2487         |
| P-value                  | ≈ 0.000000     |
| Significant field pairs (Tukey HSD) | 52 |

**Result:** There is zero probability that the field differences are due to random chance.

| Field          | Mean  | Classification |
|:--------------:|:-----:|:---------------|
| SAND CREEK     |0.605  | Critical       |
| ALKALI CREEK   |0.338  | Critical       |
| ANTELOPE       |0.305  | Critical       |
| WEST AMBROSE   |0.050  | Normal         |
|REUNION BAY     |0.024  | Normal         |

SAND CREEK declines **25x faster** than REUNION BAY

---

## Correlation Analysis (Downtime vs Production)
**980 downtime to recovery transitions** were identified across all wells. After cleaning extreme outliers, **801 events** were analysed.

**Recovery breakdown:**

| Category          | Count  | Percentage |
|:-----------------:|:------:|:-----------|
|Full recovery(>90%)| 590    | 73.7%      |
|Partial recovery(50-90%) | 150 | 18.7%   |
|Severe damage(<50%)| 61     | 7.6%       |

**Correlation results:**

| Test                                    | r      | P-value    | Significant |
|:---------------------------------------:|:------:|:-----------|:------------|
|Downtime rate vs Decline rate (Pearson)  | -0.065 | 0.135      | No          |
|Downtime rate vs Decline rate (Spearman) | -0.010 | 0.822      | No          |
|Downtime rate vs Recovery ratio (Pearson)| 0.191  | 0.000091   | Yes         | 
|Downtime rate vs Recovery ratio (Spearman)| 0.225 | 0.000004   |  Yes        |

**Key Finding:** Downtime does not accelerate long term decline, but frequent downtime is significantly linked to recovery quality. Wells with regular planned maintenance shutdowns tend to recover better than wells with infrequent unplanned outages.

---

## Hypothesis Testing
**Question 1: Which wells show statistically significant decline?**

**Method:** Z-score test per well against field average decline rate.

H₀: This well's decline rate is not different from the field average.

H₁: This well's decline rate is genuinely faster than the field average.

Threshold: p < 0.05 (two-tailed)

**Result:** 26 wells (4.9%) show statistically significant decline.

**Top 3 most significant wells:**

| Well                                    | Field            | D    | Z-Score | P-Value |
|:---------------------------------------:|:----------------:|:-----|:--------|:--------|
|AMUNDSON 44-22NWH                        | SIVERSTON        |0.933 |3.715| 0.0002|
|SNOWY 147-94-13A-24H TF                  | MCGREGORY BUTTES |0.873 | 2.980 | 0.0029|
|EN-CVANCARA-LE-155-93-1522H-1            | ALGER            |0.753 |3.182 | 0.0015|

--- 
**Question 3: Do fields decline at significantly different rates?**

**Method:** One Way ANOVA followed by Tukey HSD post-hoc test.

H₀: All fields have the same mean decline rate.

H₁: At least one field has a significantly different mean decline rate.

**Result:** F = 4.25, p ≈ 0 — reject H₀. Fields are genuinely different. 52 field pairs confirmed as significantly different by Tukey HSD.

---

## Recommendations

Based on the statistical findings, the following actions are recommended for the operations team:

**Immediate (0-3 months)**

- Prioritise intervention on the 40 critical wells forecast to cross the economic threshold within 3 months
- Begin with **HOMER FEDERAL 14X-32F:** It has high current production with a 19.6% monthly decline rate
- Investigate AMUNDSON 44-22NWH in SIVERSTON field: D = 0.933 with p = 0.0002, this shows that this is a genuine emergency

**Short term (3-6 months)**

- Deploy field level intervention strategy for SAND CREEK
- Review the 9 remaining at-risk wells (months 4-6) before they become critical
- Check the 61 wells that suffered severe post shutdown damage to identify whether mechanical failure or reservoir damage was the cause


**Strategic (6+ months)**

- Invest in preventive maintenance programmes: the correlation analysis shows planned shutdowns correlate with better recovery outcomes than unplanned outages
- Prioritise reservoir pressure management in SAND CREEK, ALKALI CREEK, and ANTELOPE fields, they are all showing **Mean D** above **0.30**

---
## Tools used

- Python (pandas, numpy)
- scipy.optimize.curve_fit
- scipy.stats
- statsmodels
- matplotlib
- Power BI

---

[Interact with the dashboard here](https://app.powerbi.com/groups/me/reports/e5bd373e-2ce7-41fc-b781-64b1126070e3/a30c577b651e03784172?experience=power-bi)


*Analysis by Oluwagbemiga Agbeje - Energy Data Analyst*


