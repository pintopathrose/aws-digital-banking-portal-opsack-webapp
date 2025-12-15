# Example CloudWatch Alarms for Digital Channels

This file describes example CloudWatch alarms you can create for the digital banking portal.  
You can configure these alarms in the CloudWatch console and route them to an SNS topic for email or chat notifications.

---

## 1. ALB 5xx error rate high

- **Name:** `ALB-5XX-High`
- **Metric namespace:** `AWS/ApplicationELB`
- **Metric:** `HTTPCode_Target_5XX_Count`
- **Dimensions:** `LoadBalancer = <your-ALB-name>`
- **Statistic:** Sum
- **Period:** 300 seconds (5 minutes)
- **Alarm condition:** > 1% of total requests result in 5xx over the period
- **Action:** Notify `digital-channels-alerts` SNS topic

---

## 2. Login latency high

- **Name:** `Login-Latency-High`
- **Metric namespace:** `AWS/ApplicationELB`
- **Metric:** `TargetResponseTime`
- **Dimensions:** `LoadBalancer = <your-ALB-name>`
- **Statistic:** p95
- **Period:** 300 seconds
- **Alarm condition:** p95 latency for `/login` > 2 seconds
- **Action:** Notify `digital-channels-alerts` SNS topic

> Note: You can either filter by target group for the login endpoint or configure a separate metric filter if needed.

---

## 3. Elastic Beanstalk environment health degraded

- **Name:** `EB-Env-Health-Degraded`
- **Metric namespace:** `AWS/ElasticBeanstalk`
- **Metric:** `EnvironmentHealth`
- **Dimensions:** `EnvironmentName = digital-channels-api-env` (and/or `digital-channels-web-env`)
- **Statistic:** Average
- **Period:** 300 seconds
- **Alarm condition:** Environment health not `Green` for more than 5 minutes
- **Action:** Notify `digital-channels-alerts` SNS topic

---

## 4. RDS CPU high

- **Name:** `RDS-CPU-High`
- **Metric namespace:** `AWS/RDS`
- **Metric:** `CPUUtilization`
- **Dimensions:** `DBInstanceIdentifier = <your-RDS-instance-id>`
- **Statistic:** Average
- **Period:** 300 seconds
- **Alarm condition:** CPUUtilization > 80% for 10 minutes
- **Action:** Notify `digital-channels-alerts` SNS topic

---

## 5. RDS connections high

- **Name:** `RDS-Connections-High`
- **Metric namespace:** `AWS/RDS`
- **Metric:** `DatabaseConnections`
- **Dimensions:** `DBInstanceIdentifier = <your-RDS-instance-id>`
- **Statistic:** Average
- **Period:** 300 seconds
- **Alarm condition:** Connections above a chosen safe threshold (for example, 80% of max) for 10 minutes
- **Action:** Notify `digital-channels-alerts` SNS topic

---

These alarms are examples. You can adjust thresholds and periods to match your environment and expected load.

