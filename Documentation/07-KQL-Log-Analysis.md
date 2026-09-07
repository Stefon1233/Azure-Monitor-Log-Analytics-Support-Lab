# 07 — KQL Log Analysis

## Azure Monitor + Log Analytics Support Lab

---

## Overview

Kusto Query Language, commonly called KQL, is the primary query language used throughout this Azure Monitor + Log Analytics Support Lab.

The lab uses KQL to investigate operational telemetry collected from:

```text
AZMON-WIN01
```

and stored in:

```text
LAW-AZMON-SUPPORT-LAB
```

KQL is used throughout the project to:

```text
Validate monitoring
Investigate incidents
Identify abnormal behavior
Correlate telemetry
Measure performance changes
Analyze recovery
Build alert conditions
Visualize timelines
```

The goal of this documentation section is to explain how KQL was used as an operational troubleshooting tool rather than only as a reporting language.

---

# Environment

## Monitored Virtual Machine

```text
AZMON-WIN01
```

Operating system:

```text
Windows Server 2022 Datacenter: Azure Edition
```

---

## Log Analytics Workspace

```text
LAW-AZMON-SUPPORT-LAB
```

---

## Data Collection Rule

```text
DCR-AZMON-WINDOWS
```

---

## Monitoring Agent

```text
Azure Monitor Agent
```

The monitoring path is:

```text
AZMON-WIN01
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Log Analytics Tables
      ↓
KQL
```

---

# KQL Role in the Lab

KQL was used in several stages of the incident-response process.

```text
Known-Good Baseline
      ↓
Telemetry Query
      ↓
Incident Generation
      ↓
KQL Detection
      ↓
Trend Analysis
      ↓
Root-Cause Investigation
      ↓
Remediation
      ↓
Recovery Query
      ↓
Incident Closure
```

This made KQL a core part of the support workflow.

---

# Main Tables Used

The project primarily worked with telemetry associated with:

```text
Perf
Heartbeat
Windows event data
Azure Activity Log data
```

Each source provides a different view of the environment.

---

# Perf Table

The `Perf` table was used extensively for guest operating-system performance monitoring.

Examples included:

```text
CPU
Memory
Logical disk
Network
```

A typical query pattern is:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
```

This narrows the query to:

```text
Recent telemetry
+
AZMON-WIN01
```

---

# Performance Counter Structure

Important `Perf` columns used in the lab included:

```text
TimeGenerated
Computer
ObjectName
CounterName
InstanceName
CounterValue
```

These fields allowed the lab to identify:

```text
What was measured
Which server produced the measurement
When it was collected
Which performance counter was involved
What value was recorded
```

---

# Counter Inventory

One of the most important KQL troubleshooting techniques used in the project was performance-counter discovery.

Example:

```kusto
Perf
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| summarize SampleCount=count(),
            LastSample=max(TimeGenerated)
    by ObjectName, CounterName, InstanceName
| order by ObjectName asc, CounterName asc
```

This query helped identify the exact counters being collected by the Data Collection Rule.

---

# Why Counter Inventory Matters

A query may return no results because:

```text
Wrong ObjectName
Wrong CounterName
Wrong InstanceName
Wrong time range
Telemetry not collected
Agent interruption
Data Collection Rule configuration
```

The inventory query helps determine whether the telemetry exists before changing monitoring configuration.

---

# Memory Counter Discovery

During INC-006, the original memory query searched for:

```text
Memory\Available MBytes
```

but the query returned no results.

Counter inventory showed the actual collected counter was:

```text
Memory\Available Bytes
```

The monitoring configuration was healthy.

The problem was the query filter.

This demonstrated an important troubleshooting principle:

```text
Inspect the actual telemetry before changing the DCR.
```

---

# Available Memory Query

The corrected query used:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| top 1 by TimeGenerated desc
| project TimeGenerated,
          Computer,
          AvailableMB=round(CounterValue / 1024.0 / 1024.0,0),
          AvailableGB=round(CounterValue / 1024.0 / 1024.0 / 1024.0,2)
```

This converted the raw byte value into:

```text
Megabytes
Gigabytes
```

for easier operational interpretation.

---

# Byte Conversion

The collected counter returned:

```text
Bytes
```

KQL conversion to MB:

```kusto
CounterValue / 1024.0 / 1024.0
```

KQL conversion to GB:

```kusto
CounterValue / 1024.0 / 1024.0 / 1024.0
```

---

# CPU Performance Analysis

CPU telemetry was investigated using the `Perf` table.

Example:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where CounterName contains "% Processor Time"
| project TimeGenerated,
          Computer,
          ObjectName,
          CounterName,
          CounterValue
