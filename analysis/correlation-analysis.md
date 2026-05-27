# Metrics Correlation Analysis

During the incident timeline (14:00 - 15:00 UTC), we observed a classic cascading failure pattern:
[Traffic Spike: 500 -> 1500 req/m]
│
▼
[DB Connection Pool Hits 100% (20/20)]
│
▼
[Application Threads Blocked in Queue] ───► [Host CPU Remains Low (45%)]
│
▼
[Latency Explodes to 5000ms (P95)] ───► [HTTP 5XX Timeouts Triggered]

### Key Correlated Observations:
1. **The Trigger:** The 3x traffic volume increase perfectly matches the timeline of the `TargetResponseTime` spike at 14:00.
2. **The Bottleneck:** While CPU and Memory graphs remained perfectly flat and healthy, the `DBConnectionPoolUsage` slammed into a hard ceiling of 20. 
3. **The Consequence:** The 16x latency explosion is not due to slow database queries, but due to **application