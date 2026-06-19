# pc1
#2
In app.py add POST /assess-release-risk → RiskDecision.
Call get_ec2_signals(service, env) from dd_client.
Score with model.predict_issue_probability(features).
Map: >=0.8=high, >=0.5=medium, else low.
proceed=True only if low.
Build factors[] from FEATURE_THRESHOLDS — each feature
shows value, threshold, breached bool.
recommendation: high=Abort, medium=Caution, low=Safe.
Don't touch existing endpoints.
Validate: curl POST /assess-release-risk 
{"service":"payments-api","env":"prod"}
→ returns risk_level, score, factors[], proceed.


#3

In app.py add GET /anomaly-status/{service}?env=prod.
Call get_ec2_signals(service, env) for current signals.
Store first call result in memory dict as baseline.
Compute delta% per feature: (current-baseline)/baseline.
Score deltas with model.rule_based_score(deltas).
Return: anomaly_detected(score>=0.5), delta_score, 
risk_level, spiked_features[].
Validate: curl http://localhost:8000/anomaly-status/
payments-api?env=prod → returns anomaly_detected bool.