| order by TimeGenerated desc
```

This provided guest operating-system CPU telemetry independently of Azure platform Percentage CPU metrics.

---

# Platform Metric vs Guest Telemetry

The lab used both:

```text
Azure platform metrics
```

and:

```text
guest performance telemetry
```

This distinction is important.

Azure platform metrics can show resource-level behavior, while guest telemetry provides visibility into the operating system.

Using both gives stronger evidence during troubleshooting.

---

# Disk Space Analysis

INC-005 used KQL to investigate logical disk capacity.

The collected counter was:

```text
LogicalDisk\Free Megabytes
```

A typical query pattern was:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
```

This allowed storage changes to be independently verified in Log Analytics.

---

# Disk Capacity Incident

INC-005 generated a controlled storage change of approximately:

```text
10 GB
```

The KQL evidence showed:

```text
Baseline
      ↓
Approximately 10 GB reduction
      ↓
Controlled incident
      ↓
File removed
      ↓
Storage recovery
```

The query data confirmed that guest-level disk changes were successfully collected and ingested.

---

# Heartbeat Table

The `Heartbeat` table was used to validate Azure Monitor Agent communication.

Example:

```kusto
Heartbeat
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| project TimeGenerated,
          Computer
| order by TimeGenerated desc
```

This answers an important operational question:

```text
Is the monitored server still reporting?
```

---

# Missing Heartbeat Investigation

INC-002 demonstrated that:

```text
Missing telemetry
```

does not necessarily mean:

```text
VM unavailable
```

The investigation separated:

```text
Agent / telemetry interruption
```

from:

```text
Infrastructure failure
```

using multiple sources.

---

# Latest Heartbeat

A useful operational query pattern is:

```kusto
Heartbeat
| where Computer =~ "AZMON-WIN01"
| summarize LastHeartbeat=max(TimeGenerated)
    by Computer
```

This quickly identifies the most recent monitoring heartbeat.

---

# Heartbeat Gap Analysis

A missing heartbeat can be detected by comparing the current time with:

```text
LastHeartbeat
```

A production environment could use this logic to identify servers that have stopped reporting.

---

# VM Availability Investigation

INC-003 used Log Analytics together with Azure resource state and Activity Log evidence.

The goal was to distinguish:

```text
Monitoring failure
```

from:

```text
Actual VM availability change
```

During controlled VM deallocation, heartbeat stopped because the VM itself was unavailable.

---

# Windows Event Analysis

INC-004 generated a controlled Windows Application event.

The event included:

```text
Event ID:
200

Source:
AZMON-AppLab

Log:
Application

Type:
Error
```

KQL was used to locate and validate the event after it reached Log Analytics.

---

# Windows Event Query Pattern

The exact available columns depend on the collected event data, but the investigation workflow included filtering by:

```text
Computer
Time range
Event ID
Event source
Event severity
```

This allowed the generated Windows event to be correlated with the controlled incident.

---

# Time Filtering

Nearly every investigation included a time filter.

Example:

```kusto
| where TimeGenerated > ago(30m)
```

Common time windows used in the lab included:

```text
ago(5m)
ago(15m)
ago(30m)
ago(45m)
ago(60m)
ago(2h)
```

---

# Why Time Windows Matter

A query that searches too little history may miss:

```text
Baseline data
Incident onset
Previous samples
Recovery samples
```

A query that searches too much history may return:

```text
Unrelated events
Older incidents
Excess noise
```

Time windows were selected according to the investigation phase.

---

# Latest Record Analysis

The project frequently needed the newest telemetry record.

Two common approaches were used.

## top

```kusto
| top 1 by TimeGenerated desc
```

This is useful when only the latest record is required.

---

## arg_max

```kusto
| summarize arg_max(TimeGenerated, CounterValue) by Computer
```

This is useful when grouping by a resource such as:

```text
Computer
```

and returning the newest value.

---

# Scheduled Query Alert Logic

INC-007 used KQL as automated detection logic.

The alert query was:

```kusto
Perf
| where TimeGenerated > ago(5m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| summarize arg_max(TimeGenerated, CounterValue) by Computer
| extend AvailableMB=CounterValue / 1024.0 / 1024.0
| where AvailableMB < 2048
| project TimeGenerated,
          Computer,
          AvailableMB=round(AvailableMB,0)
```

---

# Alert Query Behavior

The query intentionally behaves differently depending on system state.

Healthy state:

```text
Available memory > 2048 MB
      ↓
No rows returned
```

Low-memory state:

```text
Available memory < 2048 MB
      ↓
One matching row returned
```

