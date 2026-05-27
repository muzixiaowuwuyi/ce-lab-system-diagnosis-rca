# Cloud Engineering Lab: System Diagnosis & RCA

## Project Overview
This repository contains a comprehensive Root Cause Analysis (RCA) and system diagnosis framework for a critical production degradation incident. The scenario simulates a classic **Database Connection Pool Exhaustion** under a sudden 3x traffic spike.

### Repository Layout
* `rca/`: Core Root Cause Analysis documents and investigation timelines.
* `analysis/`: Multi-dimensional metrics inspection utilizing both **RED** and **USE** methodologies.
* `proposals/`: Tiered remediation strategies covering immediate hotfixes, long-term dynamic auto-scaling architecture, and operational prevention rules.

---

## 🧠 Reflection & Technical Defenses (Debrief)

### 1. Why use the RED method first, then the USE method?
* **RED Method** targets **Services (External-facing)**. It directly measures user-experience degradation (Latency, Errors, Throughput). When an incident occurs, we always check RED first to determine *how badly the user is impacted*.
* **USE Method** targets **Resources (Internal-facing)**. Once the symptom is confirmed via RED, we use the USE method to pinpoint *which specific component (CPU, Memory, or Logical Pools) is broken*. 
* In short: **RED tells us *what* is broken; USE tells us *why* it broke.**

### 2. How did correlation help identify the root cause?
By superimposing the timelines of multiple separate metrics, we identified a textbook cascading failure pattern:
The exact minute the ALB `RequestCount` tripled (14:00), the custom metric `DBConnectionPoolUsage` hit its hard limit of 100% (20/20), and *simultaneously*, the `TargetResponseTime` exploded to 5,000ms. 
Since compute metrics (CPU at 45%, Memory at 70%) stayed flat and perfectly healthy, this precise cross-metric temporal correlation isolated the logical connection pool as the sole system bottleneck.

### 3. What is the difference between immediate and long-term fixes?
* **Immediate Fix:** A tactical "firefighting" response. It focuses on **mitigation and fast recovery** with minimal code changes (e.g., changing a hardcoded configuration value from 20 to 50). It accepts sub-optimal resource efficiency just to bring the service back online.
* **Long-term Fix:** A strategic "engineering" response. It focuses on **permanent remediation and system resilience** by re-architecting the component (e.g., rewriting the pool manager to dynamically auto-scale between 20 and 120 based on concurrent demand), removing the risk of hard ceilings entirely.

### 4. Why write an RCA even after the issue is resolved?
* **Systemic Learning:** To capture exact metrics, timelines, and behaviors before they disappear from short-term logs.
* **Blameless Culture:** It shifts the focus from "who messed up" to "how can we engineer our systems to be bulletproof against human mistakes."
* **Defect Elimination:** It forces the team to commit to concrete, tracked action items (Action Items) so that the exact same vulnerability never brings down the platform again.

### 5. How do you determine incident severity (P0, P1, P2)?
Severity is driven by two main pillars: **User Impact Scope** and **Core Revenue Degradation**.
* **P0 (Catastrophe):** Core system completely down for all users. Critical business operations halted (e.g., Payment Gateway entirely failing). Massive financial/legal bleeding.
* **P1 (Critical Degradation - This Incident):** System is online but severely degraded. High latency (5s) and a noticeable error rate (5%) impacting a significant chunk of active sessions (~5,000 users), causing immediate user friction.
* **P2 (Major/Minor):** Non-blocking bugs or performance hits confined to non-critical auxiliary services (e.g., Profile picture upload delay), with easy workarounds available.