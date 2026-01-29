# CloudWatch Logs Insights Queries

## Authentication & Access

### Failed Console Logins
```
fields @timestamp, userIdentity.principalId, eventName, sourceIPAddress, errorMessage
| filter eventName = "ConsoleLogin" and errorMessage = "Failed authentication"
| sort @timestamp desc
| limit 20
```

### Successful Console Logins
```
fields @timestamp, userIdentity.principalId, sourceIPAddress
| filter eventName = "ConsoleLogin" and errorMessage = "Success"
| sort @timestamp desc
| limit 50
```

### Root Account Activity
```
fields @timestamp, eventName, sourceIPAddress, requestParameters
| filter userIdentity.type = "Root"
| sort @timestamp desc
| limit 50
```

## API Activity

### Unauthorized API Calls
```
fields @timestamp, userIdentity.principalId, eventName, errorMessage, requestParameters
| filter errorCode like /Unauthorized|AccessDenied/
| sort @timestamp desc
| limit 50
```

### Top API Callers
```
fields userIdentity.principalId, eventName
| stats count() as callCount by userIdentity.principalId
| sort callCount desc
| limit 20
```

## Security Changes

### Security Group Modifications
```
fields @timestamp, userIdentity.principalId, eventName, requestParameters.groupId
| filter eventName like /SecurityGroup/
| sort @timestamp desc
| limit 20
```

### IAM Policy Changes
```
fields @timestamp, userIdentity.principalId, eventName, requestParameters
| filter eventName like /Policy/
| sort @timestamp desc
| limit 20
```

### IAM User/Role Creation
```
fields @timestamp, userIdentity.principalId, eventName, requestParameters
| filter eventName = "CreateUser" or eventName = "CreateRole"
| sort @timestamp desc
| limit 50
```

## Resource Changes

### EC2 Instance Launches
```
fields @timestamp, userIdentity.principalId, requestParameters.instanceType
| filter eventName = "RunInstances"
| sort @timestamp desc
| limit 20
```

### S3 Bucket Changes
```
fields @timestamp, userIdentity.principalId, eventName, requestParameters.bucketName
| filter eventSource = "s3.amazonaws.com"
| sort @timestamp desc
| limit 50
```

## Network Analysis

### VPC Flow Logs - Rejected Connections
```
fields @timestamp, srcAddr, dstAddr, dstPort, action
| filter action = "REJECT"
| stats count() as rejectedCount by srcAddr, dstPort
| sort rejectedCount desc
| limit 20
```

### VPC Flow Logs - Top Talkers
```
fields srcAddr, dstAddr, bytes
| stats sum(bytes) as totalBytes by srcAddr
| sort totalBytes desc
| limit 20
```

### SSH Connection Attempts
```
fields @timestamp, srcAddr, dstAddr, dstPort, action
| filter dstPort = 22
| sort @timestamp desc
| limit 50
```

### RDP Connection Attempts
```
fields @timestamp, srcAddr, dstAddr, dstPort, action
| filter dstPort = 3389
| sort @timestamp desc
| limit 50
```

## Error Analysis

### API Errors by Type
```
fields errorCode, errorMessage
| filter ispresent(errorCode)
| stats count() as errorCount by errorCode
| sort errorCount desc
| limit 20
```

### Failed API Calls by User
```
fields userIdentity.principalId, errorCode
| filter ispresent(errorCode)
| stats count() as failureCount by userIdentity.principalId
| sort failureCount desc
| limit 20
```

## Compliance & Auditing

### AWS Config Changes
```
fields @timestamp, configurationItem.resourceType, configurationItem.resourceId
| filter configurationItemDiff.changeType = "UPDATE"
| sort @timestamp desc
| limit 50
```

### KMS Key Usage
```
fields @timestamp, userIdentity.principalId, eventName
| filter eventSource = "kms.amazonaws.com"
| sort @timestamp desc
| limit 50
```
