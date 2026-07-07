In dd_client.py add get_service_dependencies(service, env):
  Call Datadog GET /api/v1/services?env={env} to get 
  all services list.
  Call Datadog GET /api/v1/service_dependencies?env={env}
  to get full dependency graph.
  Parse response to find direct dependencies of 
  given service (both upstream and downstream).
  If API fails or returns empty, fallback to:
  DEPENDENCY_MAP = {
    "payments-api": ["checkout-service"],
    "checkout-service": ["payments-api"],
    "auth": ["payments-api"],
    "guestbook": ["auth", "checkout-service"],
  }
  Return List[str] of dependent service names.

In schemas.py add:
  DependencyHealth(service, risk_score, 
    risk_level, signals_used, triggered_factors[])
  DependencyRiskResponse(service, env,
    overall_risk_level, worst_dependency_score,
    dependencies[], proceed, 
    dependency_source: "datadog" or "fallback")

In app.py add GET /dependency-risk/{service}?env=prod:
  1. deps = await dd_client.get_service_dependencies(
     service, env)
  2. For each dep in deps:
     try get_ec2_signals(dep, env)
     score = model.rule_based_score(features)
     triggered = [f for f in factors if f.triggered]
     append DependencyHealth(dep, score, risk_level,
       signals_used, triggered)
     skip silently if DatadogLookupError
  3. If no deps found return empty dependencies[]
     with overall_risk_level: "unknown"
  4. worst_score = max(d.risk_score for d in deps)
     if deps else 0.0
  5. overall_risk: >=0.8=high, >=0.5=medium, else low
  6. proceed = True only if overall_risk == "low"
  7. Return DependencyRiskResponse with 
     dependency_source indicating if real or fallback

Validate no other code changes ok 
unknown-service?env=production
→ returns empty dependencies[], 
  overall_risk_level: "unknown", no crash.
