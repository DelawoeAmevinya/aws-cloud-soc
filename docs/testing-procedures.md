# SOC Testing & Validation Procedures

## Overview

This document outlines all tests performed to validate the SOC's detection and alerting capabilities.

---

## Test 1: Failed Authentication Detection

### Objective
Validate that the SOC detects and alerts on brute force login attempts.

### Prerequisites
- CloudWatch alarm `SOC-FailedConsoleLogins` configured
- Metric filter for failed logins active
- SNS subscription confirmed

### Test Steps

1. Create a test IAM user (or use existing)
2. Note the current time
3. Attempt to log in to AWS Console with **incorrect password**
4. Repeat 3 times (trigger threshold)
5. Wait 5 minutes

### Expected Results

✅ CloudWatch alarm enters ALARM state  
✅ Email notification received within 2 minutes  
✅ CloudTrail logs all failed attempts  
✅ CloudWatch Logs Insights query returns failures  

### Validation Query
```
fields @timestamp, userIdentity.principalId, sourceIPAddress, errorMessage
| filter eventName = "ConsoleLogin" and errorMessage = "Failed authentication"
| sort @timestamp desc
| limit 20
```

### Actual Results
- [ ] Detection time: _____ seconds
- [ ] Email received: Yes / No
- [ ] CloudTrail logged: Yes / No
- [ ] Query successful: Yes / No

**Status:** ⬜ PASS ⬜ FAIL

---

## Test 2: Security Group Modification Detection

### Objective
Verify detection of firewall rule changes.

### Prerequisites
- CloudWatch alarm `SOC-SecurityGroupChanges` configured
- Metric filter for security group changes active
- At least one security group exists

### Test Steps

1. Navigate to EC2 → Security Groups
2. Select any security group
3. Click **Edit inbound rules**
4. Add a new rule (any port, any source)
5. Save changes
6. Wait 2 minutes
7. Delete the rule (cleanup)

### Expected Results

✅ CloudWatch alarm triggers  
✅ Email notification received  
✅ CloudTrail captures the change  
✅ Event details include user and resource  

### Validation Query
```
fields @timestamp, userIdentity.principalId, eventName, requestParameters.groupId
| filter eventName like /SecurityGroup/
| sort @timestamp desc
| limit 10
```

### Actual Results
- [ ] Detection time: _____ seconds
- [ ] Email received: Yes / No
- [ ] User identified: Yes / No
- [ ] Resource ID captured: Yes / No

**Status:** ⬜ PASS ⬜ FAIL

---

## Test 3: GuardDuty Threat Detection

### Objective
Validate end-to-end alert pipeline for threat intelligence.

### Prerequisites
- GuardDuty enabled
- EventBridge rules configured
- SNS topics subscribed

### Test Steps

1. Navigate to GuardDuty console
2. Click **Settings**
3. Scroll to **Sample findings**
4. Click **Generate sample findings**
5. Wait 3 minutes
6. Check email inbox

### Expected Results

✅ 80+ sample findings generated  
✅ EventBridge rules triggered  
✅ SNS notifications sent  
✅ Emails delivered (100% delivery rate)  
✅ High severity routed to critical alerts  

### Findings to Verify

Check that these finding types were generated:
- [ ] Backdoor:EC2/C&CActivity.B
- [ ] CryptoCurrency:EC2/BitcoinTool.B
- [ ] UnauthorizedAccess:EC2/SSHBruteForce
- [ ] Trojan:EC2/DriveBySourceTraffic!DNS
- [ ] Execution:Kubernetes/AnomalousBehavior

### Actual Results
- [ ] Findings generated: _____ count
- [ ] Emails received: _____ count
- [ ] Processing time: _____ minutes
- [ ] Delivery success rate: _____%

**Status:** ⬜ PASS ⬜ FAIL

---

## Test 4: Root Account Usage Detection

### Objective
Verify critical alert for root account activity.

### Prerequisites
- CloudWatch alarm `SOC-RootAccountUsage` configured
- SNS configured for critical alerts

### Test Steps

**⚠️ WARNING: Use root account carefully**

1. Log in with root account credentials
2. Perform any action (e.g., view S3 buckets)
3. Log out
4. Wait 5 minutes
5. Check for alert

### Expected Results

✅ Alarm triggers immediately  
✅ Email sent to critical alerts channel  
✅ CloudTrail logs root activity  
✅ Action details captured  

