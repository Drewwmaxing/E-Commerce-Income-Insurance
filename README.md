# Insura
### AI-Powered Parametric Income Insurance for Food Delivery Partners

---

## Table of Contents
1. [Problem Statement](#1-problem-statement)
2. [Our Solution](#2-our-solution)
3. [Application Workflow](#3-application-workflow)
4. [Tech Stack](#4-tech-stack)
5. [Parametric Trigger Engine](#5-parametric-trigger-engine)
6. [Weekly Premium Model](#6-weekly-premium-model)
7. [Payout Calculation Engine](#7-payout-calculation-engine)
8. [AI Integration Architecture](#8-ai-integration-architecture)
9. [Fraud Detection System](#9-fraud-detection-system)
10. [Platform Choice Justification](#10-platform-choice-justification)

---

## 1. Problem Statement

India's food delivery partners (Zomato, Swiggy) are the backbone of our fast-paced digital economy, yet they operate with no financial safety net. External disruptions — extreme weather, severe pollution, civic curfews, and bandhs — can reduce their working hours and cause them to lose 20–30% of their monthly earnings overnight.

A Zomato delivery partner in Bangalore earning ₹900/day loses approximately ₹450 during a 5-hour monsoon evening. They have no mechanism to recover this loss. No insurance product exists that automatically compensates gig workers for lost income during these events without requiring manual claim filing, documentation, or waiting periods.

**The gap:** Parametric insurance — where payouts trigger automatically when a pre-defined external condition is met — has never been applied to India's gig economy at scale.

---

## 2. Our Solution

**Insura** is an AI-enabled parametric income insurance platform built exclusively for Zomato and Swiggy food delivery partners. When a verified external disruption crosses a defined threshold in a worker's operating zone, the system automatically detects the event, validates the worker's active policy, runs a fraud check, and transfers the payout — with zero intervention required from the worker.

**"Verified external disruption"** refers to an external parameter (weather, AQI, civic event) that meets both conditions:
1. **Threshold Breach:** The parameter exceeds its defined threshold (e.g., > 15mm/hr rainfall)
2. **Temporal Confirmation:** The threshold is breached for 2 consecutive polling cycles (30 minutes) as per the trigger definitions in Section 5

**Core principles:**
- **Income loss coverage only** — no health, life, accident, or vehicle repair coverage
- **Weekly premium model** — aligned with gig worker earning cycles
- **Zero-touch claims** — no forms, no waiting, no rejection risk
- **Peak-hour weighted payouts** — compensation proportional to actual income lost, not a flat hourly rate
- **Explainable AI** — workers and admins can see exactly why a premium or payout was calculated the way it was

---

## 3. Application Workflow

```
┌──────────────────────────────────────────────────────────────────┐
│                        WORKER JOURNEY                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1 — ONBOARDING                                             │
│  Worker enters: name, phone, city, operating pincode,            │
│  platform (Zomato/Swiggy), UPI ID, average weekly earnings,      │
│  typical shift hours, days active per week.                      │
│                          │                                       │
│  STEP 2 — AI RISK PROFILING                                      │
│  Random Forest Regressor analyses: zone flood history,           │
│  seasonal disruption frequency, city tier, shift timing.         │
│  SHAP explains which features drove the risk score.              │
│  Claude API translates SHAP output into plain-language           │
│  explanation shown to the worker.                                │
│                          │                                       │
│  STEP 3 — POLICY CREATION                                        │
│  Worker selects coverage tier (Basic / Standard / Premium).      │
│  System calculates weekly premium using parametric model.        │
│  Worker pays via Razorpay. Policy activates immediately.         │
│                          │                                       │
│  STEP 4 — ACTIVE COVERAGE DASHBOARD                              │
│  Worker sees: active policy details, current weather risk        │
│  level in their zone, earnings protected this week,              │
│  payout history, remaining weekly coverage cap.                  │
│                          │                                       │
│  STEP 5 — TRIGGER ENGINE (Fully Automated)                       │
│  APScheduler polls OpenWeatherMap + AQICN every 15 minutes.      │
│  Evaluates all trigger thresholds for each active zone.          │
│  Requires 2 consecutive threshold breaches before firing         │
│  to prevent false positives from brief weather spikes.           │
│                          │                                       │
│  STEP 6 — FRAUD VALIDATION                                       │
│  Random Forest Classifier scores the claim.                      │
│  SHAP identifies which signals flagged it.                       │
│  Claude API generates human-readable fraud assessment            │
│  for admin review if score is Medium or High risk.               │
│                          │                                       │
│  STEP 7 — INSTANT PAYOUT                                         │
│  Clean claims → Razorpay test mode (Phase 1) / Live mode (Phase 2) │
│  → UPI transfer. Payout logged in worker & admin dashboards.     │
│  Expo push notification delivered to worker's device.            │
│  Flagged claims → held in admin review queue.                    │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. Tech Stack

| Layer | Technology | Reason for Selection |
| :--- | :--- | :--- |
| **Mobile Frontend** | React Native + Expo | Cross-platform (Android + iOS), fast iteration, native push notifications. |
| **UI Styling** | NativeWind | Tailwind for React Native; consistent, clean design system. |
| **Backend** | FastAPI (Python) | High performance, async support, auto-generated OpenAPI docs. |
| **Database & Auth** | Supabase (PostgreSQL) | Free hosted DB with out-of-the-box secure authentication. |
| **Predictive Risk (ML)** | Scikit-learn — Random Forest Regressor | Data-driven premium calculation using zone, season, and worker profile features. |
| **Fraud Detection (ML)** | Scikit-learn — Random Forest Classifier | Anomaly scoring on claim context, GPS consistency, and account behaviour. |
| **Explainable AI (XAI)** | SHAP | Industry-standard library to mathematically prove feature importance and eliminate black-box decision making. |
| **Communication AI** | Claude API (Anthropic) | Translates SHAP output into user-friendly, multilingual notifications and admin insights. |
| **Weather Triggers** | OpenWeatherMap API | Reliable free tier, excellent coverage for Indian cities. Polling frequency: 15 min (balance between real-time detection and API rate limits). |
| **AQI Triggers** | AQICN API | Real-time AQI by city, free tier, covers all major Indian metros. |
| **Civic Triggers** | Mock JSON API (self-hosted) | Admin-controlled boolean simulating platform zone suspension events. **Phase 3:** Will be replaced by real Zomato/Swiggy webhook integration. |
| **Payments** | Razorpay Test Mode (Phase 1) / Live Mode (Phase 2+) | Free sandbox for demonstrating instant simulated UPI payouts (Phase 1). Real KYC and UPI transfers in Phase 2+. |
| **Push Notifications** | Expo Push Notifications | Free, native delivery for zero-touch payout alerts to worker's device. |
| **Trigger Automation** | APScheduler (Python) | Lightweight polling inside FastAPI; checks all triggers every 15 minutes. Trigger fires only after 2 consecutive threshold breaches (30-minute confirmation window). |
| **Deployment** | Railway.app | Seamless GitHub integration and zero-config Python deployment. |

### Payment Integration Note

**Phase 1 (Hackathon/Prototype):** Payouts are processed in Razorpay test mode, which means transfers are simulated and no real money changes hands. Workers see a confirmation in their dashboard and a push notification, but no actual UPI debit occurs.

**Phase 2 (Beta):** Razorpay live mode will be integrated with proper KYC validation and real UPI payouts to worker bank accounts.

---

## 5. Parametric Trigger Engine

Triggers are objective, externally verifiable thresholds. No worker input is required. A trigger fires only when the parameter has been above threshold for **two consecutive polling cycles (30 minutes)** — preventing false positives from brief weather spikes. This 2-cycle rule applies to all triggers unless explicitly noted otherwise.

### Trigger Definitions

| # | Trigger | Parameter | Threshold | Duration Requirement | Data Source | Notes |
|---|---|---|---|---|---|---|
| 1 | Heavy Rain | Rainfall intensity | > 15mm/hr sustained | 30 min (2 cycles × 15 min) | OpenWeatherMap | IMD "heavy rain" classification; platforms begin reducing assignments |
| 2 | Flash Flood Risk | Rainfall accumulation | > 40mm in 3hrs | Cumulative threshold breach in 2 consecutive polling windows | OpenWeatherMap | Urban road flooding threshold; cumulative measurement |
| 3 | Extreme Heat | Heat Index (feels like) | > 48°C | 30 min sustained | OpenWeatherMap | Accounts for humidity; more accurate than raw temperature alone |
| 4 | Severe AQI | AQI Index | > 400 | 30 min sustained | AQICN API | GRAP Stage 4 — government-classified severe outdoor exposure risk |
| 5 | Dense Fog | Visibility | < 150m | 30 min sustained | OpenWeatherMap | Time-windowed: 5am–9am and 9pm–12am only (peak traffic periods) |
| 6 | High Wind | Wind gust speed | > 60 kmph | 30 min sustained | OpenWeatherMap | Two-wheeler safety threshold; applied to coastal cities only (Mumbai, Chennai, Goa) |
| 7 | Civic Disruption | Boolean flag (zone suspended) | = true | 2 consecutive breaches (admin confirmed) | Mock Admin API (Phase 1) / Zomato/Swiggy webhook (Phase 3) | Admin-toggled; simulates aggregator zone suspension. GPS validation required (see below). |

### Multi-Trigger Rule

If multiple triggers are active simultaneously in the same hour, payouts **do not stack**. The payout is calculated as:

```
final_payout = MAX(
    payout from Trigger A,
    payout from Trigger B,
    payout from Trigger C,
    ...
)
```

Only the trigger yielding the highest payout is applied. No additional bonus or cumulative compensation is granted for multiple simultaneous triggers. This prevents over-compensation and maintains actuarial viability.

**Example:** If both Heavy Rain (dinner peak: 1.6×) and Severe AQI trigger simultaneously at 8pm, only the one with the higher final payout amount is awarded — no stacking.

### Simulated Platform Webhooks (Phase 3 Roadmap)

Our system exposes a POST endpoint that ingests mock state-change events simulating aggregator API behaviour. When an admin triggers a zone suspension event — replicating what a real aggregator would push when flagging a geofence as "SUSPENDED" due to a local strike or unplanned curfew — our backend cross-references all active policies within that zone.

**Geofence Definition:**
Geofence boundaries are defined using the worker's registered operating pincode. During onboarding, the worker selects their primary delivery pincode. Civic trigger validation checks if the GPS ping falls within the postal code boundary (determined via reverse geocoding API). If the worker operates across multiple pincodes, the system checks the pincode where the trigger was reported by the aggregator.

To validate the claim, the worker's GPS is pinged at the moment of trigger (not continuously tracked, preserving privacy) and checked against the suspended geofence boundary. GPS validation requires:
- Worker device has location permissions granted to Insura app
- GPS accuracy is within ±100m; if worse, claim is flagged for manual review
- GPS ping occurs within 5 minutes of trigger detection
- Workers cannot opt out; GPS validation is a condition of civic trigger coverage

Based on this two-factor validation, the system assigns a confidence score:

| Condition | Confidence | Action |
|---|---|---|
| Geofence suspended + Worker GPS inside zone + Accuracy ±100m | **High** | Auto-approve payout |
| Geofence suspended + Worker GPS outside zone | **Medium** | Flag for manual review (worker may have been at zone boundary) |
| Geofence suspended + GPS accuracy > ±100m | **Medium** | Flag for manual review (unreliable location data) |
| No geofence suspension + Worker claims disruption | **Low** | Auto-reject |

**Phase 3 Roadmap:** In production, this endpoint would be replaced by a real webhook subscription to the aggregator's platform API.

---

## 6. Weekly Premium Model

### Formula

```
weekly_premium = base_rate
               × season_multiplier
               × zone_risk_multiplier
               × city_tier_factor
               - loyalty_discount
               + coverage_tier_addon
```

### Component Breakdown

#### Base Rate
Set at 3–5% of weekly declared income.

**Example:** Worker declaring ₹5,400/week → base premium = ₹162–₹270/week.

#### Season Multiplier

| Season | Multiplier | Period | Transition |
|---|---|---|---|
| Monsoon | 1.40× | June – September | Changes on 1st of each month; workers notified 7 days in advance |
| Pre-Monsoon | 1.15× | April – May | Changes on 1st of each month; workers notified 7 days in advance |
| North India Winter Fog | 1.20× | December – January | Changes on 1st of each month; workers notified 7 days in advance |
| Dry / Normal | 0.90× | Rest of year | Changes on 1st of each month; workers notified 7 days in advance |

**Note:** Season multiplier changes apply to the next policy week (7-day cycle). Workers can downgrade their coverage tier anytime to reduce the impact of a season multiplier increase.

#### Zone Risk Multiplier

Derived from historical weather disruption frequency at the worker's operating pincode using the Random Forest Regressor model.

**Phase 1 (Prototype):** Zone risk multipliers are modelled using synthetic training data generated from IMD historical rainfall records and urban flood zone classifications. Flood-prone zones receive higher multipliers (1.15–1.25×); historically safe zones receive discounts (0.85–0.95×).

**Phase 3 (Production):** Zone risk multipliers will be calibrated against actual disruption frequency observed from real claims data and trigger events. This ensures premiums evolve with real-world loss ratios.

**Expected Disruption Baseline Assumptions (Phase 1):**

| City | Dominant Risk | Expected Disrupted Hours/Week | Zone Risk Range |
|---|---|---|---|
| Mumbai | Monsoon floods | 3–4 hrs (monsoon) | 1.15–1.25× |
| Bangalore | Isolated heavy rains | 2–3 hrs (monsoon) | 1.00–1.05× |
| Delhi | Fog + pollution | 1–2 hrs (winter) | 1.10–1.15× |
| Hyderabad | Flash floods | 2 hrs (monsoon) | 1.08–1.12× |
| Chennai | Cyclones + floods | 3–4 hrs (monsoon + Oct–Nov) | 1.20–1.30× |

#### City Tier Factor

Mumbai monsoon risk is not the same as Bangalore rain risk which is not the same as Delhi fog risk. City-specific base calibration is applied at onboarding to reflect each city's dominant disruption profile.

| City | Dominant Risk | City Tier Factor | Reason |
|---|---|---|---|
| Mumbai | Monsoon floods | 1.25× | Highest rainfall + urban flooding frequency in India |
| Bangalore | Isolated heavy rains | 1.05× | Seasonal but less sustained than coastal cities |
| Delhi | Fog + pollution | 1.15× | Winter disruptions; less monsoon risk but fog/pollution severe |
| Hyderabad | Flash floods | 1.10× | Localized flooding in low-lying zones during monsoon |
| Chennai | Cyclones + floods | 1.30× | Coastal, high seasonal volatility, cyclone risk |
| Other tier-1 cities | Regional average | 1.00× | Baseline for moderate disruption zones |

#### Loyalty Discount

5% reduction applied to weekly premium starting from week 5, provided weeks 1–4 had active coverage with zero claims triggered. Discount resets if any claim is triggered in subsequent weeks.

**Timeline Example:**
- Week 1–4: Active coverage, no claims → Loyalty conditions met
- Week 5: 5% discount applied to premium
- Week 6: If no claim triggered, 5% discount continues
- Week 6: If any claim triggered, discount resets → Week 7 onwards, must rebuild 4-week streak

#### Coverage Tiers

| Tier | Income Protected | Weekly Add-on | Coverage Ratio |
|---|---|---|---|
| Basic | 70% | ₹0 (base) | 0.70 |
| Standard | 85% | +₹35/week | 0.85 |
| Premium | 100% | +₹70/week | 1.00 |

**Coverage Ratio Definition:**
`coverage_ratio` = the percentage of weekly declared income the worker selects at policy purchase. This percentage directly determines:
- The weekly payout cap (= weekly_income × coverage_ratio)
- The proportion of each hour's income loss compensated per trigger event

**Example:** A Standard tier worker with coverage_ratio = 0.85 receives 85% of hourly income loss as payout, capped at 85% of their weekly declared income.

### Actuarial Viability Check

```
expected_weekly_payout = hourly_income_rate
                       × expected_disrupted_hours_per_week
                       × avg_peak_multiplier
                       × coverage_ratio

viable_premium = expected_weekly_payout × (1 + expense_ratio)
```

Where:
- `expected_disrupted_hours_per_week` = predicted by Random Forest Regressor for each zone and season (see baseline assumptions above)
- `avg_peak_multiplier` = weighted average of peak/off-peak multipliers based on worker's shift hours
- `coverage_ratio` = selected tier (0.70, 0.85, or 1.00)
- `expense_ratio` = 0.25 (25% overhead for operations and margin; set conservatively for Phase 1)

Premiums are calibrated so that a consistent payout scenario remains financially sustainable for the insurer.

**Sensitivity Analysis Note:**
If actual disruption frequency is 2–3× higher than baseline Phase 1 assumptions, premiums will require repricing or risk pooling adjustments in Phase 2. Expected disruption data from Phase 1 real claims will be used to calibrate Phase 2 models.

---

## 7. Payout Calculation Engine

### Hourly Income Rate Calculation

```
hourly_income_rate = weekly_declared_income / active_hours_per_week

Example:
Worker declares ₹5,400/week, active 11am–11pm daily (72 hours/week)
hourly_income_rate = ₹5,400 / 72 = ₹75/hour
```

### Peak Hour Multiplier

Payouts are not flat-rate per hour. They reflect actual income loss using the hourly income distribution of food delivery workers:

| Time Slot | Type | Multiplier | Justification |
|---|---|---|---|
| 12pm – 2pm | Lunch Peak | 2.0× | Highest order density and earnings |
| 7pm – 10pm | Dinner Peak | 1.6× | Second-highest density; slightly lower than lunch |
| 11am – 12pm, 6pm – 7pm | Ramp-up | 1.2× | Transition periods; moderate earning potential |
| 2pm – 6pm | Off-Peak | 0.6× | Lowest order density; lower earnings |
| 10pm – 12am | Late Night | 0.4× | Very few orders; marginal earnings |
| 12am – 11am | Closed/Inactive | 0.0× | Worker typically offline |

**Justification:** These multipliers are based on industry estimates of peak-hour order frequency for Indian food delivery platforms. Actual multipliers will be calibrated in Phase 2 using primary worker survey data and earnings logs from Zomato/Swiggy partner APIs.

### Disrupted Hours Calculation

Only hours that fall within the worker's **registered shift window** are counted. 

**Edge Case Handling:**
- Disruptions are rounded **DOWN to the nearest hour**
- A 30-minute disruption counts as 0 hours (no payout)
- A 90-minute disruption counts as 1 hour
- A 125-minute disruption counts as 1 hour
- A 150-minute disruption counts as 2 hours

This conservative approach prevents over-compensation for brief events.

**Boundary Example:**
Worker's shift: 11am–11pm. Heavy rain fires at 10:30pm (30 min before shift end).
- Rain duration: Let's say 1 hour total (10:30pm–11:30pm)
- Overlap with shift: 10:30pm–11pm = 30 minutes
- Rounded down: 0 hours → **No payout**
- Worker receives: ₹0

If rain had fired at 10:00pm and lasted 2 hours (10:00pm–12:00am):
- Overlap with shift: 10:00pm–11pm = 1 hour
- Rounded down: 1 hour → **Payout applies**

### Per-Hour Payout Calculation

For disruptions spanning multiple time slots, each hour is calculated separately using its respective peak multiplier:

```
for each disrupted hour H:
    sub_payout[H] = hourly_income_rate 
                  × peak_multiplier[H] 
                  × coverage_ratio
```

Sum all sub-payouts. Cap at weekly limit (see below).

### Weekly Payout Cap

```
weekly_payout_cap = weekly_declared_income × coverage_ratio
```

A worker cannot receive more in cumulative weekly payouts than their declared weekly income multiplied by their coverage ratio.

**Policy Week Definition:**
Policy weeks run on a rolling 7-day cycle from the date the policy was activated. Once the weekly cap is reached, subsequent claims in that week are auto-rejected with a notification: 

*"Weekly payout limit reached. Excess claims will be eligible next week."* 

Rejected claims are logged but do NOT carry over; they expire at week-end.

### Full Payout Formula

```
final_payout = MIN(
    Σ (hourly_income_rate × peak_multiplier × coverage_ratio) for each disrupted hour,
    weekly_payout_cap
)
```

### Worked Example 1: Simple Disruption

**Worker:** Rahul, Bangalore, Standard tier (85% coverage)
- **Weekly declared income:** ₹5,400
- **Active hours:** 11am–11pm (12 hours/day × 6 days) = 72 hours/week
- **Hourly rate:** ₹5,400 / 72 = ₹75/hr
- **Coverage ratio:** 0.85 (Standard tier)
- **Weekly payout cap:** ₹5,400 × 0.85 = ₹4,590

**Event:** Heavy rain, 7:00pm – 10:00pm (3 hours, dinner peak = 1.6×)

| Hour | Time | Multiplier | Hourly Rate | Coverage | Sub-payout |
|---|---|---|---|---|---|
| 1 | 7pm–8pm | 1.6× (dinner) | ₹75 | 0.85 | ₹102.00 |
| 2 | 8pm–9pm | 1.6× (dinner) | ₹75 | 0.85 | ₹102.00 |
| 3 | 9pm–10pm | 1.6× (dinner) | ₹75 | 0.85 | ₹102.00 |
| **Total Payout** | | | | | **₹306.00** |

**Status:** **Approved** (within ₹4,590 weekly cap)

Rahul receives **₹306** automatically — well within his weekly cap.

### Worked Example 2: Multi-Hour Disruption Crossing Time Slots

**Same Worker:** Rahul, ₹75/hr, 0.85 coverage ratio

**Event:** Heavy rain, 1:30pm – 3:30pm (2 hours, lunch + off-peak transition)

| Hour | Time | Multiplier | Hourly Rate | Coverage | Sub-payout |
|---|---|---|---|---|---|
| 1 | 1:30pm–2:30pm | 2.0× (lunch peak) | ₹75 | 0.85 | ₹127.50 |
| 2 | 2:30pm–3:30pm | 0.6× (off-peak) | ₹75 | 0.85 | ₹38.25 |
| **Total** | | | | | **₹165.75** |

**Status:** **Approved**

Rahul receives **₹165.75**.

---

## 8. AI Integration Architecture

Insura uses a **layered AI architecture** that separates mathematical prediction (Random Forest + SHAP) from human communication (Claude API). This ensures every decision is both statistically defensible and plainly explainable to workers and admins.

```
Raw Data (weather, worker profile, claim context)
            │
            ▼
  Random Forest Model (Scikit-learn)
  → Outputs: risk score, fraud score (numeric 0.0–1.0)
            │
            ▼
       SHAP Analysis
  → Outputs: feature importance values (normalized to sum = 100%)
  → Example: "Zone risk: 40%, monsoon season: 35%, loyalty history: 25%"
            │
            ▼
     Claude API (Anthropic)
  → Input: structured SHAP data + context + worker/admin context
  → Output: plain-language explanation for worker or admin
```

### Use Case 1 — Risk Profiling at Onboarding

**Input to Claude API:** 
- Worker's operating pincode, city, platform, weekly earnings, shift hours
- SHAP-explained risk score from Random Forest (e.g., "0.68 out of 1.0")
- Feature importance breakdown (normalized percentages summing to 100%)

**Output:** Plain-language risk tier explanation shown to worker at onboarding.

**Example:** 
> *"You operate in Bangalore zone 560034, which has moderate flood risk during monsoon season (June–Sept). Your weekly premium is ₹220 for Standard coverage. You can reduce your premium by selecting Basic coverage (70% income protection) for ₹150/week."*

---

### Use Case 2 — Dynamic Premium Explanation

**Input to Claude API:** 
- Final premium amount
- SHAP feature breakdown (zone risk weight %, season weight %, loyalty discount applied %)

**Output:** One-sentence explanation displayed under the premium amount in the app.

**Example:** 
> *"Your premium increased by ₹25 this week because IMD issued a heavy rain advisory for your zone (June monsoon). Next week's premium will return to ₹220 if the advisory is lifted."*

---

### Use Case 3 — Fraud Detection Communication

**Input to Claude API:** 
- Fraud score (0.0–1.0 from Random Forest Classifier)
- SHAP features that flagged the claim (with normalized importance values):
  - "GPS movement during trigger window: 45% importance"
  - "Claim frequency anomaly: 35% importance"
  - "Device duplication: 20% importance"

**Output:** Human-readable fraud assessment for admin review queue. Only generated for Medium and High fraud scores.

**Example:** 
> *"This claim is flagged as Medium risk (0.58 score). The worker's GPS showed active movement during 2 of the 3 disrupted hours, suggesting partial work continuation despite the heavy rain trigger. Claim frequency is within zone average. Recommendation: Review order completion logs from the aggregator API for those 2 hours before approving. If orders were completed, the claim may be valid despite GPS movement."*

---

### Use Case 4 — Admin Dashboard Insights

**Input to Claude API:** 
- Weekly aggregated claims data (total payouts, count by trigger type)
- Zone-wise trigger frequency (how many times each trigger fired per zone)
- Loss ratios (total payouts vs. total premiums collected)

**Output:** Natural language weekly summary for the insurer dashboard.

**Example:** 
> *"Zone 560034 (Bangalore) had 18 claims this week (₹4,280 payouts), 3× higher than average, driven by 4 heavy rain trigger events. Loss ratio for the zone: 15% (sustainable). Recommend monitoring rainfall forecasts for next week; if another high-rain week occurs, consider reviewing the zone risk multiplier from 1.05× to 1.10× before the next premium cycle."*

---

### SHAP Value Explanation for Workers

SHAP returns raw feature importance values. For transparency, these are converted to percentages representing relative contribution to the final score (sum = 100%). Workers see this as:

> *"Your premium breakdown: Zone risk contributes 40% (you're in a monsoon-prone area), Monsoon season contributes 35% (June–Sept high disruption period), and your loyalty history reduces it by 25% (4 weeks with no claims)."*

This ensures workers understand that these are **relative weights**, not **absolute accuracy scores**.

---

## 9. Fraud Detection System

### Signals Monitored

| Signal | Method | Risk Weight | Notes |
|---|---|---|---|
| GPS stationary during trigger window | GPS ping at claim time vs trigger zone boundary | High | Geofence validation (civic triggers only) |
| GPS active movement during trigger window | Movement detected despite claimed disruption | High | May indicate partial work continuation; flagged for review |
| Duplicate device fingerprint across accounts | Device ID cross-check at registration | High | Prevents multiple account fraud |
| Duplicate UPI ID across accounts | UPI ID uniqueness check at registration | High | Prevents beneficiary fraud |
| Claim frequency above zone average | Statistical outlier detection (Z-score > 2.5) | Medium | Workers claiming 5× more than peers in same zone |
| Orders completed during trigger window | Cross-check via platform API | High | **Phase 1:** Not implemented (mock data only). **Phase 3:** Integration with Zomato/Swiggy order logs. |
| Account age < 1 week with first claim | Registration timestamp check | Medium | New accounts with immediate claims flagged for review |
| Premium payment failure history | Check for bounced UPI/card payments | Medium | Indicates potential financial instability |

**Note on Platform API Integration:**
In Phase 1 (prototype), order completion validation is NOT implemented. This fraud signal is marked as "low confidence" in the Random Forest Classifier. In Phase 3 (production roadmap), Insura will integrate with Zomato/Swiggy partner APIs to verify if the worker completed orders during the trigger window. Integration status: Estimated Q4 2026 (pending partner API access).

### Fraud Score Tiers

```
fraud_score = Random Forest Classifier output (0.0 – 1.0)

LOW    (0.0 – 0.35) → Auto-approve payout
MEDIUM (0.35 – 0.70) → Hold for admin review (24hr SLA)
HIGH   (0.70 – 1.0) → Auto-reject, flag account
```

SHAP explains which signals drove the score. Claude API translates this into an admin-readable summary in the review queue.

### Appeal Mechanism

Auto-rejected claims (fraud_score > 0.70) are **not** immediately cause for account suspension. Instead:

1. **Worker Notification:** The worker receives a notification: 
   > *"Your claim for ₹XXX was not approved. Please contact support for details."*

2. **Support Review:** Insura support team reviews the case within 24 hours.

3. **Worker Escalation:** Workers can escalate via in-app support chat. Support can provide limited context (e.g., "Your claim was flagged due to unusual account activity").

4. **Dispute Form (Phase 3):** In Phase 3, workers will have access to a dispute form showing the SHAP-explained fraud signals, allowing them to provide context or counter-evidence.

5. **Appeal Outcome:**
   - If appeal is successful, claim is approved and payout processed
   - If appeal is denied, claim remains rejected but no penalty is applied to account
   - Two rejected claims within 30 days → account flagged for manual compliance review
   - Three rejected claims within 30 days → account suspension + manual review required before reactivation

---

## 10. Platform Choice Justification

**Insura** is built as a **mobile-first application** using React Native with Expo for the following reasons:

**1. Context Alignment:** Zomato and Swiggy delivery partners are smartphone-native users who manage all aspects of their work — order acceptance, navigation, earnings tracking — through mobile apps. A mobile interface matches the context they already operate in.

**2. Push Notifications (Critical):** Push notifications are essential for the zero-touch claim experience. When a payout is processed automatically, the worker must be notified instantly on their device. This real-time, native notification capability is only reliably achievable through a mobile app.

**3. Cross-Platform Efficiency:** React Native with Expo enables cross-platform development (Android + iOS) from a single codebase without requiring separate native development expertise. Expo's managed workflow also means the app is accessible via mobile browser as a Progressive Web App (PWA), reducing the adoption barrier by eliminating app store submission requirements during the prototype phase.

**4. Zero-Touch User Experience:** The mobile-first design enables workers to:
   - See real-time disruption alerts
   - Receive instant payout notifications
   - View claim history and coverage status
   - Access support chat — all without leaving their delivery work

---

## Project Phases

### **Phase 1: Prototype (Current — Hackathon)**
- Core trigger engine (weather + AQI + mock civic)
- Parametric premium calculation
- Zero-touch claim processing
- Random Forest risk profiling & fraud detection
- SHAP explainability + Claude API communication
- Razorpay test mode (simulated payouts)
- Synthetic training data for zone risk multipliers
- Real Zomato/Swiggy API integration (not yet)
- Real UPI payouts (not yet)
- Order completion validation (not yet)

### **Phase 2: Beta (Q2 2026)**
- Razorpay live mode with real KYC
- Actual UPI transfers to worker accounts
- Mobile app store submission (iOS + Android)
- Localized language support (Hindi, Tamil, Telugu)
- Primary worker survey to calibrate peak-hour multipliers

### **Phase 3: Production (Q4 2026)**
- Real Zomato/Swiggy platform API integration for:
  - Order completion validation (fraud detection)
  - Geofence suspension events (civic triggers)
- Zone risk multiplier calibration using real claims data
- Worker dispute/appeal form with SHAP explanations
- Risk pooling or reinsurance structure for scalability

---

## Key Assumptions & Limitations (Phase 1)

1. **Zone risk multipliers** are modelled using synthetic IMD data. In Phase 3, these will be calibrated against real disruption events.

2. **Peak-hour multipliers** are based on industry estimates. Phase 2 will validate these with primary worker data.

3. **15-minute polling frequency** balances real-time detection with API rate limits. Faster polling (5 min) would increase infrastructure costs; slower polling (30 min) could miss brief disruptions.

4. **Razorpay test mode** means Phase 1 payouts are simulated. Real UPI transfers begin in Phase 2.

5. **Fraud detection** lacks order completion signals in Phase 1 (no platform API access yet). High-confidence claims are auto-approved; medium/high-risk claims are reviewed manually.

6. **GPS validation** for civic triggers requires worker permission; this is a condition of coverage for that trigger type.

---

## Contact & Support

For questions, feedback, or partnership inquiries, please reach out to the Insura team.

**Status:** Actively building toward Phase 2 Beta.

---

**Last Updated:** June 2026  
**Document Version:** 2.0 (All critical inconsistencies resolved)