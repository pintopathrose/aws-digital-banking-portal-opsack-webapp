# Runbook – Login Outage (Digital Banking Portal)

## 1. Purpose

This runbook describes how to detect, triage, mitigate, and follow up on a login outage or severe login degradation for the online banking portal running on AWS Elastic Beanstalk with an Application Load Balancer and RDS/DynamoDB backend. 

---

## 2. Scope and Impact

- **Scope:** Web and mobile users accessing the online banking portal via Elastic Beanstalk.
- **Impact examples:**
  - High login failure rate (HTTP 5xx/4xx from `/login` endpoint).
  - Login latency above agreed SLO (for example, p95 > 2 seconds).
  - Complete inability to log in for a subset or all users.

---

## 3. Preconditions / Monitoring

### 3.1 Key alarms

These CloudWatch alarms should already exist:

- `ALB-5XX-Login-High`:  
  - Metric: `HTTPCode_Target_5XX_Count` for ALB target group on `/login`  
  - Threshold: > 1% of requests for 5 minutes  
- `Login-Latency-High`:  
  - Metric: ALB target group `TargetResponseTime` for `/login`  
  - Threshold: p95 > 2s for 5 minutes  
- `EB-Env-Health-Degraded`:  
  - Metric: Elastic Beanstalk `EnvironmentHealth` not `Green` for > 5 minutes  
- `RDS-Connectivity-Errors`:  
  - Metric: Application custom metric or error count in CloudWatch Logs related to DB connection failures

Alarms notify the operations team via Amazon SNS (email or chat integration). 

---

## 4. Incident Handling

### 4.1 Detection

1. Incident is triggered by one or more of the alarms above.
2. Confirm that the issue affects real users:
   - Check synthetic canary (if configured) or manually attempt login via portal.
   - Check status of other APIs/features to see if the issue is isolated to login.

### 4.2 Initial Triage (10–15 minutes)

Perform these steps in parallel where possible:

1. **Check high‑level health**
   - Open Elastic Beanstalk console and verify:
     - Environment health (Green/Yellow/Red).
     - Any recent failed deployments or configuration changes. 
   - Open ALB metrics:
     - `HTTPCode_Target_5XX_Count`
     - `HTTPCode_ELB_5XX_Count`
     - `TargetResponseTime`

2. **Review recent changes**
   - Check CI/CD pipeline (GitHub Actions / CodePipeline) for:
     - Recent deployments in last 60 minutes.
     - Configuration changes (environment variables, DB connection strings, security groups).

3. **Inspect application logs**
   - Use CloudWatch Logs Insights on application log group:
     - Filter on `/login` path and last 15–30 minutes.
     - Look for common errors (DB connection timeouts, authentication errors, external service failures).

4. **Check database health**
   - Open RDS (or DynamoDB) console:
     - CPU utilization, connections, free storage, read/write IOPS.
     - Any failover events or maintenance in progress. 

### 4.3 Classification

Quickly classify the incident:

- **Auth/logic issue:** Application or IDP misconfiguration, bad code deployment, failed integration.
- **Infrastructure issue:** ALB health failing, Beanstalk instances unhealthy, scaling problem.
- **Database issue:** RDS overload, connection pool exhaustion, DB outage.
- **Upstream dependency issue:** Identity provider, external API, or network dependency.

Document the current hypothesis in your incident record.

---

## 5. Mitigation Actions

### 5.1 If caused by a recent deployment

1. Stop/disable any in‑progress deployment to prevent further changes.
2. Roll back:
   - If using Elastic Beanstalk:
     - Revert to previous application version in Beanstalk console.
   - If using blue/green:
     - Swap DNS/CNAME back to the previous (blue) environment.
3. Monitor:
   - Verify ALB errors and login failures drop back to normal.
   - Confirm successful login via manual test.

### 5.2 If application instances are unhealthy

1. Check Beanstalk environment events for errors on instance provisioning or health checks.
2. Trigger environment rebuild or restart:
   - `Actions → Restart App Server(s)` or re‑deploy last known good version.
3. Confirm Auto Scaling is launching healthy instances across at least two Availability Zones.

### 5.3 If database is the bottleneck

1. Confirm symptomatic metrics:
   - High CPU, high connections, or throttling on RDS/DynamoDB.
2. Immediate optimizations:
   - Temporarily increase instance size or storage IOPS if within allowed limits.
   - Reduce connection pool size or enable connection reuse in application.
   - Enable on‑demand mode (for DynamoDB) or raise capacity units if using provisioned. 
3. Consider temporary rate limiting:
   - Use simple throttling (e.g., small WAF rule or app‑level backoff) for non‑critical APIs to protect login path.

### 5.4 If external dependency is failing

1. Identify the failing dependency from logs (IDP, fraud service, profile API, etc.).
2. Apply a fail‑open or graceful degradation path where acceptable:
   - Use cached responses or limited login flows when safe.
3. Coordinate with external provider / internal team responsible for that service.

---

## 6. Communication

1. **Internal**
   - Open or update an incident ticket with:
     - Start time, symptoms, impacted percentage of logins.
     - Current hypothesis and actions taken.
2. **Stakeholders**
   - Send short updates to channel owners (e.g., Digital Channels / Customer Experience teams) at agreed intervals (e.g., every 15–30 minutes).
3. **Closure**
   - Document exact fix and time of resolution.

---

## 7. Post‑Incident Review

Within 24–72 hours:

1. Confirm root cause (config, code, capacity, dependency).
2. Identify and implement permanent fixes:
   - Additional alarms, better validation, dependency timeouts, retry logic.
3. Update:
   - This runbook with new checks or steps.
   - Dashboards and SLO/SLA documentation if needed.

---

## 8. References

- AWS Well‑Architected Framework – Operations excellence and reliability pillars.
- RDS integration with Elastic Beanstalk. [web:74]