---

# Alert Rule Measurement

The Azure Monitor scheduled query rule used:

```text
Measurement:
Table rows / Count
```

Condition:

```text
Greater than 0
```

Therefore:

```text
0 rows
=
Healthy
```

and:

```text
1 or more rows
=
Alert
```

---

# Healthy Alert Query Validation

Before triggering INC-007, available memory was approximately:

```text
2911 MB
```

The detection query returned:

```text
No results
```

This confirmed:

```text
Alert condition:
FALSE
```

---

# Triggered Alert Query

During the controlled memory-pressure workload, the query returned:

```text
AvailableMB:
1691
```

Because:

```text
1691 < 2048
```

the scheduled query rule detected the condition.

---

# Alert Lifecycle

```text
Healthy
      ↓
Query returns 0 rows
      ↓
Controlled low-memory workload
      ↓
Query returns 1 row
      ↓
Azure Monitor alert fires
      ↓
Remediation
      ↓
Healthy telemetry returns
      ↓
Query returns 0 rows
      ↓
Alert resolves
```

---

# Health State Classification

KQL can classify performance data into human-readable states.

Example:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| extend AvailableMB=CounterValue / 1024.0 / 1024.0
| extend State=iff(AvailableMB < 2048, "LOW MEMORY", "HEALTHY")
| project TimeGenerated,
          Computer,
          AvailableMB=round(AvailableMB,0),
          State
| order by TimeGenerated desc
```

---

# State Classification Result

INC-007 produced a clear sequence:

```text
HEALTHY
HEALTHY
HEALTHY
LOW MEMORY
LOW MEMORY
LOW MEMORY
LOW MEMORY
LOW MEMORY
LOW MEMORY
HEALTHY
```

This made the incident transition easy to visualize.

---

# summarize

The `summarize` operator was used to aggregate performance data.

Examples:

```kusto
min(CounterValue)
max(CounterValue)
avg(CounterValue)
count()
```

These functions help answer questions such as:

```text
What was the lowest value?
What was the highest value?
What was the average?
How many samples were collected?
```

---

# Minimum and Maximum Memory

Example:

```kusto
Perf
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| summarize MinimumBytes=min(CounterValue),
            MaximumBytes=max(CounterValue)
    by Computer
```

This quantifies the range of memory availability during an incident.

---

# Calculated Fields with extend

The `extend` operator was used to create calculated fields.

Example:

```kusto
| extend AvailableMB=CounterValue / 1024.0 / 1024.0
```

Another example:

```kusto
| extend State=iff(AvailableMB < 2048, "LOW MEMORY", "HEALTHY")
```

This allows raw telemetry to be transformed into operationally useful information.

---

# project

The `project` operator was used to limit query output to the fields needed for the investigation.

Example:

```kusto
| project TimeGenerated,
          Computer,
          AvailableMB
