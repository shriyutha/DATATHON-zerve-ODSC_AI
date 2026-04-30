# Zerve × ODSC AI Datathon
## Predicting User Upgrades & Behavioural Funnel Analysis

Video: https://drive.google.com/file/d/1-_jXsdyx9kHQ0tevIdBipuZS9i4R4W7X/view?usp=share_link

Zerve Project: https://app.zerve.ai/notebook/c5e0208b-26a0-48ec-8f47-9a32eccb2a2a

Zerve Report: https://app.zerve.ai/report/5c04a671-79ce-4c05-ad68-498708be55d4

Deployment App: https://zerve-upgrade-datathon.hub.zerve.cloud

User Upgrade Prediction: Zerve × ODSC AI Datathon

### Executive Summary
This analysis of 3.5 million Zerve product events identifies which free users are most likely to upgrade to a paid plan. A Random Forest classifier trained on 17,485 users achieves a ROC-AUC of 0.853 by focusing on genuine predictive signals rather than leakage. The work delivers two actionable outputs: an upgrade prediction model and a behavioral funnel revealing where users convert.

The core finding is that upgrade behavior is driven by resource constraints (hitting credit limits) combined with high AI usage, not by sustained activity alone. This insight directly shapes the recommendations: real-time event-triggered interventions (e.g., prompts at the moment of addon credit purchase) will outperform batch-scored weekly emails for this product.

### Methodology: Data Integrity and Feature Engineering and Target Leakage Prevention

The most common failure mode in conversion models is target leakage—accidentally including post-upgrade signals in training data. We eliminated it systematically:

Identified the exact upgrade timestamp (first subscription_upgraded or upgrade_subscription event) for each user
Kept only events with timestamp < upgrade_ts (or null for non-converters)
Removed inherently post-upgrade events: subscription_cancelled, downgrade_subscription, renew_plan, auto_charge_enabled
Verified programmatically: 0 leaked rows out of 922 upgraded users
Result: 1,709,462 pre-upgrade rows analyzed, 841,284 post-upgrade rows excluded.

### Feature Engineering

All 22 features were computed from the pre-upgrade event table:

Family	Features	Signal
Volume	total_events, unique_event_types, events_per_day	How much the user engaged
Lifecycle	lifespan_days, days_active	Whether they returned over time
AI Usage	agent_actions, agent_tool_calls	Deepest product engagement—strongest predictor
Credit / Billing	credits_used_count, credit_warning_count, addon_credits_count	Resource pressure indicates willingness to pay
Identity	role, purpose, work_type, source (encoded)	Who the user is
Additionally, two interaction features were created: power_and_warned (heavy AI + credit warnings) and high_agent_high_credits (heavy AI + addon purchases), which flag the highest-intent user combinations.

### Model Performance
Algorithm: Random Forest Classifier

n_estimators: 100 (stable importance scores)
max_depth: 8 (prevents overfitting while capturing interactions)
class_weight: balanced (corrects 94.7% / 5.3% class imbalance)
Train / test split: 80% / 20%, stratified
Results

Metric	Value
ROC-AUC	0.853
Upgraded Recall	61%
Upgraded Precision	24%
Average Precision	0.312
The model correctly ranks 85.3% of user pairs (upgrader vs. non-upgrader) in the right order, significantly outperforming the 50% random baseline. The 61% recall on upgraded users means the model identifies 6 out of 10 actual converters, making it suitable for prioritizing outreach campaigns.

ROC Curve showing 0.853 AUC. The model achieves strong separation between upgraders and non-upgraders across a wide range of decision thresholds, substantially outperforming random (50%) classification.
Feature Importance

