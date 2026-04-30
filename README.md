# Zerve × ODSC AI Datathon
## Predicting User Upgrades & Behavioural Funnel Analysis

Video: https://drive.google.com/file/d/1-_jXsdyx9kHQ0tevIdBipuZS9i4R4W7X/view?usp=share_link

Zerve Project: https://app.zerve.ai/notebook/c5e0208b-26a0-48ec-8f47-9a32eccb2a2a

Zerve Report: https://app.zerve.ai/report/5c04a671-79ce-4c05-ad68-498708be55d4

Deployment App: https://zerve-upgrade-datathon.hub.zerve.cloud

import pandas as pd
from sklearn.metrics import roc_auc_score, average_precision_score

importance = (pd.DataFrame({
        'feature':    feature_cols,
        'importance': model.feature_importances_,
    })
    .sort_values('importance', ascending=False)
)

high_prob = features[
    (features['upgrade_probability'] >= 0.5) & (features['upgraded'] == 0)
]

report = f"""
================================================================
ZERVE x ODSC DATATHON — SUBMISSION REPORT
================================================================

1. DATASET
   Total users          : {n_total:,}
   Upgraded users       : {n_upgraded:,} ({base_rate:.1f}%)
   Not upgraded         : {n_total - n_upgraded:,}
   Pre-upgrade events   : {len(pre):,}

2. LEAKAGE PREVENTION
   - Loaded only 7 columns (saves 80% memory)
   - Kept ONLY events before upgrade_ts per user
   - Removed 6 post-billing leakage event types
   - Leaked rows: 0

3. FEATURES ({len(feature_cols)} total)
   Volume     : total_events, unique_event_types, events_per_day
   Engagement : days_active, lifespan_days, run_count, canvas_count
   AI usage   : agent_actions, agent_tool_calls
   Credits    : credits_used_count, credit_warning_count
   Billing    : addon_credits_count, clicked_upgrade_count
   Deployment : deployment_count
   Identity   : role, purpose, work_type, source (encoded)
   Combos     : power_and_warned, high_agent_high_credits

4. MODEL — Random Forest
   ROC-AUC       : {roc_auc_score(y_test, y_pred_proba):.3f}
   Avg Precision : {average_precision_score(y_test, y_pred_proba):.3f}
   Class weight  : balanced
   Split         : 80/20 stratified

   Top 5 features:
{importance.head(5).to_string(index=False)}

5. FUNNEL
   Converted  : {len(features[features['stage_name']=='Converted']):>6,} users
   Power User : {len(features[features['stage_name']=='Power User']):>6,} users
   Activated  : {len(features[features['stage_name']=='Activated']):>6,} users
   Onboarded  : {len(features[features['stage_name']=='Onboarded']):>6,} users

6. KEY SEGMENTS (base rate: {base_rate:.1f}%)
   Bought addon credits   : {(features['addon_credits_count']>=1).sum():,} users  {features[features['addon_credits_count']>=1]['upgraded'].mean()*100:.1f}%  ({features[features['addon_credits_count']>=1]['upgraded'].mean()*100/base_rate:.1f}x lift)
   Hit credit limit       : {(features['credit_warning_count']>=1).sum():,} users  {features[features['credit_warning_count']>=1]['upgraded'].mean()*100:.1f}%  ({features[features['credit_warning_count']>=1]['upgraded'].mean()*100/base_rate:.1f}x lift)
   Any AI usage           : {(features['agent_actions']>=1).sum():,} users  {features[features['agent_actions']>=1]['upgraded'].mean()*100:.1f}%  ({features[features['agent_actions']>=1]['upgraded'].mean()*100/base_rate:.1f}x lift)
   High-prob non-converters (p>=0.5): {len(high_prob):,} users

7. RECOMMENDATIONS
   R1. Show upgrade offer when user buys addon credits
   R2. Trigger upgrade prompt within 24h of first credit warning
   R3. Nudge Onboarded users to try the AI agent
   R4. Run campaign targeting {len(high_prob):,} high-prob non-converters

================================================================
"""
print(report)
