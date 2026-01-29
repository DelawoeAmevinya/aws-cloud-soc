Part 2: AWS Decommission Guide

## Complete AWS Resource Teardown Guide

# AWS Cloud SOC - Decommission Guide

**⚠️ WARNING:** This guide will completely tear down all SOC resources and stop all monitoring and alerting.

**Estimated Time:** 30-45 minutes  
**Cost Savings:** ~$25-50/month

---

## Pre-Decommission Checklist

Before you begin:
- [ ] Export any logs you want to keep
- [ ] Document any important findings
- [ ] Take final screenshots if needed
- [ ] Verify you have the implementation guide if you want to rebuild later

---

## Phase 1: Disable Real-Time Alerting (5 minutes)

### Step 1: Disable EventBridge Rules

1. Navigate to **EventBridge** → **Rules**
2. For each rule:
   - Select `SOC-GuardDuty-HighSeverity`
   - Click **Disable**
   - Select `SOC-GuardDuty-AllFindings`
   - Click **Disable**

**Result:** No more GuardDuty email alerts

---

### Step 2: Disable CloudWatch Alarms

1. Navigate to **CloudWatch** → **Alarms** → **All alarms**
2. Select all SOC alarms:
   - `SOC-UnauthorizedAPICalls`
   - `SOC-RootAccountUsage`
   - `SOC-FailedConsoleLogins`
   - `SOC-IAMPolicyChanges`
   - `SOC-SecurityGroupChanges`
   - `SOC-High-Rejected-Traffic`
3. Click **Actions** → **Disable**

**Result:** No more CloudWatch alarm emails

**Optional:** Delete alarms instead of disabling:
- Select alarms → **Actions** → **Delete**

---

## Phase 2: Stop Data Collection (10 minutes)

### Step 3: Disable GuardDuty

**⚠️ Note:** GuardDuty has a 30-day retention period after suspension

1. Navigate to **GuardDuty**
2. Click **Settings** (left menu)
3. Scroll to **Suspend GuardDuty**
4. Click **Suspend**
5. Type "suspend" to confirm

**Cost Impact:** Stops all GuardDuty charges immediately

**To permanently delete:**
1. After suspension, go back to Settings
2. Click **Disable GuardDuty**
3. Wait 90 days for complete deletion (AWS requirement)

---

### Step 4: Stop AWS Config Recording

1. Navigate to **AWS Config**
2. Click **Settings** (left menu)
3. Click **Edit**
4. Toggle **Recording** to **Off**
5. Click **Save**

**Result:** Config stops recording resource changes

**Optional - Delete Config Data:**
1. Go to **S3**
2. Find bucket: `soc-config-logs-[account-id]`
3. Empty bucket → Delete bucket

---

### Step 5: Disable VPC Flow Logs

1. Navigate to **VPC** → **Your VPCs**
2. Select your VPC
3. Click **Flow logs** tab
4. Select `soc-vpc-flow-logs`
5. Click **Actions** → **Delete flow log**
6. Confirm deletion

**Also delete subnet flow logs if created:**
1. Go to **Subnets**
2. Select each subnet → **Flow logs** tab
3. Delete any SOC flow logs

---

### Step 6: Disable Security Hub

1. Navigate to **Security Hub**
2. Click **Settings** (left menu)
3. Click **General**
4. Scroll to **Disable AWS Security Hub**
5. Type "DISABLE" to confirm
6. Click **Disable AWS Security Hub**

---

### Step 7: Stop CloudTrail Logging (OPTIONAL - READ CAREFULLY)

**⚠️ IMPORTANT:** Only disable CloudTrail if you created it specifically for this project. Many organizations require CloudTrail for compliance/auditing.

**To stop logging without deleting:**
1. Navigate to **CloudTrail** → **Trails**
2. Select `soc-audit-trail`
3. Click **Stop logging**

**To completely delete the trail:**
1. Select `soc-audit-trail`
2. Click **Delete**
3. Confirm deletion

**Delete CloudTrail S3 bucket:**
1. Navigate to **S3**
2. Find bucket: `soc-cloudtrail-logs-[account-id]`
3. Select bucket → **Empty**
4. After emptied, select bucket → **Delete**

