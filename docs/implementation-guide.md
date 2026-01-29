# AWS Cloud SOC - Complete Implementation Guide

## Prerequisites

- AWS Account with administrative access
- Basic understanding of AWS services
- Email address for alert notifications

**Estimated Time:** 2-3 hours  
**Estimated Cost:** $25-50/month

---

## Phase 1: Enable CloudTrail (15 minutes)

### Step 1: Create CloudTrail

1. Sign in to AWS Console
2. Navigate to **CloudTrail** service
3. Click **Create trail**

**Configuration:**
- Trail name: `soc-audit-trail`
- Storage location: Create new S3 bucket
- S3 bucket name: `soc-cloudtrail-logs-[your-account-id]`
- Log file validation: ✅ Enabled
- CloudWatch Logs: ✅ Enabled
  - Log group: `CloudTrail/SOC-Logs`
  - IAM Role: Create new
- Management events: ✅ Read and Write
- Insights events: ✅ Enabled

4. Click **Create trail**

---

## Phase 2: Enable GuardDuty (5 minutes)

### Step 2: Activate GuardDuty

1. Navigate to **GuardDuty**
2. Click **Get Started** → **Enable GuardDuty**
3. In Settings, enable:
   - S3 Protection
   - EKS Protection (if using Kubernetes)
   - Malware Protection

---

## Phase 3: Configure AWS Config (10 minutes)

### Step 3: Set Up AWS Config

1. Navigate to **AWS Config**
2. Click **Get started**

**Configuration:**
- Resource types: All resources in this region
- Include global resources: ✅ Enabled
- S3 bucket: Create new `soc-config-logs-[account-id]`
- SNS topic: Skip (we'll use EventBridge)
- AWS Config role: Use existing `AWSServiceRoleForConfig`

3. Click **Confirm**

### Step 4: Add Config Rules

Add these rules:
- `root-account-mfa-enabled`
- `iam-user-mfa-enabled`
- `s3-bucket-public-read-prohibited`
- `s3-bucket-public-write-prohibited`
- `restricted-ssh`
- `encrypted-volumes`

---

## Phase 4: Enable VPC Flow Logs (10 minutes)

### Step 5: Create Log Group

1. Navigate to **CloudWatch** → **Log groups**
2. Click **Create log group**
- Name: `/aws/vpc/flowlogs`

### Step 6: Create IAM Role

1. Navigate to **IAM** → **Roles** → **Create role**
2. Trusted entity: VPC - Flow Logs
3. Role name: `VPCFlowLogsRole`
4. Attach policy: `CloudWatchLogsFullAccess`

### Step 7: Enable Flow Logs

1. Navigate to **VPC** → **Your VPCs**
2. Select your VPC → **Actions** → **Create flow log**

**Configuration:**
- Name: `soc-vpc-flow-logs`
- Filter: All
- Interval: 1 minute
- Destination: CloudWatch Logs
- Log group: `/aws/vpc/flowlogs`
- IAM role: `VPCFlowLogsRole`

---

## Phase 5: Enable Security Hub (5 minutes)

### Step 8: Activate Security Hub

1. Navigate to **Security Hub**
2. Click **Enable Security Hub**
3. Select standards:
   - AWS Foundational Security Best Practices
   - CIS AWS Foundations Benchmark

---

## Phase 6: Set Up Alerting (20 minutes)

### Step 9: Create SNS Topics

1. Navigate to **SNS** → **Topics** → **Create topic**

**Create 3 topics:**
- `SOC-CriticalAlerts` (for high severity)
- `SOC-SecurityAlerts` (for medium severity)
- `SOC-ComplianceAlerts` (for Config violations)

For each topic:
2. Click **Create subscription**
3. Protocol: Email
4. Endpoint: your-email@example.com
5. **Confirm subscription in your email**

### Step 10: Create CloudWatch Metric Filters

1. Navigate to **CloudWatch** → **Logs** → **Log groups**
2. Select `CloudTrail/SOC-Logs`
3. Click **Actions** → **Create metric filter**

**Create filters for:**

**Filter 1: Unauthorized API Calls**
- Pattern: `{ ($.errorCode = "*UnauthorizedOperation") || ($.errorCode = "AccessDenied*") }`
- Metric name: `UnauthorizedAPICallCount`
- Namespace: `SOC/Security`

**Filter 2: Root Account Usage**
- Pattern: `{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }`
- Metric name: `RootAccountUsageCount`
- Namespace: `SOC/Security`

**Filter 3: Failed Console Logins**
- Pattern: `{ $.eventName = "ConsoleLogin" && $.errorMessage = "Failed authentication" }`
- Metric name: `FailedConsoleLoginCount`
- Namespace: `SOC/Security`

**Filter 4: IAM Policy Changes**
- Pattern: See full pattern in README
- Metric name: `IAMPolicyChangeCount`

**Filter 5: Security Group Changes**
- Pattern: See full pattern in README
- Metric name: `SecurityGroupChangeCount`

### Step 11: Create CloudWatch Alarms

For each metric filter:
1. Click **Create alarm**
2. Threshold: >= 1 (or >= 3 for failed logins)
3. Period: 5 minutes
4. SNS topic: `SOC-SecurityAlerts`
5. Alarm name: `SOC-[EventType]`

---

## Phase 7: Set Up EventBridge Rules (15 minutes)

### Step 12: Create GuardDuty Alert Rules

1. Navigate to **EventBridge** → **Rules** → **Create rule**

**Rule 1: High Severity Findings**
- Name: `SOC-GuardDuty-HighSeverity`
- Event pattern: Custom (JSON editor)
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [7, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7, 7.8, 7.9, 8, 8.1, 8.2, 8.3, 8.4, 8.5, 8.6, 8.7, 8.8, 8.9]
  }
}
```
- Target: SNS topic `SOC-CriticalAlerts`

**Rule 2: All GuardDuty Findings**
- Name: `SOC-GuardDuty-AllFindings`
- Event pattern:
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"]
}
```
- Target: SNS topic `SOC-SecurityAlerts`

---

## Phase 8: Create Dashboard (10 minutes)

### Step 13: Build CloudWatch Dashboard

1. Navigate to **CloudWatch** → **Dashboards** → **Create dashboard**
2. Name: `SOC-Security-Overview`
3. Add widgets:
   - Line graph: Security metrics
   - Number: Active GuardDuty findings
   - Log table: Recent unauthorized attempts
   - Number: Non-compliant resources

---

## Phase 9: Testing (30 minutes)

### Step 14: Validate Detection

**Test 1: Failed Logins**
- Attempt console login with wrong password 3+ times
- Verify email alert received

**Test 2: Security Group Change**
- Modify any security group
- Verify email alert received

**Test 3: GuardDuty**
- GuardDuty → Settings → Generate sample findings
- Verify email alerts received

---

## Verification Checklist

- [ ] CloudTrail logging to CloudWatch
- [ ] GuardDuty enabled and monitoring
- [ ] Config recording all resources
- [ ] VPC Flow Logs active
- [ ] Security Hub enabled with standards
- [ ] All SNS subscriptions confirmed
- [ ] CloudWatch alarms created and OK
- [ ] EventBridge rules enabled
- [ ] Dashboard displaying metrics
- [ ] All tests passed

**Congratulations! Your SOC is operational! 🎉**