Rank	Feature	Importance
1	lifespan_days	0.570
2	unique_event_types	0.083
3	events_per_day	0.077
4	total_events	0.073
5	credits_used_count	0.047
6	agent_actions	0.046
7	agent_tool_calls	0.028
8	run_count	0.019
9	days_active	0.019
10	credit_warning_count	0.018
Lifespan (how long the user has been on the platform) dominates, accounting for 57% of the model's predictive power. Engagement diversity (unique_event_types) and frequency (events_per_day) are secondary signals. AI usage features (agent_actions, agent_tool_calls) matter but less than overall lifespan—indicating that sustained presence is more predictive than intensity alone.

### Behavioral Funnel: Where Users Get Stuck
We mapped user progression through five mutually exclusive behavioral stages based on their deepest product engagement pre-upgrade:

Stage	Definition	Users	%
Dormant	No events recorded after signup	0	0.0%
Onboarded	At least 1 product event	13,326	76.2%
Activated	At least 1 credit warning event	32	0.2%
Power User	10+ AI agent actions	3,205	18.3%
Converted	subscription_upgraded event fired	922	5.3%
Key Finding: The Activation Bottleneck

The funnel reveals a severe drop-off between Onboarded and Activated: only 32 users (0.24% of onboarded users) hit a credit warning without heavy AI usage. This is not a funnel leakage problem—it reflects genuine product behavior: users either skip credit limits entirely (by using the free tier lightly) or they reach them via heavy AI usage (jumping directly to Power User). The Activated stage exists but is rarely traversed in isolation.

User funnel showing progression through behavioral stages: 76.2% remain at Onboarded, 18.3% advance to Power User, and only 5.3% reach Converted. The small Activated stage (0.2%) indicates most users bypass it, moving directly from basic onboarding to heavy AI usage.
What Drives Forward Progression?

Analysis of feature distributions between upgraders and non-upgraders within each stage reveals:

Signal	Non-Upgraded	Upgraded	Ratio	Strength
agent_actions	16.6	26.2	1.6×	✓✓
agent_tool_calls	13.3	21.3	1.6×	✓✓
credit_warning_count	7.4	9.9	1.3×	✓
addon_credits_count	9.2	12.2	1.3×	✓
total_events	90.2	102.4	1.1×	~
Critical Insight: Users do not upgrade after a long journey of gradually increasing activity. They upgrade when they hit a resource ceiling—specifically when AI usage is high and credit constraints are binding simultaneously. This means event-driven, real-time interventions (triggered the moment someone buys addon credits or hits a credit limit) will vastly outperform batch-scored campaigns for this product.

### High-Value Segments: 2–3× Baseline Lift
Beyond individual features, combining signals reveals the highest-conversion user groups (base rate: 5.3%):

Segment	Users	Upgrade Rate	Lift	Action
Heavy AI + Addon Credits	643	14.6%	2.8×	Personal outreach now
Bought Addon Credits	656	14.3%	2.7×	Upgrade offer immediately
Hit Credit Limit	1,732	11.7%	2.2×	Prompt within 24 hours
Heavy AI + Credit Warning	1,713	11.7%	2.2×	In-app nudge now
Any AI Usage	6,756	7.6%	1.4×	Feature highlight email
These segments are mutually exclusive. The top two (Heavy AI + Addon Credits and Bought Addon Credits) represent the gold standard: users who are already monetizing their usage and on the verge of paying. The convergence of high AI usage and prior addon purchases creates a 2.8× multiplier on base conversion rate.

Segment lift chart showing conversion rates across six user segments versus the 5.3% baseline. Heavy AI + Addon Credits achieves 14.6% conversion (2.8× lift), validating addon purchases as the strongest conversion signal.

### Key Recommendations
R1: Real-Time Trigger on Addon Credit Purchase (Immediate)

Target: 643–656 users who have bought addon credits Conversion Rate: 14.3–14.6% (2.7–2.8× baseline) Action: Show an in-app or email upgrade offer the moment someone purchases addon credits

These users are already paying for more on the free tier. This is the highest explicit intent signal in the dataset. Do not delay with batch processing; the moment is now. This cohort converts at nearly 3× the base rate.

