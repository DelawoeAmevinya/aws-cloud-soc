# Incident Response Playbook: Unauthorized Access

## Alert Trigger
**CloudWatch Alarm:** `SOC-UnauthorizedAPICalls`  
**Severity:** Medium to High

---

## Detection Indicators

- AccessDenied errors in CloudTrail
- UnauthorizedOperation errors
- Multiple failed API calls from same user/IP
- Attempts to access restricted resources

---

## Initial Response (0-5 minutes)

### Step 1: Verify the Alert

- [ ] Check CloudWatch alarm details
- [ ] Review CloudTrail event in Logs Insights
- [ ] Identify: Who, What, When, Where

**Query to run:**
```
fields @timestamp, userIdentity.principalId, eventName, sourceIPAddress, errorCode, errorMessage
| filter errorCode like /Unauthorized|AccessDenied/
| sort @timestamp desc
| limit 50
```

**Document:**
- User/Role: _____
- Resource attempted: _____
- Source IP: _____
- Time: _____
- Number of attempts: _____

---

### Step 2: Assess Severity

**Low Risk Indicators:**
- Single AccessDenied from known user
- User has legitimate reason to access
- Source IP matches known location

**High Risk Indicators:**
- Multiple different resources attempted
- Unknown source IP
- After-hours access
- Privilege escalation attempts
- Sensitive resources (IAM, KMS, Secrets Manager)

**Severity Assessment:** ⬜ Low ⬜ Medium ⬜ High ⬜ Critical

---

## Investigation Phase (5-30 minutes)

### Step 3: User/Role Analysis

- [ ] Check user's normal activity pattern
- [ ] Review recent successful API calls
- [ ] Verify if user's credentials may be compromised
- [ ] Check for recent password/key changes

**Query:**
```
fields @timestamp, eventName, errorCode
| filter userIdentity.principalId = "USER_ARN_HERE"
| sort @timestamp desc
| limit 100
```

---

### Step 4: Source IP Investigation

- [ ] Check IP reputation (VirusTotal, AbuseIPDB)
- [ ] Verify if IP matches user's known locations
- [ ] Check for other activity from this IP
- [ ] Review VPC Flow Logs for network context

**IP Reputation Check:**
- VirusTotal: https://www.virustotal.com/gui/ip-address/[IP]
- AbuseIPDB: https://www.abuseipdb.com/check/[IP]

**Findings:** _____

---

### Step 5: Scope Assessment

- [ ] Check if other users affected
- [ ] Review similar attempts in time window
- [ ] Check GuardDuty for related findings
- [ ] Review Security Hub for correlations

**Query for similar activity:**
```
fields userIdentity.principalId, eventName, sourceIPAddress
| filter errorCode like /Unauthorized|AccessDenied/
| filter @timestamp > ago(24h)
| stats count() by userIdentity.principalId, sourceIPAddress
| sort count desc
```

---

## Containment Actions (30-60 minutes)

### Step 6: If Compromise Suspected

**For IAM User:**
- [ ] Disable access keys immediately
  - IAM → Users → [User] → Security credentials → Deactivate
- [ ] Reset password if console access
- [ ] Revoke active sessions
- [ ] Review and remove any suspicious permissions

**For IAM Role:**
- [ ] Identify role assumption source
- [ ] Modify trust policy to restrict access
- [ ] Revoke active sessions
- [ ] Check for privilege escalation

**For EC2 Instance (if instance profile):**
- [ ] Isolate instance (change security group)
- [ ] Stop instance if necessary
- [ ] Snapshot volumes for forensics
- [ ] Review instance for compromise

---

### Step 7: Limit Exposure

- [ ] Review and restrict affected resource permissions
- [ ] Enable MFA if not already enabled
- [ ] Update IAM policies to enforce MFA
- [ ] Block malicious IP addresses (if external)

**Commands (AWS CLI if needed):**
```bash
# Deactivate access key
aws iam update-access-key --user-name [USER] --access-key-id [KEY] --status Inactive

# Delete access key (if confirmed compromised)
aws iam delete-access-key --user-name [USER] --access-key-id [KEY]
```

---

## Remediation (1-4 hours)

### Step 8: Credential Rotation

- [ ] Rotate all potentially compromised credentials
- [ ] Update applications with new credentials
- [ ] Store new credentials in Secrets Manager
- [ ] Document rotation in incident log

---

### Step 9: Permission Review

- [ ] Apply principle of least privilege
- [ ] Remove unnecessary permissions
- [ ] Implement more restrictive resource policies
- [ ] Add condition keys (IP, MFA, time-based)

**Example restrictive policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "BoolIfExists": {"aws:MultiFactorAuthPresent": "false"}
    }
  }]
}
```

---

### Step 10: Monitoring Enhancement

- [ ] Add specific CloudWatch alarm for this user/resource
- [ ] Create custom GuardDuty threat list if applicable
- [ ] Update detection rules based on findings
- [ ] Implement additional logging if needed

---

## Recovery & Prevention

### Step 11: Restore Normal Operations

- [ ] Verify legitimate access restored
- [ ] Test user/application functionality
- [ ] Monitor for 24-48 hours for issues
- [ ] Remove temporary restrictions if safe

---

### Step 12: Preventive Measures

- [ ] Enforce MFA for all users
- [ ] Implement IP whitelisting where possible
- [ ] Use AWS Organizations SCPs for guardrails
- [ ] Regular access reviews
- [ ] Security awareness training

**Preventive Controls Checklist:**
- [ ] MFA enforced
- [ ] Jan 27
 Least privilege implemented
 Session duration limits set
 IP restrictions applied
 Logging and monitoring enhanced
Documentation
Step 13: Incident Report
Incident ID: _____
Date/Time: _____
Reporter: _____
Severity: _____

Summary:

Timeline:

Detection: _____
Investigation started: _____
Containment: _____
Resolution: _____
Root Cause:

Impact:

Actions Taken:

Lessons Learned:

Follow-up Actions:

 _____
 _____
Post-Incident Activities
Step 14: Review and Improve
 Update runbook based on experience
 Share lessons with team
 Update detection rules
 Schedule follow-up review (30 days)
Review Date: _____
Attendees: _____
Improvements Identified: _____

Escalation Criteria
Escalate to senior security if:

Multiple accounts compromised
Data exfiltration suspected
Privilege escalation successful
Regulatory compliance impact
Unable to contain within 1 hour
Requires legal/PR involvement
Escalation Contact: _____
Escalation Method: _____

