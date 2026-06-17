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
