+++
title = "SRE Reference Sheet"
weight = 5
+++

# SRE Quick Reference & Formulas

A high-density reference guide for core Site Reliability Engineering mathematics, metrics, and incident classification frameworks.

---

## 1. Reliability Mathematics

### Availability Formulas
The core metrics for calculating system uptime and tracking error budgets:

* **Availability by Time:**
  $$A = \frac{\text{Uptime}}{\text{Uptime} + \text{Downtime}}$$

* **Availability by Aggregate Request Success Rate:**
  $$A = \frac{\text{Successful Requests}}{\text{Total Requests}}$$

### The "Nines" Reference Table
Allowed downtime windows based on target availability percentages across common engineering tiers:

| Availability % | Downtime per Month | Downtime per Week | Downtime per Day |
| :--- | :--- | :--- | :--- |
| **99%** (Two Nines) | 7.31 hours | 1.68 hours | 14.40 minutes |
| **99.9%** (Three Nines) | 43.83 minutes | 10.08 minutes | 1.44 minutes |
| **99.99%** (Four Nines) | 4.38 minutes | 1.01 minutes | 8.64 seconds |
| **99.999%** (Five Nines) | 26.30 seconds | 6.05 seconds | 864 milliseconds |

---

## 2. The Four Golden Signals
If you can only monitor four metrics on a user-facing system, focus on these:

1. **Latency:** The time it takes to service a request (split into success vs. failure latency).
2. **Traffic:** A measure of how much demand is being placed on your system (e.g., HTTP requests per second, I/O throughput).
3. **Errors:** The rate of requests that fail, either explicitly (e.g., HTTP 500s) or implicitly (e.g., an HTTP 200 that returns wrong data).
4. **Saturation:** A measure of how "full" your service is, emphasizing the most constrained system resources (e.g., memory, CPU, thread pool exhaustion).

---

## 3. Incident Severity Framework
| Severity | Impact | Response Level | Target MTTR |
| :--- | :--- | :--- | :--- |
| **Sev 1 (Critical)** | Core user-facing functionality down for all users. Direct financial/reputational damage. | Immediate page. Incident Commander assigned. Continuous updates. | < 1 Hour |
| **Sev 2 (Major)** | Major system degraded or partial outage impacting a subset of users. No clear workaround. | Immediate page. Team triage. Updates every 30-60 mins. | < 4 Hours |
| **Sev 3 (Minor)** | Minor system degradation, localized issue, or redundant component failure. Workaround exists. | Next business day or standard ticketing queue. | < 48 Hours |

---

> 💡 **Tip:** When calculating error budgets, remember that your **SLO** is the boundary where your users start noticing unreliability. Setting an SLO tighter than user perception wastes engineering velocity unnecessarily.
