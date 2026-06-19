AC3 is partial — anomaly endpoint does baseline 
comparison but no dedicated canary vs stable endpoint.

Add POST /canary-validation to app.py:
Request: {service, env, canary_version, stable_version}
Fetch signals for f"{env}-canary" via get_ec2_signals,
fallback to env if DatadogLookupError.
Fetch signals for env (stable) via get_ec2_signals.
Compute delta% canary vs stable per feature 
(reuse existing delta logic if present).
Score deltas with model.rule_based_score(deltas).
Return: canary_risk_score, risk_level, 
proceed_with_rollout(true if low), metric_deltas,
canary_version, stable_version.
Validate: curl POST /canary-validation 
{"service":"payments-api","env":"prod",
"canary_version":"v2","stable_version":"v1"}
→ same source both sides → score near 0, low risk.
