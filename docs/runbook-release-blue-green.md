# Runbook – Blue/Green Release for Online Banking Portal (Elastic Beanstalk)

## 1. Purpose

This runbook describes how to safely perform a blue/green release of the online banking portal hosted on AWS Elastic Beanstalk, minimizing downtime and reducing risk to critical customer channels. 

---

## 2. Scope and Prerequisites

- **Scope:** Application and configuration releases for the online banking web portal.
- **Environments:**
  - **Blue environment:** Current production environment serving live users.
  - **Green environment:** New version to be validated and then promoted to production.
- **Prerequisites:**
  - Route 53 DNS or CNAME pointing to the blue environment.
  - CI/CD pipeline (e.g., GitHub Actions / CodePipeline) capable of deploying to Beanstalk. 
  - CloudWatch dashboards and alarms in place (availability, latency, error rate). 

---

## 3. Release Criteria

Before starting:

- All critical alarms for the blue environment are in an OK state.
- No active Sev‑1/Sev‑2 incidents affecting the portal.
- Change ticket (if used) is approved with:
  - Version to deploy.
  - Back‑out plan (revert to blue).
  - Change window and communication plan.

---

## 4. High‑Level Steps

1. Deploy new version to the green environment.
2. Validate green with smoke tests and basic load testing.
3. Swap traffic (CNAME/URL) from blue to green.
4. Monitor production metrics.
5. Either decommission or keep blue as fallback for a defined period.

---

## 5. Detailed Procedure

### 5.1 Prepare the green environment

1. In Elastic Beanstalk console, create or update the **green** environment:
   - Same configuration as blue (instance type, scaling, VPC, security groups, DB access).
   - Ensure the environment is using the same RDS instance or a clone, according to your design.
2. Deploy the new application version to green:
   - Trigger pipeline or upload the version bundle (ZIP/Container image).
   - Wait for deployment to complete and environment health to turn **Green**.

### 5.2 Validation of green (pre‑cutover)

1. **Smoke tests**
   - Access the green environment URL (e.g., `https://green.example.com`).
   - Verify:
     - Home/dashboard page loads.
     - Login (with test accounts) works.
     - Basic navigation and key APIs (account summary, transactions) respond correctly.

2. **Functional checks**
   - Run your automated smoke/functional test suite (if available) against green.
   - Verify environment variables, feature flags, and configuration are correct.

3. **Performance checks (light load)**
   - Run a short, low‑volume load test (e.g., 5–10 minutes, small RPS) to:
     - Check p95 latency for key endpoints.
     - Confirm CPU/memory metrics remain within acceptable ranges.

4. **Review metrics and logs**
   - Confirm no unusual spikes in errors or resource usage on the green environment:
     - ALB 4xx/5xx
     - Application exceptions in CloudWatch Logs
     - RDS/DynamoDB metrics

If validation fails, **stop here**, fix issues, and redeploy to green until validation passes.

---

## 6. Traffic Cutover (Blue → Green)

### 6.1 DNS / CNAME swap

1. In Elastic Beanstalk console:
   - Use **Swap Environment URLs** to swap the CNAMEs of blue and green; or
2. In Route 53:
   - Update the alias/CNAME of the production hostname (e.g., `onlinebanking.example.com`) to point from blue to green.

### 6.2 Post‑cutover validation (first 15–30 minutes)

Immediately after cutover:

1. Confirm that:
   - Production hostname now resolves to green.
   - Synthetic tests (if configured) are targeting green and passing.
2. Monitor:
   - ALB metrics (latency, 4xx/5xx, request count).
   - Environment health of green (must remain **Green**).
   - Key business metrics as available (e.g., login success rate).
3. Perform quick manual checks:
   - Login with test account through the **production** URL.
   - Verify basic portal flows.

If metrics remain healthy, proceed to the observation window.

---

## 7. Observation Window and Rollback

### 7.1 Observation window

Define an observation window (for example, 1–2 hours) where:

- Green is serving all production traffic.
- Blue environment remains available as a standby.

Continuously monitor:

- Error rate and latency vs historical baselines.
- Environment health and any anomalous logs.

### 7.2 Rollback procedure (Green → Blue)

Trigger rollback if:

- Sustained increase in 5xx errors or major functional issues.
- Latency or error metrics breach agreed thresholds and no quick fix is available.

Steps:

1. Announce rollback in the incident/change channel.
2. Swap CNAME/URL back from green to blue using the same mechanism as cutover.
3. Monitor blue environment metrics to confirm recovery.
4. Mark the release as **failed** in change records and log root cause investigation items.

---

## 8. Post‑Release Activities

### 8.1 If release is successful

1. Update change record with:
   - Deployment time, validation results, metrics during observation.
2. Decide on blue environment:
   - Decommission after a safe period (e.g., 24–48 hours) or repurpose as the new staging environment.
3. Tag and document:
   - Application version and configuration used in this successful release.

### 8.2 If release failed and rollback was executed

1. Conduct a post‑incident / post‑release review:
   - Identify whether failure was due to code, config, capacity, or process.
2. Plan corrective actions:
   - Additional tests, canary strategy, or better pre‑prod validation.
3. Update this runbook to capture lessons learned.

---

## 9. References

- AWS Elastic Beanstalk deployment strategies and best practices. 
- AWS Well‑Architected Framework – change management and operations KPIs.  
- Deploying from GitHub to Elastic Beanstalk with CI/CD. 

