In app.py /assess-release-risk, after fetching live 
EC2 signals, merge in optional request fields if present:

if req.test_pass_rate is not None:
    features["test_pass_rate"] = req.test_pass_rate
if req.lower_env_error_rate is not None:
    features["lower_env_error_rate"] = req.lower_env_error_rate
if req.lower_env_latency_p95_ms is not None:
    features["lower_env_latency_p95_ms"] = req.lower_env_latency_p95_ms

In models.py add these 3 to EC2_FEATURE_WEIGHTS and 
EC2_FEATURE_THRESHOLDS:
test_pass_rate: weight 0.15, threshold 0.95 
  (risk if BELOW this, not above — handle inverse logic)
lower_env_error_rate: weight 0.05, threshold 0.10
lower_env_latency_p95_ms: weight 0.05, threshold 500.0

Reduce existing weights proportionally so sum stays 1.0.

In rule_based_score(), handle test_pass_rate as inverse:
breached if value < threshold (not >=).

Validate: curl POST /assess-release-risk with 
test_pass_rate=0.80 (below 0.95) → factors[] shows 
test_pass_rate triggered=true, score increases vs 
without that field.