R2: Upgrade Prompt Within 24 Hours of Credit Warning (Immediate)

Target: 1,732 users who hit credit limits Conversion Rate: 11.7% (2.2× baseline) Action: Trigger an in-app prompt or email within 24 hours of the first credit_warning event

Users hit credit limits when resource pressure is acute. Prompt them immediately while motivation is at peak. This segment converts at more than double the baseline.

R3: Nudge Onboarded Users Toward AI Agent Features (Within 48 Hours)

Target: 13,326 users stuck at the Onboarded stage (76% of all users) Conversion Rate: 7.6% if they use AI, vs 5.3% overall (1.4× lift) Action: Show an in-product prompt 24–48 hours after signup for users who have not yet tried the AI agent

This is the largest volume opportunity. AI usage is the second-most-important feature in the model after lifespan. Moving even 10% of onboarded users to Power User status (10+ AI actions) would unlock a 1.4× conversion uplift across a much larger cohort.

R4: Campaign for 2,439 High-Probability Non-Converters (Ongoing)

Target: 718 users in Very High tier (≥75% predicted probability) + 1,721 users in High tier (50–75%) Conversion Rate: Expected 40–50% based on predicted probability Action: In-app or email campaign with direct upgrade offer or feature demo

Model-scored probabilities are now available in data_final_output.parquet. This group represents users with the strongest behavior patterns aligning with upgrade likelihood but who have not yet converted. They are the best available targets for a focused acquisition push.

R5: Optimize Acquisition Channel Mix (Strategic)

Observation: Users from Press/Media and Reddit convert at 7.7–8.0%, vs 5.1–5.3% for LinkedIn and Colleague referrals. Action: Shift acquisition budget toward Press/Media and Reddit sources; maintain or reduce spend on lower-converting channels.

### Data Quality & Assumptions

Analysis covers Sept 2025–April 2026, reflecting 8 months of product history
3.5M events, 17,485 users, 922 upgrades (5.3% conversion)
All features computed from pre-upgrade windows; 0 rows leaked
Random Forest parameters chosen for interpretability and robustness to class imbalance
Feature importance reflects the test set (20% holdout); train-test performance is aligned, indicating no overfitting

### Output & Next Steps Deliverables

data_final_output.parquet: One row per user (17,485 total). Contains all 22 features, funnel stage assignment, predicted upgrade probability, and probability tier (Low / Medium / High / Very High).
Feature importance scores and model artifacts for reproducibility and retraining.
Funnel and segment visualizations (charts embedded in this report) for stakeholder communication.
Implementation Priority

Immediate (This Week): Implement R1 (addon credit purchase trigger) and R2 (24-hour credit warning prompt). These are the highest intent signals and require minimal engineering.
Short-term (1–2 Weeks): Launch R3 (AI agent nudge for onboarded users) using the funnel stage assignments already computed.
Medium-term (Ongoing): Deploy R4 campaign targeting High and Very High probability users. Refresh model scores monthly.
Strategic (Quarterly): Monitor R5 (channel mix), benchmark against the 2.8× lift baseline.
Methodological Transparency

### Business Recommendations:
R1 — Trigger upgrade offer when user buys addon credits
656 users are already paying for more on the free tier.
Show an upgrade prompt immediately. Highest intent signal available.
R2 — Real-time credit warning trigger
1,732 users hit credit limits before converting.
Show upgrade prompt within 24 hours of first warning —
not a weekly batch email. Captures intent at peak motivation.
R3 — Nudge Onboarded users to try AI agent
76% of users never touched AI features.
An in-product prompt 24–48h after signup for non-AI users
would move the largest volume of users toward conversion.
R4 — Campaign for 2,439 high-probability non-converters
Users with upgrade probability ≥ 0.50 who haven't converted yet
are your easiest wins. Run a focused in-app or email campaign now.
