# VPC Flow Logs Analysis Queries

## Security Analysis

### All Rejected Traffic
```
fields @timestamp, srcAddr, dstAddr, srcPort, dstPort, protocol, action
| filter action = "REJECT"
| sort @timestamp desc
| limit 100
```

### Rejected Traffic by Source IP
```
fields srcAddr, dstPort, protocol
| filter action = "REJECT"
| stats count() as rejectedCount by srcAddr, dstPort
| sort rejectedCount desc
| limit 50
```

### Rejected Traffic by Destination Port
```
fields dstPort, protocol
| filter action = "REJECT"
| stats count() as attempts by dstPort, protocol
| sort attempts desc
| limit 20
```

## Network Traffic Analysis

### Top Source IPs by Traffic Volume
```
fields srcAddr, bytes
| stats sum(bytes) as totalBytes by srcAddr
| sort totalBytes desc
| limit 20
```

### Top Destination IPs by Traffic Volume
```
fields dstAddr, bytes
| stats sum(bytes) as totalBytes by dstAddr
| sort totalBytes desc
| limit 20
```

### Top Destination Ports
```
fields dstPort, protocol
| stats count() as connectionCount by dstPort, protocol
| sort connectionCount desc
| limit 20
```

## Suspicious Activity Detection

### Port Scanning Detection (Multiple Ports from Single Source)
```
fields srcAddr, dstPort
| filter action = "REJECT"
| stats count_distinct(dstPort) as uniquePorts by srcAddr
| filter uniquePorts > 10
| sort uniquePorts desc
```

### High Volume Rejected Connections (Potential DDoS)
```
fields srcAddr
| filter action = "REJECT"
| stats count() as rejectedCount by srcAddr, bin(5m)
| filter rejectedCount > 100
| sort rejectedCount desc
```

### SSH Brute Force Detection
```
fields @timestamp, srcAddr, dstAddr, action
| filter dstPort = 22
| stats count() as attempts by srcAddr, dstAddr
| filter attempts > 10
| sort attempts desc
```

### RDP Brute Force Detection
```
fields @timestamp, srcAddr, dstAddr, action
| filter dstPort = 3389
| stats count() as attempts by srcAddr, dstAddr
| filter attempts > 10
| sort attempts desc
```

## Application Analysis

### Web Traffic Analysis (HTTP/HTTPS)
```
fields srcAddr, dstAddr, dstPort, bytes
| filter dstPort = 80 or dstPort = 443
| stats sum(bytes) as totalBytes, count() as requests by dstAddr
| sort totalBytes desc
| limit 20
```

### Database Connection Analysis
```
fields srcAddr, dstAddr, dstPort
| filter dstPort = 3306 or dstPort = 5432 or dstPort = 1433
| stats count() as connectionCount by srcAddr, dstPort
| sort connectionCount desc
```

### DNS Query Analysis
```
fields srcAddr, dstAddr
| filter dstPort = 53
| stats count() as queryCount by srcAddr
| sort queryCount desc
| limit 20
```

## Performance & Troubleshooting

### Connections by Protocol
```
fields protocol
| stats count() as connectionCount by protocol
| sort connectionCount desc
```

### Traffic by 5-Minute Intervals
```
fields bytes, action
| stats sum(bytes) as totalBytes by action, bin(5m)
| sort bin(5m) desc
```

### Accepted vs Rejected Traffic Ratio
```
fields action
| stats count() as flowCount by action
```

## Geographic Analysis (Requires IP enrichment)

### Traffic by Country (if using enriched logs)
```
fields srcAddr, country
| stats count() as connectionCount by country
| sort connectionCount desc
| limit 20
```
