---
title: "2. SLO & SLI Frameworks"
weight: 12
---

# Pillar 1 — Chapter 2: Defining Reliability (The SLI/SLO Framework)
To manage an infrastructure stack effectively, you must be able to measure its behavior quantitatively. However, in a modern cloud-native ecosystem, traditional monitoring metrics—like checking if a single specific virtual machine is pinging—are completely obsolete.

This chapter details how to define service health from the user's perspective using SLIs and SLOs, and explains how this monitoring framework matches modern, automated cluster architecture.

---

## Infrastructure Context: From Pets to Cattle
The way we measure reliability depends entirely on how we treat our infrastructure. Traditional operations treat servers like **Pets**: each machine is given a unique name, configured manually, and carefully nursed back to health when it degrades. If a "pet" server goes down, an engineer is alerted immediately to fix it.

Modern distributed platforms—pioneered by Google's internal orchestrator *Borg* and mirrored globally by *Kubernetes*—shift this paradigm to treating servers like **Cattle**. 

In a cattle model, individual infrastructure instances are completely anonymous and interchangeable. If a node suffers a hardware failure, the control plane automatically provisions a replacement container on a completely different machine, terminating the broken instance without human intervention. 

### The Monitoring Shift
Because workloads move fluidly across a cluster, tracking the uptime of a single operating system instance is a waste of engineering time. SRE shifts monitoring focus away from **component uptime** and moves it entirely toward **user-centric service health**. The user doesn't care *which* physical server handles their API request; they only care that the request returns successfully and quickly.

---

## The Measurement Hierarchy: SLI vs. SLO vs. SLA
SRE establishes three distinct layers to define, track, and guarantee service reliability.

```
[ SLI ]  -->  What raw percentage of requests succeeded right now?
|
[ SLO ]  -->  What target percentage must succeed over a 30-day window?
|
[ SLA ]  -->  What are the business/legal consequences if we fail the SLO?
```

### 1. Service Level Indicators (SLIs)
An SLI is a quantifiable metric that measures the performance of a service at a specific point in time. It represents the raw performance data driving your operational awareness.

*   **The Math:** An SLI is almost always expressed as a ratio of good events to total valid events, multiplied to create a percentage:

$$\text{SLI} = \left( \frac{\text{Successful Events}}{\text{Total Valid Events}} \right) \times 100$$

*   **Example (Latency):** The number of HTTP GET requests to `/profile` that returned in less than 200ms, divided by the total number of valid HTTP GET requests to `/profile`.

### 2. Service Level Objectives (SLOs)
An SLO is a target reliability percentage set by engineering and product teams. It defines the acceptable performance boundary over a specific time window (such as a rolling 30 days).

*   **Example:** "The 30-day rolling availability SLI for the authentication service must be greater than or equal to 99.9%."
*   **Design Principle:** SLOs should closely track actual user satisfaction. If your system metrics show 99.99% availability but your customer support queue is flooded with outage complaints, your SLIs are measuring the wrong indicators.

### 3. Service Level Agreements (SLA)
An SLA is a legal or financial contract between a service provider and its customers. It explicitly outlines the financial penalties, credits, or contract clauses triggered if the service fails to meet its targets. 

SREs do not write SLAs; that is a legal and business function. SREs focus entirely on protecting the underlying SLOs. To provide a safe operating buffer, an internal engineering SLO should always be stricter than the public-facing SLA.

---

## Practical Blueprint: Choosing Indicators by Workload
Not all services are measured the same way. SRE targets distinct indicators based on the architectural shape of the system component:

| Service Type | Primary SLI Focus | Example SLO Target (30-Day Window) |
| :--- | :--- | :--- |
| **User-Facing Web APIs** | Availability & Latency | $\ge$ 99.5% of HTTP responses must return status code 200 within 300ms. |
| **Asynchronous Data Pipelines** | Pipeline Throughput & Freshness | $\ge$ 99.0% of processed records must have a data age of less than 15 minutes. |
| **Storage & Databases** | Durability & Correctness | 100% of validated data blocks must read back without corruption loops. |

---

## Summary Takeaway
By moving from a "Pets" infrastructure model to an automated "Cattle" framework, your monitoring must adapt. SRE achieves this by ignoring individual machine health and building automated SLI ratios that reflect true end-user experiences. These metrics feed directly into your SLOs, giving you the raw data required to calculate your remaining Error Budget.