### Validation Query
```
fields @timestamp, eventName, sourceIPAddress
| filter userIdentity.type = "Root"
| sort @timestamp desc
| limit 10
```

### Actual Results
- [ ] Alarm triggered: Yes / No
- [ ] Critical alert sent: Yes / No
- [ ] Activity logged: Yes / No

**Status:** ⬜ PASS ⬜ FAIL

---

## Test 5: IAM Policy Modification Detection

### Objective
Detect privilege escalation attempts.

### Prerequisites
- CloudWatch alarm `SOC-IAMPolicyChanges` configured

### Test Steps

1. Navigate to IAM → Policies
2. Create a new policy (any permissions)
3. Save the policy
4. Wait 5 minutes
5. Delete the policy (cleanup)

### Expected Results

✅ Alarm triggers on policy creation  
✅ Email notification received  
✅ Policy details captured in logs  

### Validation Query
```
fields @timestamp, userIdentity.principalId, eventName
| filter eventName like /Policy/
| sort @timestamp desc
| limit 10
```

### Actual Results
- [ ] Creation detected: Yes / No
- [ ] Email received: Yes / No
- [ ] Policy name captured: Yes / No

**Status:** ⬜ PASS ⬜ FAIL

---

## Test 6: VPC Flow Logs Analysis

### Objective
Verify network traffic monitoring and analysis.

### Prerequisites
- VPC Flow Logs enabled
- CloudWatch Logs receiving flow data

### Test Steps

1. Wait 10 minutes for flow data to accumulate
2. Navigate to CloudWatch → Logs Insights
3. Select log group: `/aws/vpc/flowlogs`
4. Run rejected traffic query
5. Verify data is present

### Expected Results

✅ Flow logs streaming to CloudWatch  
✅ Data queryable via Insights  
✅ Rejected connections identified  
✅ Source/destination details available  

### Validation Query
```
fields @timestamp, srcAddr, dstAddr, dstPort, action
| filter action = "REJECT"
| stats count() by srcAddr, dstPort
| sort count desc
| limit 20
```

### Actual Results
- [ ] Logs present: Yes 
- [ ] Query successful: Yes
- [ ] Rejected traffic found: Yes 

**Status:** ⬜ PASS ⬜ FAIL
pass
---

## Test 7: AWS Config Compliance

### Objective
Validate configuration compliance monitoring.

### Prerequisites
- AWS Config enabled
- Config rules deployed

### Test Steps

1. Navigate to AWS Config → Rules
2. Review compliance status
3. Identify any non-compliant resources
4. Remediate if possible
5. Verify rule re-evaluation

### Expected Results

✅ All rules evaluating resources  
✅ Compliance status visible  
✅ Non-compliant resources identified  
✅ Security Hub integration working  

### Rules to Check

- [ ] root-account-mfa-enabled
- [ ] iam-user-mfa-enabled
- [ ] s3-bucket-public-read-prohibited
- [ ] restricted-ssh
- [ ] encrypted-volumes

### Actual Results
- [ ] Rules active: _yes____ / _____
- [ ] Compliant resources: _100____%
- [ ] Security Hub updated: Yes 

**Status:** ⬜ PASS ⬜ FAIL

---

## Performance Benchmarks

### Detection Time Goals

| Event Type | Target | Actual |
|------------|--------|--------|
| Failed logins | < 2 min | __2___ |
| Security group changes | < 1 min | __1___ |
| GuardDuty findings | < 2 min | ___2__ |
| Root account usage | < 1 min | ____1_ |
| IAM changes | < 2 min | __2___ |

### Alert Delivery Goals

| Metric | Target | Actual |
|--------|--------|--------|
| Email delivery rate | 100% | _100___% |
| False positive rate | < 5% | __5__% |
| Alert completeness | 100% | _100___% |

---

## Test Summary

**Total Tests:** 7  
**Passed:** ___100%__  
**Failed:** ____0%_  
**Pass Rate:** ___100%__%  

**Overall Status:** ⬜ ALL TESTS PASSED ⬜ ISSUES FOUND

---

## Issues & Resolutions

### Issue 1
**Description:**  
**Resolution:**  
**Status:**  

### Issue 2
**Description:**  
**Resolution:**  
**Status:**  

---

## Recommendations

Based on testing:
- [ ] Adjust alarm thresholds if needed
- [ ] Fine-tune metric filter patterns
- [ ] Add additional detection rules
- [ ] Document false positives
- [ ] Schedule regular testing (monthly)

---
