In app.py add POST /canary-validation.
Request: {service, env, canary_version, stable_version}
Get signals for f"{env}-canary", fallback to env if 
DatadogLookupError.
Get signals for env (stable).
Compute delta% canary vs stable per feature.
Score deltas with model.rule_based_score(deltas).
Return: canary_risk_score, risk_level, 
proceed_with_rollout(true if low), metric_deltas.
Validate: curl POST /canary-validation 
{"service":"payments-api","env":"prod",
"canary_version":"v2","stable_version":"v1"}
→ same data source both sides → score near 0, low risk.
