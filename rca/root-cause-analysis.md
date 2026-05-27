# Root Cause Analysis: Production API Latency Degradation

## Executive Summary
* **Incident Description:** Target response time (P95) escalated from 300ms to 5,000ms, causing massive frontend page load delays.
* **Impact:** Affected ~5,000 active sessions with a 5% HTTP 5XX error rate.
* **Duration:** 60 Minutes (14:00 - 15:00 UTC, Jan 20, 2024)
* **Root Cause:** Database connection pool exhaustion under sudden traffic spike.
* **Resolution Status:** Resolved via emergency hotfix (Pool capacity raised to 50).

## 5 Whys Deep Dive
1. **Why was the application slow?** P95 API latency spiked to 5,000ms.
2. **Why was the latency so high?** Application execution threads were blocked queuing for database connections.
3. **Why were threads blocking?** The database connection pool reached its hard limit of 20/20 utilization (100%).
4. **Why did the pool exhaust?** Traffic tripled from 500 to 1,500 requests/min due to an unannounced marketing campaign.
5. **Why was the system unprepared?** The pool size was hardcoded for static baseline traffic, lacking both runtime autoscaling capabilities and cross-departmental alignment.

## Action Items
* [DONE] **Immediate:** Increase pool size to 50 and set maximum overflow.
* [TODO] **Short-term:** Configure CloudWatch Alarm on `DBConnectionPoolUsage > 80%` (Owner: Sarah, Due: Next Friday).
* [TODO] **Long-term:** Refactor the database client wrapper to support dynamic runtime connection pool scaling.