---

## Phase 3: Clean Up Alerting Infrastructure (10 minutes)

### Step 8: Delete EventBridge Rules

1. Navigate to **EventBridge** → **Rules**
2. Select `SOC-GuardDuty-HighSeverity`
3. Click **Delete**
4. Repeat for `SOC-GuardDuty-AllFindings`

---

### Step 9: Delete CloudWatch Alarms

1. Navigate to **CloudWatch** → **Alarms**
2. Select all SOC alarms (Ctrl+Click or Cmd+Click):
   - All alarms starting with `SOC-`
3. Click **Actions** → **Delete**
4. Confirm deletion

---

### Step 10: Delete CloudWatch Metric Filters

1. Navigate to **CloudWatch** → **Logs** → **Log groups**
2. Select `CloudTrail/SOC-Logs`
3. Click **Metric filters** tab
4. For each filter:
   - Select filter
   - Click **Delete**
   - Confirm

---

### Step 11: Delete SNS Topics and Subscriptions

1. Navigate to **SNS** → **Subscriptions**
2. Select all SOC subscriptions (your email subscriptions)
3. Click **Delete**

4. Navigate to **SNS** → **Topics**
5. Select all SOC topics:
   - `SOC-CriticalAlerts`
   - `SOC-SecurityAlerts`
   - `SOC-ComplianceAlerts`
6. Click **Delete**
7. Type "delete me" to confirm

**Result:** No more email notifications

---

## Phase 4: Delete Logs and Storage (10 minutes)

### Step 12: Delete CloudWatch Log Groups

**⚠️ WARNING:** This permanently deletes all logs

1. Navigate to **CloudWatch** → **Logs** → **Log groups**
2. Select and delete these log groups:
   - `CloudTrail/SOC-Logs`
   - `/aws/vpc/flowlogs`
   - Any other SOC-related log groups
3. For each: Click **Actions** → **Delete log group(s)**
4. Confirm deletion

**Cost Impact:** Stops CloudWatch Logs storage charges

---

### Step 13: Delete S3 Buckets

1. Navigate to **S3**
2. Find and delete these buckets:
   - `soc-cloudtrail-logs-[account-id]`
   - `soc-config-logs-[account-id]`
   - Any other SOC-related buckets

**For each bucket:**
1. Select bucket
2. Click **Empty** (this deletes all objects first)
3. Confirm by typing bucket name
4. After empty, select bucket again
5. Click **Delete**
6. Confirm by typing bucket name

**⚠️ Note:** You cannot delete a bucket until it's completely empty

---

## Phase 5: Remove AWS Config Rules (5 minutes)

 Step 14: Delete Config Rules

1. Navigate to **AWS Config** → **Rules**
2. Select all rules (or select SOC-specific rules):
   - `root-account-mfa-enabled`
   - `iam-user-mfa-enabled`
   - `s3-bucket-public-read-prohibited`
   - `s3-bucket-public-write-prohibited`
   - `restricted-ssh`
   - `encrypted-volumes`
3. Click **Actions** → **Delete rule**
4. Confirm deletion

---

## Phase 6: Clean Up IAM Roles (Optional - 5 minutes)

### Step 15: Delete IAM Roles

**⚠️ CAUTION:** Only delete roles you created for this project

1. Navigate to **IAM** → **Roles**
2. Search for and delete these roles (if you created them):
   - `VPCFlowLogsRole`
   - `CloudTrailRoleForCloudWatchLogs` (if created)
   
**For each role:**
1. Select role
2. Click **Delete**
3. Confirm

**Note:** Do NOT delete:
- `AWSServiceRoleForConfig` (service-linked role, managed by AWS)
- `AWSServiceRoleForGuardDuty` (service-linked role, managed by AWS)
- Any roles you didn't create for this project

---

## Phase 7: Delete Dashboard (2 minutes)

### Step 16: Remove CloudWatch Dashboard

