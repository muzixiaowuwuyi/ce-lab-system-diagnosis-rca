# USE Method Analysis - Resource Component Inspection

## 1. Host Compute (EC2 Instance)
* **CPU Utilization:** Avg: 45%, Max: 55% 🟢 **Status: OK**
  * *Saturation:* Load average 2.0 (on a 4-core machine), no queuing.
  * *Errors:* 0 system level errors.
* **Memory (mem_used_percent):** Avg: 70%, Max: 72% 🟢 **Status: OK**
  * *Saturation:* Swap usage is 0%.
  * *Errors:* None (No OOM killer invocations).

## 2. Database Connection Pool (Application Resource)
* **Resource Definition:** Logical connection pool size allocated within the Python application wrapper (Max capacity hardcoded to `20`).
* **Utilization:** **100% (Constant at 20/20 connections)** 🔴 **Status: EXHAUSTED**
* **Saturation:** Severe queueing detected. Incoming threads are blocked waiting for an available database connection. Average wait time in queue reached **4,700ms**.
* **Errors:** High volume of log appends with: `sqlalchemy.exc.TimeoutError: QueuePool limit of size 20 overflow 10 has been reached, connection timed out.`

---

## Conclusion
The physical hardware has plenty of headroom. The system bottleneck is entirely a **logical resource saturation issue** due to database connection pool exhaustion.