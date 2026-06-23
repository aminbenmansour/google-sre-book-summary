---
title: "1. On-Call Management"
weight: 21
---

# Pillar 2 — Chapter 11: Managing On-Call & Operational Health
On-call duty is often treated as an unpleasant, chaotic tax of running software. In an SRE model, on-call is treated as an engineered engineering discipline. The goal is not simply to ensure someone answers the pager, but to design a system where human operational load remains mathematically sustainable.

This chapter outlines the frameworks required to balance service reliability with team health, establishing strict limits on alert volumes and cognitive fatigue.

---

## The Core Blueprint: What Makes On-Call Sustainable?
SRE teams do not accept infinite operational scaling. For an on-call rotation to remain healthy, it must adhere to tight numerical guardrails. Google's empirical data shows that a rotation degrades rapidly if it violates the following structural boundaries:

* **The 50% Safety Valve:** As established in Pillar 1, operational activities (handling alerts, pages, and tickets) must never consume more than 50% of an engineer's time. The remaining 50% must be preserved for deep project work.
* **The 2-Event Rule:** On average, an engineer on a 12-hour shift should handle **no more than two alerting incidents**. This ensures they have sufficient buffer time (roughly six hours per incident) to diagnose the root cause, mitigate the immediate failure, write a detailed timeline, and hand off context cleanly. 
* **The Minimum Team Size:** To prevent burnout and ensure proper shift distribution, a single localized on-call rotation requires a absolute minimum of **five engineers**. For a true 24/7 follow-the-sun model (splitting day/night shifts across geographic regions), a minimum of **eight engineers** divided across two locations is required to prevent continuous weekend and holiday coverage fatigue.

---

## The Signal-to-Noise Filter: Symptom-Based Alerting
The quickest way to burn out an on-call team is to page them for things that do not require immediate human intervention. SRE strictly segregates operational data into three actionable channels:

### 1. Alerts (The Pager)
An engineer is woken up or interrupted **only** if a user-facing system is actively breaking or about to break critically within minutes, AND a human must take manual action to fix it. If no immediate action can be taken, it is not an alert.

### 2. Tickets
Non-urgent production anomalies that require human evaluation but can wait until normal business hours are routed to a ticket queue. Examples include a single drive failure in a RAID array or a slow, predictable disk growth trend that will breach thresholds next week.

### 3. Logs / Metrics
Data collected purely for post-incident diagnostics, performance tuning, or long-term capacity planning. No one should ever look at this data proactively unless troubleshooting an active issue or building an architecture review.

> **The Golden Rule of Paging:** If an alert fires and the engineer can simply click "Acknowledge" and go back to sleep without taking an immediate remediation step, the alert is broken. It must be demoted to a ticket or deleted entirely.

---

## Evaluating On-Call Health
How do you know if your engineering rotation is tracking toward a burnout disaster? Use this quick-reference categorization matrix to evaluate your operational state:

| Operational Metric | Healthy Rotation | Toxic / Burnout Rotation |
| :--- | :--- | :--- |
| **Pager Volume** | 0 to 2 actionable pages per 12-hour shift. | 10 or more pages per shift; frequent "alert storms." |
| **Alert Type** | **Symptom-based** (e.g., HTTP 5xx error rate greater than 5%). | **Cause-based** (e.g., High CPU on host X, which occurs normally during cron jobs). |
| **Runbook Coverage** | Every page links to a specific, updated remediation runbook. | Pages are cryptic; engineers rely entirely on tribal knowledge. |
| **Post-Incident Action** | Action items are prioritized in the next sprint to fix the root cause. | Incidents are patched temporarily; the same alerts fire every week. |

---

## The On-Call Sustainability Calculator
To determine if your current team size and incident rate can sustain your infrastructure without inducing burnout, use the interactive calculator below. It applies Google's 50% operational cap and 2-event rule limits against your actual operational metrics.

{{< sre-sustainability-calculator >}}

---

## Managing the Feedback Loop: Operational Cooldown
When a production service falls into disrepair, it begins generating an overwhelming volume of alerts. If left unchecked, the SRE team becomes entirely reactive, spending all their time firefighting.

SRE models handle this via a formal **Operational Cooldown** mechanism:

If a service's alert volume blows past sustainable thresholds, the SRE team has the explicit authority to **hand the pager back to the product developers**. The product development team takes over primary night-and-day on-call duties for that service. They are forced to halt new feature code entirely and focus exclusively on stabilizing the system. 

Once the service's error budget and alert metrics return to a stable, healthy baseline for a consecutive 14-day window, the SRE team absorbs the pager back into the core rotation. This structural gatekeeping ensures that product groups are held financially and operationally accountable for the architectural quality of the code they write.

---

## Summary Takeaway
Sustainable on-call is an engineering constraint, not an emotional goal. By enforcing symptom-based alerting, keeping a hard wall at 2 alerts per shift, and actively using the operational cooldown to push back against unstable systems, organizations ensure that their platforms can scale massively while keeping their best engineers healthy, focused, and retained.