```

This helps produce clean screenshots and reduces unnecessary output.

---

# order by

The project commonly used:

```kusto
| order by TimeGenerated desc
```

to display newest records first.

For timelines, it used:

```kusto
| order by TimeGenerated asc
```

to display events chronologically.

---

# render timechart

Time-series visualization was used for:

```text
CPU
Memory
Disk capacity
Incident timelines
Recovery validation
```

Example:

```kusto
Perf
| where TimeGenerated > ago(60m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| summarize AvailableMB=avg(CounterValue) / 1024.0 / 1024.0
    by bin(TimeGenerated,1m)
| order by TimeGenerated asc
| render timechart
```

---

# Why Timecharts Are Valuable

Tables show exact values.

Charts make it easier to see:

```text
Baseline
Sudden change
Sustained incident
Recovery
Repeated patterns
```

INC-005 and INC-006 both used time-series evidence to prove the entire incident lifecycle.

---

# bin()

The `bin()` function groups timestamps into fixed intervals.

Example:

```kusto
bin(TimeGenerated,1m)
```

This is useful when many telemetry samples need to be represented as a readable timeline.

---

# ingestion_time()

INC-006 used:

```kusto
ingestion_time()
```

to compare:

```text
TimeGenerated
```

with:

```text
Log Analytics ingestion time
```

Example:

```kusto
Perf
| where TimeGenerated > ago(45m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| project TimeGenerated,
          IngestedAt=ingestion_time(),
          AvailableMB=round(CounterValue / 1024.0 / 1024.0,0)
| order by TimeGenerated desc
```

---

# Collection vs Ingestion

These timestamps represent different stages.

```text
TimeGenerated
=
When the telemetry sample was produced
```

```text
ingestion_time()
=
When Log Analytics ingested the record
```

This distinction is important when troubleshooting delayed monitoring updates.

---

# Telemetry Delay

After an incident is remediated, Log Analytics may temporarily show the previous state.

This does not automatically indicate monitoring failure.

The telemetry path is:

```text
Windows state changes
      ↓
Performance counter updates
      ↓
Azure Monitor Agent collects
      ↓
Telemetry transmitted
      ↓
Workspace ingests
      ↓
KQL sees new record
```

---

# Post-Remediation Validation

A strong investigation does not stop after the remediation command.

KQL was used to confirm that the monitoring platform observed the recovery.

Examples included:

```text
Disk free space restored
Memory availability restored
Heartbeat returned
VM telemetry resumed
Alert condition cleared
```

---

# Baseline vs Incident vs Recovery

A useful analysis model is:

```text
Baseline
      ↓
Incident
      ↓
Recovery
```

Each phase should have measurable telemetry.

---

# Example — Memory

Baseline:

```text
~2933 MB available
```

Pressure:

```text
~1809–1827 MB available
```

Recovery:

```text
~2915 MB available
```

---

# Example — Disk

Baseline:

```text
~112 GB free
```

Incident:

```text
~102 GB free
```

Recovery:

```text
~112 GB free
```

---

# Example — Scheduled Query Alert

Healthy:

```text
2911 MB
0 alert rows
```

Incident:

```text
1691 MB
1 alert row
```

Recovery:

```text
2764 MB
0 alert rows
```

---

# Query Validation Strategy

Before trusting a KQL query, the lab validated:

```text
Correct table
Correct computer
Correct time range
Correct object
Correct counter
Correct units
Expected row count
Expected state
```

---

# Common Query Troubleshooting Workflow

```text
Query returns no results
      ↓
Increase time range
      ↓
Confirm computer name
      ↓
Inspect available tables
      ↓
Inventory ObjectName / CounterName
      ↓
Check latest TimeGenerated
      ↓
Validate Heartbeat
      ↓
Validate AMA
      ↓
Validate DCR
```

---

# Query Design for Support Engineers

Operational queries should ideally be:

```text
Simple
Readable
Reusable
Focused
Easy to validate
Easy to screenshot
Easy to modify
```

The project therefore stores reusable queries separately from incident documentation.

---

# KQL Query Repository

Reusable queries are stored under:

```text
KQL/
```

Current files include:

```text
Activity-Log-Queries.kql
Disk-Space-Queries.kql
Heartbeat-Queries.kql
Incident-Investigation-Queries.kql
Log-Query-Alert-Queries.kql
Memory-Pressure-Queries.kql
Missing-Heartbeat-Queries.kql
Performance-Queries.kql
VM-Availability-Queries.kql
Windows-Event-Incident-Queries.kql
Windows-Event-Queries.kql
```

---

# Activity Log Queries

File:

```text
KQL/Activity-Log-Queries.kql
```

Purpose:

```text
Azure control-plane operation analysis
Administrative activity
Resource operation status
Change investigation
```

---

# Disk Space Queries

File:

```text
KQL/Disk-Space-Queries.kql
```

Purpose:

```text
Logical disk capacity
Free megabytes
Storage reduction
Recovery validation
```

---

# Heartbeat Queries

File:

```text
KQL/Heartbeat-Queries.kql
```

Purpose:

```text
Agent health
Recent heartbeat
Telemetry continuity
```

---

# Incident Investigation Queries

File:

```text
KQL/Incident-Investigation-Queries.kql
```

Purpose:

```text
Reusable cross-incident support investigation
```

---

# Log Query Alert Queries

File:

```text
KQL/Log-Query-Alert-Queries.kql
```

Purpose:

```text
Low-memory detection
Health classification
Scheduled query alerts
Recovery validation
```

---

# Memory Pressure Queries

File:

```text
KQL/Memory-Pressure-Queries.kql
```

Purpose:

```text
Available memory
Memory pressure
Timeline analysis
Recovery validation
```

---

# Missing Heartbeat Queries

File:

```text
KQL/Missing-Heartbeat-Queries.kql
```

Purpose:

```text
Monitoring interruption investigation
```

---

# Performance Queries

File:

```text
KQL/Performance-Queries.kql
```

Purpose:

```text
General VM performance analysis
```

---

# VM Availability Queries

File:

```text
KQL/VM-Availability-Queries.kql
```

Purpose:

```text
Availability investigation
Heartbeat correlation
Recovery verification
```

---

# Windows Event Incident Queries

File:

```text
KQL/Windows-Event-Incident-Queries.kql
```

Purpose:

```text
Controlled Windows event investigation
```

---

# Windows Event Queries

File:

```text
KQL/Windows-Event-Queries.kql
```

Purpose:

```text
Reusable Windows event analysis
```

---

# KQL and Root-Cause Analysis

KQL does not automatically determine root cause.

It provides evidence.

A support engineer must correlate:

```text
Telemetry
Resource state
Operating-system state
Changes
Alerts
Process behavior
Timing
```

The query results support the root-cause conclusion.

---

# Example Root-Cause Correlation

INC-006:

```text
PowerShell worker consumes ~1105 MB
      ↓
Windows available memory falls
      ↓
Memory\Available Bytes falls
      ↓
Log Analytics reports ~1.8 GB
      ↓
Worker stopped
      ↓
Windows memory recovers
      ↓
Log Analytics reports ~2.9 GB
```

This chain provides stronger RCA evidence than a single query result.

---

# KQL and Alerting

INC-007 demonstrated that a troubleshooting query can become automated monitoring logic.

```text
Manual KQL
      ↓
Validated detection condition
      ↓
Scheduled query alert
      ↓
Continuous evaluation
      ↓
Automated incident detection
```

This is one of the strongest operational capabilities demonstrated in the lab.

---

# KQL Best Practices Demonstrated

The project applied several practical habits:

```text
Always filter time
Filter to the specific computer
Use the exact collected counter name
Convert raw units
Project only useful fields
Order records intentionally
Validate healthy state first
Validate incident state
Validate recovery state
Store reusable queries
```

---

# Production Considerations

In a production environment, KQL queries may need additional controls for:

```text
Multiple servers
Application roles
Resource groups
Dynamic thresholds
Missing data
Duplicate records
Maintenance windows
Expected workloads
Different VM sizes
Regional differences
Service ownership
```

The lab uses a controlled single-server environment to clearly demonstrate the troubleshooting concepts.

---

# Skills Demonstrated

This section demonstrates:

- Microsoft Azure
- Azure Monitor
- Log Analytics
- Kusto Query Language
- Azure Monitor Agent
- Data Collection Rules
- Performance monitoring
- Heartbeat analysis
- Memory monitoring
- Disk capacity monitoring
- CPU analysis
- Windows event investigation
- Azure Activity Log investigation
- Scheduled query alerts
- KQL filtering
- KQL aggregation
- KQL calculations
- KQL visualization
- Time-series analysis
- Telemetry validation
- Ingestion analysis
- Root-cause investigation
- Incident response
- Recovery verification

---

# Key Takeaways

```text
KQL is central to Azure operational troubleshooting.

A no-results query does not automatically mean monitoring is broken.

Counter inventory should be performed before changing collection configuration.

Guest telemetry and Azure platform telemetry should be correlated.

TimeGenerated and ingestion time represent different stages.

Incident queries should validate baseline, failure, and recovery.

KQL can be converted into scheduled automated alert logic.

Recovery should be proven through new telemetry before an incident is closed.
```

---

# Final Validation

```text
Perf analysis:
VERIFIED

Heartbeat analysis:
VERIFIED

Counter inventory:
VERIFIED

CPU investigation:
VERIFIED

Disk investigation:
VERIFIED

Memory investigation:
VERIFIED

Windows event analysis:
VERIFIED

Time-series visualization:
VERIFIED

Telemetry ingestion analysis:
VERIFIED

Scheduled query alert logic:
VERIFIED

Post-remediation validation:
VERIFIED

Reusable KQL library:
COMPLETE
```

---

# Screenshot Evidence

General KQL workspace validation evidence is stored in:

```text
Screenshots/07-KQL/
```

Captured evidence:

```text
01-KQL-Workspace-Validation.png
```

This screenshot demonstrates that the Log Analytics query workspace was available and operational for KQL investigation.

Incident-specific KQL evidence is stored with the corresponding incident screenshots, including:

```text
07-Missing-Heartbeat
08-VM-Availability
09-Windows-Events
10-Disk-Space
11-Memory-Pressure
12-Log-Query-Alert
```

---

# Conclusion

KQL provided the primary analytical layer for the Azure Monitor + Log Analytics Support Lab.

The project demonstrates how an Azure support engineer can move from raw monitoring telemetry to a structured troubleshooting workflow:

```text
Collect
      ↓
Query
      ↓
Analyze
      ↓
Correlate
      ↓
Detect
      ↓
Remediate
      ↓
Validate
```

The reusable KQL library allows the same techniques to be applied across CPU, heartbeat, availability, Windows events, disk capacity, memory pressure, and automated alerting scenarios.
