---
title: "3. Data Integrity"
weight: 43
---

# Pillar 4 — Chapter 26: Data Integrity: What You Read Is What You Wrote

If a server goes down, it is an availability crisis. If a data center loses power, it is a capacity crisis. But if a distributed database silently corrupts or deletes its data, it is an existential crisis. A system can recover from an outage in minutes, but recovering from catastrophic data loss can take weeks—or prove completely impossible.

For SREs, data integrity is the absolute baseline of system reliability. This chapter focuses on how to architect systems to prevent data corruption, why traditional replication strategies fail to protect data, and how to build automated validation systems to ensure your backups are actually recoverable.

---

## The Great Misconception: Replication $\neq$ Backup

The most common architectural mistake engineers make is assuming that highly available, multi-region database replication protects data integrity. 

Replication protects against **hardware failure**, not **data corruption**.

```text
 [ Application Bug / Malicious SQL ] 
                 |
                 V (e.g., UPDATE users SET email = NULL;)
   +---------------------------+
   |   Primary DB (Region 1)   |
   +---------------------------+
                 |
                 V  (Synchronous / Asynchronous Replication)
   +---------------------------+
   |   Replica DB (Region 2)   |  <--- Corruption is faithfully 
   +---------------------------+       replicated in milliseconds.
```

If an application bug, a malicious actor, or an exhausted engineer executes an unconstrained `DROP TABLE` or a corrupted data mutation, a distributed consensus engine (like Raft) or a database replication stream will faithfully, perfectly, and instantly execute that destruction across every replica worldwide.

### Distinguishing the Two Protection Layers

|Dimension | Distributed Replication | Data Integrity Backups |
|----------|-------------------------|------------------------|
| **Primary Goal** | Minimize downtime (Low MTTR) and protect against localized hardware/network failure. | Protect against logical corruption, human error, and system-wide software bugs.|
| **Mechanisms** | Real-time consensus logs, live read-replicas, active-active cross-region mirrors. | Point-in-Time Recovery (PITR), isolated cold storage snapshots, transaction log archives.|
| **Failure Scope** | Survives a physical disk failure or an entire datacenter blackout. | Survives a catastrophic database-wide software bug or a rogue internal script.|

## The Core Sources of Data Corruption

SREs categorize data integrity threats into three main vectors:

1. **Software Bugs**: Application code that erroneously overwrites fields, improperly serializes data types, or experiences race conditions that introduce logical corruption.

2. **Human Operational Error**: Engineers running scripts against production environments instead of staging, misconfiguring schema migrations, or accidentally deleting cloud storage buckets.

3. **Hardware Deficiencies**: Silent bit rot, bad memory chips (ECC failures), or faulty disk controllers that flip bits on the physical media without throwing low-level kernel errors.

## Defensive Engineering Patterns for Data Integrity
To safeguard critical state against these vectors, systems must be built with structural guardrails.

### Hardening Mutations with Soft Deletes
Never allow application-tier users or standard internal APIs to execute hard `DELETE` operations on raw storage. Instead, implement a **Soft Delete** pattern by introducing a lifecycle state (e.g., moving records to a "Trash" state or appending a `deleted_at` timestamp). This keeps the underlying data intact for a buffer window (e.g., 30 days), allowing simple recovery from accidental deletions.

### Immutable Ledger Architectures
Wherever possible, use append-only data structures. In an event-sourced architecture, state is never modified in-place; instead, new events are continuously appended to a ledger. If a bug introduces corruption, the ledger can be replayed up to the exact millisecond before the bug occurred, completely neutralizing the corrupting events.

### Fencing and Access Isolation
Production database access must adhere to the principle of least privilege. Regular application service accounts should not have administrative privileges like `DROP`, `ALTER`, or unconstrained `DELETE`. Furthermore, automated tools or scripts executing data migrations must be wrapped in strict transaction boundaries that automatically roll back changes if anomaly thresholds are tripped.

## The Data Recovery Paradox: Validating Backups

The ultimate law of SRE data integrity states: **The value of a backup is strictly determined by its restore success rate.**

Many organizations monitor their backup pipelines by checking if the backup script exited with code `0`. This is a dangerous anti-pattern. A backup job can successfully dump a 500GB file of pure whitespace or corrupted headers, leaving you with an unrecoverable asset during a disaster.

```text
[ Production DB ] ---> [ Nightly Snapshot ] ---> [ Isolated Sandbox Env ]
                                                         |
                                                         V (Automated Restoration)
                                                 [ Temporary Live DB ]
                                                         |
                                                         V (Data Verification Tests)
                                                 [ Run Checksums / Queries ]
                                                         |
                                                         V
                                                 [ Publish Integrity Report ]
```

1. **Automated Restoration**: Every night, an isolated, automated sandbox environment provisions temporary infrastructure and pulls down the latest production backup.

2. **Schema and Data Verification**: The pipeline runs automated validation checks:
    * It verifies the schema structure matches the active production definition.
    * It runs a battery of known transactional queries to check record volume consistency (e.g., ensuring total user count hasn't plummeted by 90% compared to the day before).
    * It performs cryptographic checksum evaluations against critical tables.

3. **Alerting on Failure**: If the restoration fails, or if the data consistency checks fail, an immediate high-priority alert is routed to the on-call SRE.

## Summary Takeaway

Data integrity is the ultimate line of defense for digital infrastructure. While replication preserves uptime, only a rigorous combination of immutable logs, soft deletes, and out-of-band backup validation preserves data correctness. SREs treat backups not as an archival chore, but as an active software system that must be continuously restored, tested, and proven functional every single day.
