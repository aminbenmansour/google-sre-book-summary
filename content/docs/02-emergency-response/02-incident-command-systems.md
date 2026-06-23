---
title: "2. Incident Command"
weight: 22
---

# Pillar 2 — Chapter 14: Managing Incidents & Incident Command
When a high-severity outage strikes, technical teams often fall into a predictable trap: a single brilliant engineer tries to simultaneously fix the underlying bug, update anxious executives on Slack, field calls from customer support, and coordinate five other developers over a messy Zoom call. 

This multi-tasking approach is an operational failure mode. It drastically increases your Mean Time to Resolution (MTTR) and destroys team focus. 

Google solves this by pulling a framework directly from real-world emergency responders: the **Incident Command System (ICS)**. This chapter details how to split emergency response into strict, isolated roles to preserve technical focus and resolve critical outages systematically.

---

## The Core Concept: The Separation of Concerns
The foundational rule of SRE incident response is absolute: **The person debugging the system should never be the person managing the communication or logistics.**

When an incident is declared, corporate hierarchy is instantly discarded. A junior engineer acting as the designated commander can tell a Vice President to leave a debugging channel, and that VP must comply. This authority is necessary to defend the cognitive bandwidth of the engineers executing tactical remediation.

While municipal emergency responders deploy comprehensive structures like the one shown above (with branches dedicated to complex logistics, fleet movements, and finance), SRE strips this model down into a highly efficient, agile three-role configuration tailored for software environments.

---

## The Three Crucial Roles
For almost all software infrastructure incidents, the command structure is boiled down to three distinct responsibilities:

### 1. The Incident Commander (IC)
The IC owns the entire event but does not fix the problem. They hold ultimate logistical authority.
* **Responsibilities:** Defines the scope of the problem, assigns the other roles, keeps the response organized, and resolves high-level strategic deadlocks. 
* **Key Trait:** The IC focuses on the big picture. If the IC logs into a server to run `top` or check database metrics, they have failed. They must hand off command to someone else before diving into technical execution.

### 2. The Operations Lead (OL)
The OL owns the active technical mitigation work.
* **Responsibilities:** Leads the debugging efforts, coordinates code rollbacks, spins up isolated database clusters, and directs the technical traffic. 
* **Key Trait:** The OL speaks directly to the IC and directs the tactical engineering team ("Strike Teams"). They are completely insulated from outside noise.

### 3. The Communications Lead (CL)
The CL owns the interface between the crisis room and the rest of the world.
* **Responsibilities:** Writes public status-page updates, drafts internal emails to executives, and handles incoming requests from the customer support team.
* **Key Trait:** The CL ensures that the IC and OL are never interrupted by people asking, *"When will it be back up?"*

---

## Comparative Operational Breakdown
To understand how these roles interact seamlessly without stepping on each other's toes during a live incident, review their boundaries:

| Attribute | Incident Commander (IC) | Operations Lead (OL) | Communications Lead (CL) |
| :--- | :--- | :--- | :--- |
| **Primary Goal** | Keep the response structure organized and moving forward. | Isolate, mitigate, and resolve the technical failure. | Keep internal and external stakeholders informed. |
| **Focus Area** | Logistics, role allocation, and tracking high-level milestones. | Production systems, configuration state, and log analysis. | Status pages, public notices, and internal chat updates. |
| **Banned Activities** | Writing code, running commands, or diving into log traces. | Answering executive queries or writing public status updates. | Making manual infrastructure modifications or architectural choices. |

---

## Live Incident Flow: A Procedural Runbook
To see how these roles orchestrate a response in real-time, step through the strict procedural sequence below:

<Sequence>
{/* Reason: Procedural step-by-step sequence where strict ordering prevents communication breakdown and chaotic troubleshooting during a live outage. */}
  <Step title="Declare the Incident" subtitle="Minute 0">
    Any engineer identifies a major SLO violation or severe customer-facing anomaly and formally declares an incident (e.g., via an internal Slack bot command `/incident declare`). By default, the reporter becomes the initial **Incident Commander (IC)**.
  </Step>
  <Step title="Establish the Command Outpost" subtitle="Minute 2">
    The IC spins up a dedicated incident communication hub—a secure bridge line, a distinct Slack channel, and a live shared scratchpad document (Incident State Doc). This separates incident noise from everyday team chatter.
  </Step>
  <Step title="Delegate the Operational & Comms Leads" subtitle="Minute 5">
    The IC appoints an **Operations Lead (OL)** to drive technical troubleshooting and a **Communications Lead (CL)** to manage stakeholders. Roles are pinned directly to the top of the incident channel for crystal-clear visibility.
  </Step>
  <Step title="Execute Periodic Status Syncs" subtitle="Every 15-20 Minutes">
    The IC runs structured, 90-second syncs at fixed intervals. The OL provides a brief technical update; the IC evaluates strategy; the CL takes the notes to draft external updates. The cycle repeats until the system achieves stable mitigation.
  </Step>
  <Step title="Hand Off or Hand Down" subtitle="Post-Mitigation">
    Once the system is stabilized and customer impact drops to zero, the IC formally stands down the incident. The team closes out the shared scratchpad and preserves it directly as the primary data source for the upcoming postmortem phase.
  </Step>
</Sequence>

---

## When to Spin Up Incident Command
An incident command structure introduces operational overhead. If you use it for every minor bug, your team will experience process fatigue. SREs trigger this formal framework only when specific operational thresholds are crossed:

* **The SLO Breach:** Your error budget is burning at a rate that threatens to exhaust the month's allowance within hours.
* **Ambiguity:** A critical alert is firing, but after 10 minutes of investigation, no one knows *why* it's happening or *who* should fix it.
* **High Stakeholder Inundation:** The moment you notice engineers spend more time answering questions in chat loops than running diagnostics, you must spin up an IC and CL to close the floodgates.

---

## Summary Takeaway
Incident management is an exercise in reducing cognitive noise. When your infrastructure is on fire, don't rely on chaotic heroism. By explicitly separating logistics (IC), engineering execution (OL), and messaging (CL), you let your technical experts do exactly what they excel at—fixing the system—while the rest of the business remains informed and at peace.