1. Navigate to **CloudWatch** → **Dashboards**
2. Select `SOC-Security-Overview`
3. Click **Delete**
4. Confirm deletion

---

## Phase 8: Optional - Disable Inspector (if enabled)

### Step 17: Disable Amazon Inspector

1. Navigate to **Amazon Inspector**
2. Click **Settings** → **General**
3. Click **Disable Amazon Inspector**
4. Confirm

---

## Verification Checklist

Verify everything is stopped/deleted:

### Services Disabled:
- [ ] GuardDuty suspended/disabled
- [ ] AWS Config recording stopped
- [ ] VPC Flow Logs deleted
- [ ] Security Hub disabled
- [ ] CloudTrail stopped (if applicable)
- [ ] Inspector disabled (if applicable)

### Alerting Removed:
- [ ] EventBridge rules deleted
- [ ] CloudWatch alarms deleted
- [ ] CloudWatch metric filters deleted
- [ ] SNS topics deleted
- [ ] SNS subscriptions deleted

### Storage Cleaned:
- [ ] CloudWatch log groups deleted
- [ ] S3 buckets emptied and deleted
- [ ] Config data removed

### Other Cleanup:
- [ ] Config rules deleted
- [ ] IAM roles deleted (custom only)
- [ ] CloudWatch dashboard deleted
- [ ] No more email alerts arriving

---

## Cost Verification

**Check your AWS Billing Dashboard 24-48 hours after decommission:**

1. Navigate to **Billing** → **Cost Explorer**
2. Filter by service:
   - GuardDuty should show $0 (after 24-48 hours)
   - Config should show $0
   - CloudWatch should be significantly reduced
   - S3 should be reduced (no more log storage)

**Expected savings:** ~$25-50/month

---

## If You Want to Rebuild Later

All configuration details are preserved in your GitHub repository:
- Implementation guide has step-by-step setup
- Config files have all patterns and rules
- Screenshots show the final result

**Time to rebuild:** ~2-3 hours

---

## Emergency Stop (Quick Disable)

**If you just want to stop alerts immediately without full teardown:**

1. **Disable EventBridge rules** (2 minutes)
   - EventBridge → Rules → Disable all SOC rules

2. **Disable CloudWatch alarms** (2 minutes)
   - CloudWatch → Alarms → Disable all SOC alarms

3. **Suspend GuardDuty** (1 minute)
   - GuardDuty → Settings → Suspend

**Total time:** 5 minutes  
**Result:** All alerts stopped, minimal ongoing costs

You can later complete full decommission when you have more time.

---

## Troubleshooting

### "Cannot delete S3 bucket - not empty"
**Solution:** 
1. Select bucket → Click **Empty**
2. Confirm deletion of all objects
3. Wait for completion
4. Then delete bucket

### "Cannot delete log group - in use"
**Solution:**
1. First delete all metric filters on the log group
2. Stop any services writing to it (CloudTrail, VPC Flow Logs)
3. Wait 5 minutes
4. Try deleting log group again

### "SNS topic has subscriptions"
**Solution:**
1. First delete all subscriptions
2. Then delete the topic

### "IAM role cannot be deleted - in use"
**Solution:**
1. Check which service is using it (Trust relationships tab)
2. Stop/delete that service first
3. Then delete the role

---

## Still Receiving Emails After Decommission?

### Check these:
1. **Email client cache:** Refresh your inbox
2. **SNS subscriptions:** Verify all deleted in SNS console
3. **EventBridge rules:** Confirm all disabled/deleted
4. **CloudWatch alarms:** Confirm all disabled/deleted
5. **GuardDuty:** Confirm suspended

### If emails continue:
1. Check the email sender/subject
2. It might be from a different AWS service
3. Go to SNS → Email preferences → Unsubscribe

---

## Final Notes

- **Logs are permanently deleted** - no recovery possible
- **GuardDuty findings expire** after 90 days naturally
- **No charges** after complete decommission (except minimal S3/CloudWatch if anything remains)
- **Re-enabling is easy** - just follow the implementation guide again

**Decommission Complete! 🎉**

Your AWS account is clean and you're no longer incurring SOC-related charges.
