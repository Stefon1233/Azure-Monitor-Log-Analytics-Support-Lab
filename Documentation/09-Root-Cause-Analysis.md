# 09 — Root-Cause Analysis

## Azure Monitor + Log Analytics Support Lab

---

## Overview

Root-cause analysis, or RCA, is the process of identifying the underlying reason an incident occurred rather than stopping at the visible symptom.

This Azure Monitor + Log Analytics Support Lab uses RCA throughout seven controlled incidents involving:

```text
CPU utilization
Monitoring heartbeat
Virtual machine availability
Windows application events
Disk capacity
Memory pressure
KQL-based alerting
```

The purpose of RCA in this project is to demonstrate a repeatable support workflow:

```text
Symptom
      ↓
Evidence Collection
      ↓
Correlation
      ↓
Cause Identification
      ↓
Cause Validation
      ↓
Remediation
      ↓
Recovery Validation
      ↓
Incident Closure
```

The monitored server was:

```text
AZMON-WIN01
```

with monitoring provided through:

```text
Azure Monitor
Azure Monitor Agent
DCR-AZMON-WINDOWS
LAW-AZMON-SUPPORT-LAB
Kusto Query Language
Azure Activity Log
Azure Alerts
Windows Server telemetry
PowerShell
```

---

# RCA Objectives

The objectives of the RCA phase were to demonstrate how to:

- Separate symptoms from causes
- Avoid assumptions
- Use monitoring evidence to support conclusions
- Correlate multiple telemetry sources
- Identify the exact change associated with an incident
- Determine whether a problem exists in Azure, Windows, monitoring, or alert logic
- Validate causality through controlled testing
- Apply remediation to the cause
- Confirm the problem does not persist after remediation
- Document a defensible root-cause statement

---

# Environment

## Virtual Machine

```text
Name:
AZMON-WIN01

Operating System:
Windows Server 2022 Datacenter: Azure Edition

Region:
North Central US

VM Size:
Standard_D2als_v6

vCPUs:
2

Memory:
4 GiB
```

---

## Resource Group

```text
RG-AZMON-SUPPORT-LAB
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

---

# What Root Cause Means

A symptom describes what is observed.

A root cause explains why it happened.

Example:

```text
Symptom:
CPU usage above 80%

Root Cause:
Controlled PowerShell CPU workload
```

Another example:

```text
Symptom:
Available memory below 2048 MB

Root Cause:
Controlled PowerShell memory worker
```

---

# Symptom vs Root Cause

A common troubleshooting mistake is to label the symptom as the cause.

Example:

```text
Incorrect RCA:
The server had high CPU because CPU was high.
```

A useful RCA should instead identify the underlying driver.

```text
Correct RCA:
Two controlled PowerShell workers produced sustained processor utilization that caused Azure Percentage CPU to exceed the configured 80% threshold.
```

---

# RCA Evidence Standard

A root-cause statement should be supported by evidence.

The lab used this general model:

```text
Known Change
      ↓
Observable Symptom
      ↓
Monitoring Confirms Symptom
      ↓
Cause Removed
      ↓
Symptom Clears
      ↓
Monitoring Confirms Recovery
```

---

# Strong RCA Evidence

Strong root-cause evidence may include:

```text
Process state
Resource state
Performance metrics
Log Analytics records
Heartbeat records
Activity Log operations
Windows events
Alert timestamps
Remediation output
Recovery telemetry
```

---

# Weak RCA Evidence

Weak RCA conclusions include statements such as:

```text
Probably caused by Azure
Likely network-related
Maybe an agent issue
CPU was high
Disk was low
```

without evidence showing:

```text
What changed
When it changed
Why that change produced the symptom
What happened after the suspected cause was removed
```

---

# RCA Methodology

The project used a repeatable RCA workflow.

---

# Step 1 — Define the Symptom

Start by stating exactly what was observed.

Examples:

```text
Percentage CPU > 80%
Heartbeat missing
VM unavailable
Windows error event detected
Disk free space reduced by ~10 GB
Available memory reduced by ~1 GB
Low-memory scheduled query returned a row
```

---

# Step 2 — Establish the Normal State

Determine what healthy behavior looks like.

Examples:

```text
CPU below threshold
Heartbeat arriving regularly
VM running
Disk free space ~112 GB
Available memory ~2.8–2.9 GB
KQL alert condition returns 0 rows
```

---

# Step 3 — Identify What Changed

Ask:

```text
What occurred immediately before the symptom?
```

Possible changes include:

```text
Process started
File created
VM deallocated
Agent interrupted
Event generated
Configuration changed
Alert rule evaluated
```

---

# Step 4 — Correlate Timing

The timing of the suspected cause should align with the symptom.

Example:

```text
Memory worker starts
      ↓
Available memory falls
      ↓
Log Analytics reports low memory
```

If timing does not correlate, the suspected cause should be reconsidered.

---

# Step 5 — Correlate Multiple Sources

A stronger RCA uses more than one source.

Example:

```text
Windows process working set
+
Windows available memory
+
Log Analytics Perf
+
KQL timeline
```

---

# Step 6 — Test the Hypothesis

Where safe, the lab used controlled changes.

Example:

```text
Suspected cause:
Controlled memory worker

Test:
Stop the worker

Expected result:
Available memory recovers
```

Observed result:

```text
Memory recovered
```

This supports causality.

---

# Step 7 — Remove the Cause

Remediation targeted the underlying cause.

Examples:

```text
Controlled workload ended
Monitoring restored
VM started
Temporary file deleted
Memory worker stopped
```

---

# Step 8 — Confirm the Symptom Clears

After remediation:

```text
CPU falls
Heartbeat returns
VM becomes available
Disk free space returns
Memory recovers
Alert query clears
```

---

# Step 9 — Confirm Azure Observes Recovery

A local fix is not sufficient.

Recovery should also appear in:

```text
Azure Monitor
Log Analytics
Heartbeat
Alert state
```

---

# Step 10 — Write the RCA Statement

A complete RCA should include:

```text
Affected resource
Observed symptom
Underlying cause
Evidence
Remediation
Recovery confirmation
```

---

# RCA Template

A reusable RCA format is:

```text
Root Cause:
[Specific underlying cause]

Evidence:
[Telemetry / logs / process state / activity]

Impact:
[What condition was produced]

Remediation:
[What was changed]

Recovery:
[What evidence confirmed normal state]
```

---

# Causality Model

The project repeatedly used this model:

```text
Cause
      ↓
Resource State Changes
      ↓
Monitoring Detects Change
      ↓
Alert / Incident Generated
      ↓
Cause Removed
      ↓
Resource Recovers
      ↓
Monitoring Confirms Recovery
```

---

# Monitoring Layers in RCA

The lab separated several monitoring layers.

```text
Azure Resource
Guest Operating System
Azure Monitor Agent
Data Collection Rule
Log Analytics
KQL Query
Alert Rule
```

A failure can occur at any of these layers.

---

# Azure Resource Layer

Examples:

```text
VM deallocated
VM started
Resource operation performed
```

Evidence:

```text
Azure Portal
Activity Log
VM resource state
```

---

# Guest Operating System Layer

Examples:

```text
CPU workload
Memory workload
Disk consumption
Windows event
```

Evidence:

```text
PowerShell
Windows performance data
Processes
Local disk state
Event Log
```

---

# Monitoring Agent Layer

Examples:

```text
Heartbeat interruption
Missing guest telemetry
```

Evidence:

```text
Heartbeat
AMA state
Telemetry timestamps
```

---

# Collection Layer

Examples:

```text
Wrong counter expected
Counter not configured
DCR issue
```

Evidence:

```text
Perf inventory
DCR configuration
Recent samples
```

---

# Query Layer

Examples:

```text
Incorrect CounterName
Incorrect time range
Incorrect Computer filter
```

Evidence:

```text
KQL query results
Counter inventory
Known-good alternate query
```

---

# Alert Layer

Examples:

```text
Alert rule fires
Alert rule fails to fire
Alert resolves
```

Evidence:

```text
Alert rule configuration
Threshold
Query results
Alert instance
Evaluation window
```

---

# INC-001 — High CPU RCA

Incident:

```text
INC-001 — High CPU Utilization
```

Affected resource:

```text
AZMON-WIN01
```

---

# INC-001 Symptom

Azure Monitor detected:

```text
Percentage CPU > 80%
```

The metric alert:

```text
ALRT-AZMON-HIGH-CPU
```

entered:

```text
FIRED
```

---

# INC-001 Known Change

A controlled PowerShell workload was intentionally started.

Two worker processes performed repeated mathematical calculations.

---

# INC-001 Correlation

```text
PowerShell workers started
      ↓
Processor demand increased
      ↓
Azure Percentage CPU increased
      ↓
CPU exceeded 80%
      ↓
Metric alert fired
```

---

# INC-001 Root Cause

```text
Controlled PowerShell CPU workload
```

---

# INC-001 Root-Cause Evidence

Evidence included:

```text
Known workload start
Azure Percentage CPU increase
Alert threshold violation
ALRT-AZMON-HIGH-CPU fired
Post-test worker count = 0
CPU recovery
Alert resolution
```

---

# INC-001 Remediation

The controlled workload used an automatic timeout.

No active worker remained after the test.

---

# INC-001 Recovery

Azure Monitor later showed:

```text
CPU returned below threshold
Alert resolved
```

---

# INC-001 RCA Statement

```text
AZMON-WIN01 experienced sustained high CPU because two intentionally launched PowerShell worker processes generated processor-intensive calculations. Azure Percentage CPU exceeded the configured
80% threshold and ALRT-AZMON-HIGH-CPU fired. The workload reached its configured timeout, no controlled workers remained, CPU utilization returned toward baseline, and Azure Monitor automatically
resolved the alert.
```

---

# INC-001 Lesson

Do not confuse:

```text
Alert detection
```

with:

```text
Root cause
```

The alert detected high CPU.

The PowerShell workload caused high CPU.

---

# INC-002 — Missing Heartbeat RCA

Incident:

```text
INC-002 — Missing Heartbeat
```

---

# INC-002 Symptom

Observed:

```text
Heartbeat telemetry stopped
```

---

# Initial Possibilities

Possible causes included:

```text
VM offline
AMA interruption
Network issue
DCR issue
Monitoring pipeline issue
```

---

# Resource State Check

The VM state was checked independently from heartbeat.

This helped determine whether:

```text
resource availability
```

and:

```text
monitoring availability
```

were the same issue.

---

# INC-002 Root Cause

The controlled incident represented:

```text
Azure Monitor Agent / monitoring interruption
```

rather than actual VM deallocation.

---

# INC-002 Correlation

```text
VM remains available
      ↓
Heartbeat stops
      ↓
Monitoring telemetry gap appears
      ↓
Monitoring restored
      ↓
Heartbeat resumes
```

---

# INC-002 Root-Cause Evidence

Evidence included:

```text
VM remained available
Heartbeat gap
Monitoring pipeline interruption
Heartbeat restored after recovery
```

---

# INC-002 RCA Statement

```text
AZMON-WIN01 experienced a monitoring heartbeat interruption while the virtual machine itself remained available. Resource state and heartbeat telemetry were compared to distinguish a monitoring
failure from an infrastructure outage. Heartbeat resumed after the monitoring path recovered, confirming the incident was caused by a temporary monitoring interruption rather than VM
unavailability.
```

---

# INC-002 Lesson

```text
Missing heartbeat
```

is a symptom.

It does not automatically identify:

```text
VM outage
```

as the cause.

---

# INC-003 — VM Availability RCA

Incident:

```text
INC-003 — Virtual Machine Availability
```

---

# INC-003 Symptom

Observed:

```text
VM unavailable
Heartbeat stopped
```

---

# INC-003 Known Change

The VM was intentionally deallocated.

---

# Activity Log Evidence

Azure Activity Log provided evidence of the administrative VM operation.

This was critical because it showed:

```text
Why the VM became unavailable
```

rather than only:

```text
That the VM was unavailable
```

---

# INC-003 Correlation

```text
VM deallocation initiated
      ↓
VM stops
      ↓
Heartbeat stops
      ↓
Activity Log records operation
      ↓
VM restarted
      ↓
Heartbeat resumes
```

---

# INC-003 Root Cause

```text
Controlled Azure VM deallocation
```

---

# INC-003 RCA Statement

```text
AZMON-WIN01 became unavailable because the virtual machine was intentionally deallocated through an Azure control-plane operation. The unavailable VM state, missing heartbeat, and Activity Log
record aligned in time. After the VM was started again, resource availability and heartbeat telemetry recovered, confirming deallocation as the root cause.
```

---

# INC-003 Lesson

Activity Log is especially valuable for answering:

```text
Who or what changed the Azure resource state?
```

---

# INC-004 — Windows Application Event RCA

Incident:

```text
INC-004 — Windows Application Event Investigation
```

---

# INC-004 Symptom

Log Analytics received a Windows Application error.

Event:

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

---

# INC-004 Known Change

The event was intentionally generated as a controlled monitoring test.

---

# INC-004 Correlation

```text
Controlled event generated
      ↓
Windows Application log records event
      ↓
Monitoring pipeline collects event
      ↓
Log Analytics receives event
      ↓
KQL locates Event ID 200
```

---

# INC-004 Root Cause

```text
Controlled Windows Application event generated for telemetry validation
```

---

# INC-004 RCA Statement

```text
The Windows Application error detected for AZMON-WIN01 was caused by an intentionally generated test event from source AZMON-AppLab with Event ID 200. The event metadata in Log Analytics matched
the locally generated event, confirming the source and ruling out an unknown application failure.
```

---

# INC-004 Lesson

An event entry is not always proof of a broader application outage.

RCA must determine:

```text
What generated the event
```

and:

```text
Whether it represents an actual service impact
```

---

# INC-005 — Disk Space RCA

Incident:

```text
INC-005 — Disk Space / Storage Capacity Monitoring
```

---

# INC-005 Baseline

```text
Total:
126.45 GB

Free:
112.26 GB

Free:
88.78%
```

---

# INC-005 Symptom

Available disk capacity decreased by approximately:

```text
10 GB
```

---

# INC-005 Known Change

A controlled file was created:

```text
C:\AZMON-Disk-Test.bin
```

Size:

```text
10 GB
```

---

# INC-005 Correlation

```text
10 GB file created
      ↓
Windows free space decreases
      ↓
LogicalDisk\Free Megabytes decreases
      ↓
KQL timeline shows ~10 GB drop
```

---

# INC-005 Root Cause

```text
Controlled 10 GB test file
```

---

# INC-005 Remediation

The file was deleted.

---

# INC-005 Recovery

Windows returned to approximately:

```text
112.26 GB free
```

Log Analytics reported the restored capacity.

---

# INC-005 RCA Statement

```text
AZMON-WIN01 experienced an approximately 10 GB reduction in available C: drive capacity because the controlled file C:\AZMON-Disk-Test.bin consumed 10 GB of storage. Windows disk measurements and
Log Analytics LogicalDisk\Free Megabytes both recorded the reduction. Deleting the file restored approximately the same capacity, confirming the file as the direct root cause.
```

---

# INC-005 Lesson

The strongest evidence was the symmetry between:

```text
10 GB created
```

and:

```text
~10 GB recovered
```

after deletion.

---

# INC-006 — Memory Pressure RCA

Incident:

```text
INC-006 — Memory Pressure / High Memory Usage
```

---

# INC-006 Baseline

Windows:

```text
Total Memory:
3.99 GB

Available:
2.79 GB

Used:
30.23%
```

Log Analytics:

```text
~2933 MB available
```

---

# Initial Query Failure

The first query searched for:

```text
Memory\Available MBytes
```

and returned no results.

---

# Initial RCA Possibilities

Possible causes included:

```text
No memory collection
AMA failure
DCR failure
Wrong counter name
Wrong query
```

---

# Counter Inventory

KQL counter inventory showed:

```text
Memory\Available Bytes
```

was being collected.

---

# Query Root Cause

The no-results condition was caused by:

```text
Incorrect CounterName filter
```

not:

```text
Azure Monitor failure
```

---

# Controlled Memory Workload

A PowerShell worker allocated approximately:

```text
1024 MB
```

Observed worker memory:

```text
~1105 MB
```

---

# Memory Symptom

Windows showed:

```text
Available:
1.73 GB

Memory Used:
56.59%
```

Log Analytics showed:

```text
~1809–1827 MB available
```

---

# INC-006 Correlation

```text
Worker starts
      ↓
Worker consumes ~1105 MB
      ↓
Available memory decreases ~1 GB
      ↓
Perf confirms reduction
      ↓
Worker stops
      ↓
Available memory recovers
```

---

# INC-006 Root Cause

```text
Controlled PowerShell memory allocation worker
```

---

# INC-006 Remediation

The memory worker was stopped.

Temporary artifacts were removed.

---

# INC-006 Recovery

Windows:

```text
Available:
2.77 GB

WorkerPresent:
False
```

Log Analytics:

```text
~2915 MB available
```

---

# INC-006 RCA Statement

```text
AZMON-WIN01 experienced controlled memory pressure because a PowerShell worker intentionally allocated approximately 1 GB of memory. The worker working set reached approximately 1105 MB, Windows
available memory fell from roughly 2.79 GB to 1.73 GB, and Log Analytics independently recorded approximately 1.8 GB available. After the worker was stopped, Windows and Log Analytics both
returned near baseline, confirming the worker as the root cause.
```

---

# INC-006 Secondary RCA

A second troubleshooting issue occurred during the incident:

```text
KQL returned no memory data
```

Root cause:

```text
Query searched for Available MBytes
while the DCR collected Available Bytes
```

Resolution:

```text
Correct KQL filter
```

No DCR change was required.

---

# INC-006 Lesson

Not every troubleshooting problem requires a configuration change.

The safest path is:

```text
Validate existing telemetry first
```

---

# INC-007 — Scheduled Query Alert RCA

Incident:

```text
INC-007 — Log Analytics Scheduled Query Alert / Low Memory
```

---

# INC-007 Baseline

Available memory:

```text
2911 MB
```

Alert condition:

```text
Available memory < 2048 MB
```

Initial query:

```text
0 rows
```

---

# INC-007 Known Change

A controlled memory workload was started.

---

# INC-007 Symptom

Log Analytics later reported:

```text
1691 MB
```

which satisfied:

```text
1691 < 2048
```

---

# Query State

The scheduled query changed from:

```text
0 rows
```

to:

```text
1 row
```

---

# Alert State

Azure Monitor transitioned to:

```text
FIRED
```

---

# INC-007 Correlation

```text
Memory workload starts
      ↓
Available memory falls below 2048 MB
      ↓
KQL returns matching row
      ↓
Scheduled query condition true
      ↓
Alert fires
```

---

# INC-007 Root Cause

```text
Controlled PowerShell memory workload
```

---

# INC-007 Remediation

The controlled process was stopped.

Windows showed:

```text
AvailableMemoryMB:
2696

Above2048MB:
True
```

---

# INC-007 Log Analytics Recovery

New telemetry showed:

```text
2764 MB
HEALTHY
```

---

# INC-007 Alert Recovery

The exact alert query returned:

```text
No results
```

The current alert later changed from:

```text
FIRED
```

to:

```text
RESOLVED
```

---

# INC-007 RCA Statement

```text
ALRT-AZMON-LOW-MEMORY-KQL fired because a controlled PowerShell workload reduced AZMON-WIN01 available memory below the configured 2048 MB threshold. Log Analytics recorded 1691 MB and the
scheduled query returned a matching row. After the worker was stopped, available memory recovered to 2696 MB locally and 2764 MB in Log Analytics, the query returned no rows, and Azure Monitor
automatically resolved the current alert.
```

---

# INC-007 Lesson

An alert state may remain:

```text
FIRED
```

for a short period after the underlying resource recovers.

This does not invalidate the remediation.

The alert engine must complete:

```text
New telemetry ingestion
      ↓
Scheduled query evaluation
      ↓
Alert state processing
```

---

# Root Cause Matrix

| Incident | Symptom | Root Cause | Key Evidence | Resolution |
|---|---|---|---|---|
| INC-001 | CPU > 80% | Controlled CPU workload | Percentage CPU + alert | Workload timeout |
| INC-002 | Missing Heartbeat | Monitoring interruption | VM state + Heartbeat | Monitoring restored |
| INC-003 | VM unavailable | Controlled deallocation | VM state + Activity Log | VM started |
| INC-004 | Application error event | Controlled Event ID 200 | Windows event + Log Analytics | Investigation completed |
| INC-005 | Disk free space reduced | 10 GB test file | Windows + LogicalDisk Perf | File deleted |
| INC-006 | Available memory reduced | 1 GB memory worker | Process + Windows + Perf | Worker stopped |
| INC-007 | KQL low-memory alert | Controlled memory workload | Perf + KQL + alert | Worker stopped |

---

# Common RCA Patterns

Several patterns were repeated across incidents.

---

# Pattern 1 — Known Controlled Change

Controlled incident generation provided a known change.

Examples:

```text
CPU workers started
VM deallocated
Test file created
Memory worker started
Windows event generated
```

This enabled precise cause-and-effect validation.

---

# Pattern 2 — Monitoring Confirmation

Each controlled change was validated through monitoring.

Examples:

```text
CPU metric rises
Heartbeat disappears
Activity Log records operation
Disk counter decreases
Memory counter decreases
KQL query matches
Alert fires
```

---

# Pattern 3 — Cause Removal

The suspected cause was removed or allowed to terminate.

---

# Pattern 4 — Symptom Reversal

After removing the cause, the symptom reversed.

Examples:

```text
CPU decreases
Heartbeat returns
VM becomes available
Disk space returns
Memory returns
Alert clears
```

---

# Pattern 5 — Azure-Side Recovery

Azure monitoring subsequently confirmed the recovered state.

---

# RCA and False Assumptions

The lab demonstrates several assumptions that should be avoided.

---

# Assumption 1

```text
No query results = agent failure
```

Incorrect.

INC-006 showed the problem may be:

```text
Wrong counter name
```

---

# Assumption 2

```text
Missing heartbeat = VM offline
```

Incorrect.

INC-002 showed monitoring can fail while the VM remains available.

---

# Assumption 3

```text
Alert fired = root cause identified
```

Incorrect.

An alert describes a condition.

The investigation must identify the cause.

---

# Assumption 4

```text
Remediation command succeeded = incident resolved
```

Incorrect.

Monitoring recovery must still be validated.

---

# Assumption 5

```text
Resolved alert = current incident
```

Not always.

INC-007 showed multiple instances of the same rule could appear.

Timestamp correlation is required.

---

# RCA and Time Correlation

Time correlation is one of the strongest forms of operational evidence.

Example:

```text
04:23 UTC:
Healthy memory

04:24 UTC:
Controlled memory pressure visible

04:24–04:29 UTC:
Low-memory condition sustained

04:30 UTC:
Recovery visible
```

This timeline aligns:

```text
Cause
Incident
Remediation
Recovery
```

---

# RCA and Change History

In production, RCA should review recent changes such as:

```text
Deployments
Patches
Configuration changes
Scaling events
User changes
Network changes
Service restarts
Automation
Administrative operations
```

Azure Activity Log can help identify:

```text
Who
What
When
Status
```

for control-plane operations.

---

# RCA and Process Investigation

For guest performance issues, process-level data is important.

Examples:

```text
CPU-consuming process
Memory-consuming process
Unexpected service
Scheduled task
Backup process
Security scan
```

INC-006 directly correlated:

```text
Worker working set
```

with:

```text
Memory reduction
```

---

# RCA and Capacity Analysis

Capacity problems may involve:

```text
Disk
Memory
CPU
Network
```

A complete RCA should distinguish:

```text
Temporary spike
Sustained demand
Resource leak
Insufficient sizing
Controlled workload
```

---

# Production CPU RCA Questions

```text
Which process consumes CPU?
When did utilization rise?
Was there a deployment?
Is the load expected?
Is CPU sustained?
Did the alert repeat?
Is VM sizing adequate?
```

---

# Production Memory RCA Questions

```text
Which process consumes RAM?
Is the working set growing?
Is there a memory leak?
Is paging increasing?
Did a service restart?
Did workload volume increase?
Is the VM undersized?
```

---

# Production Disk RCA Questions

```text
What consumed the space?
Was a log file growing?
Was a backup created?
Was a dump file generated?
Was retention misconfigured?
Is disk growth expected?
```

---

# Production Heartbeat RCA Questions

```text
Is the VM running?
Is AMA running?
Is network connectivity available?
Is the DCR associated?
Are other tables receiving data?
Did the VM reboot?
```

---

# Production Availability RCA Questions

```text
Was the VM stopped?
Was it deallocated?
Did an administrator perform an action?
Was there Azure maintenance?
Did networking fail?
Was the guest OS responsive?
```

---

# Production Alert RCA Questions

```text
Did the monitored condition actually occur?
Did the query return rows?
Was the rule enabled?
Was the scope correct?
Was the evaluation window correct?
Was telemetry delayed?
Was the alert instance current?
```

---

# Root Cause vs Contributing Factors

A root cause is the primary underlying cause.

Contributing factors may increase impact.

Example:

```text
Root Cause:
Memory leak

Contributing Factor:
VM undersized
```

The controlled incidents generally had one intentional primary cause, but production incidents may involve several contributing conditions.

---

# Primary Cause

The primary cause should answer:

```text
What directly produced the incident?
```

---

# Contributing Factor

A contributing factor answers:

```text
What made the incident more likely or more severe?
```

---

# Remediation vs Prevention

Remediation resolves the current incident.

Prevention reduces recurrence.

Example:

```text
Remediation:
Stop runaway process

Prevention:
Fix application memory leak
```

---

# Production Preventive Actions

Possible actions include:

```text
Resize VM
Tune alert thresholds
Correct application defect
Change retention settings
Add monitoring
Improve automation
Add maintenance exclusions
Apply patch
Create runbook
Improve dashboards
```

---

# Corrective Action Validation

A corrective action should be validated.

Example:

```text
Action:
Delete large temporary file

Expected:
Disk free space increases

Observed:
~10 GB recovered
```

---

# RCA Confidence Levels

In production, RCA conclusions may have different confidence levels.

## Confirmed

```text
Direct evidence shows cause and recovery.
```

---

## Probable

```text
Evidence strongly suggests the cause but direct confirmation is unavailable.
```

---

## Unknown

```text
Evidence is insufficient to identify a defensible cause.
```

The lab incidents were designed to produce:

```text
Confirmed
```

root causes.

---

# Avoiding Unsupported RCA

If the evidence cannot support a cause, the correct conclusion may be:

```text
Root cause undetermined
```

rather than inventing a cause.

---

# Evidence Preservation

RCA evidence was preserved using:

```text
Screenshots
Markdown incident documentation
KQL query files
Git commits
```

---

# Why Git Helps RCA Documentation

Git provides:

```text
Change history
Commit timestamps
Version control
Documentation traceability
```

This helps demonstrate how the investigation evolved.

---

# Evidence Directory Structure

Incident evidence is organized under:

```text
Screenshots/
```

with dedicated incident folders.

---

# Completed Incident Evidence

Examples:

```text
06-Alerts
07-Missing-Heartbeat
08-VM-Availability
09-Windows-Events
10-Disk-Space
11-Memory-Pressure
12-Log-Query-Alert
```

---

# RCA Documentation Standard

Each incident should preserve:

```text
Baseline
Incident condition
Root-cause evidence
Remediation
Recovery
Final state
```

---

# Root-Cause Statement Quality

A strong statement should be:

```text
Specific
Evidence-based
Concise
Technically accurate
Connected to remediation
Connected to recovery
```

---

# Poor Root-Cause Statement

```text
The server had memory issues.
```

---

# Better Root-Cause Statement

```text
A PowerShell worker consumed approximately 1.1 GB of working set memory, reducing available memory from approximately 2.8 GB to 1.7 GB. Stopping the worker restored available memory to
approximately 2.8 GB.
```

---

# Root Cause and Monitoring Configuration

Not every incident requires monitoring changes.

INC-006 demonstrated this clearly.

Problem:

```text
No results for Available MBytes
```

Correct response:

```text
Inspect collected counters
```

Finding:

```text
Available Bytes already collected
```

Resolution:

```text
Correct query
```

---

# Avoiding Configuration Drift

Unnecessary changes can introduce:

```text
Configuration drift
Additional ingestion
Unexpected cost
Operational complexity
New failure points
```

RCA should determine whether configuration change is actually necessary.

---

# Alert Root-Cause Analysis

An alert investigation should separate:

```text
Why the resource condition occurred
```

from:

```text
Why the alert fired
```

For INC-007:

```text
Resource Cause:
Controlled memory allocation

Alert Cause:
Available memory dropped below 2048 MB and KQL returned a row
```

Both are useful but describe different layers.

---

# Alert Resolution RCA

The alert resolved because:

```text
Available memory recovered
      ↓
Latest KQL record became healthy
      ↓
Query returned 0 rows
      ↓
Alert condition cleared
      ↓
Azure Monitor resolved alert
```

---

# Monitoring Pipeline RCA

When telemetry is missing, investigate:

```text
Resource
      ↓
Agent
      ↓
DCR
      ↓
Network path
      ↓
Workspace ingestion
      ↓
Query
```

---

# Resource Healthy, Telemetry Missing

Possible causes:

```text
Agent issue
DCR issue
Network issue
Ingestion delay
Query error
```

---

# Resource Unhealthy, Telemetry Missing

Possible cause:

```text
Actual resource outage
```

This is why resource state must be checked separately.

---

# RCA Decision Tree

```text
Symptom observed
      ↓
Is the resource available?
      ├── No
      │     ↓
      │  Check Activity Log / resource operations
      │
      └── Yes
            ↓
      Is telemetry current?
            ├── No
            │     ↓
            │  Check AMA / DCR / ingestion
            │
            └── Yes
                  ↓
            Does telemetry confirm symptom?
                  ├── No
                  │     ↓
                  │  Check query / alert logic
                  │
                  └── Yes
                        ↓
                  Identify process / change / operation
                        ↓
                  Remediate cause
                        ↓
                  Validate recovery
```

---

# Five Whys Concept

A simplified Five Whys approach can help.

Example:

```text
Why did the alert fire?
Because available memory was below 2048 MB.

Why was available memory below 2048 MB?
Because a process consumed approximately 1 GB.

Why did that process consume 1 GB?
Because a controlled PowerShell memory test was running.

Why was the test running?
To validate scheduled query alerting.

Why did the alert resolve?
Because the process was stopped and memory returned above threshold.
```

---

# Five Whys Limitations

The technique should not be forced when evidence does not support the chain.

Use:

```text
Logs
Telemetry
Change history
Process state
```

to validate each step.

---

# RCA Timeline

A useful RCA timeline should include:

```text
Baseline timestamp
Incident start
Detection time
Alert time
Investigation time
Remediation time
Recovery time
Resolution time
```

---

# Production Post-Incident Review

After major incidents, teams may review:

```text
Impact
Duration
Detection
Root cause
Contributing factors
Response
Remediation
Prevention
Monitoring gaps
Communication
```

---

# Detection Effectiveness

Ask:

```text
Did monitoring detect the incident quickly enough?
```

---

# Diagnostic Effectiveness

Ask:

```text
Was enough telemetry available to identify the cause?
```

---

# Remediation Effectiveness

Ask:

```text
Did the remediation directly address the cause?
```

---

# Recovery Effectiveness

Ask:

```text
Was recovery verified from multiple perspectives?
```

---

# Monitoring Improvements

Potential improvements after RCA can include:

```text
New alert
Better threshold
Additional counter
New KQL query
New dashboard
Heartbeat alert
Action group change
Automation
```

---

# Lessons Learned Across All Incidents

## Lesson 1

```text
Do not equate symptoms with causes.
```

---

## Lesson 2

```text
Use independent evidence sources.
```

---

## Lesson 3

```text
Correlate timestamps.
```

---

## Lesson 4

```text
Validate monitoring before modifying it.
```

---

## Lesson 5

```text
Remediate the cause, not the alert.
```

---

## Lesson 6

```text
Verify recovery in Azure, not only locally.
```

---

## Lesson 7

```text
Do not use stale alert instances as closure evidence.
```

---

## Lesson 8

```text
A no-results query can be a query problem.
```

---

## Lesson 9

```text
Activity Log is critical for control-plane RCA.
```

---

## Lesson 10

```text
Controlled tests should be reversible and measurable.
```

---

# Skills Demonstrated

This RCA phase demonstrates:

- Root-cause analysis
- Azure Monitor
- Log Analytics
- KQL
- Azure Monitor Agent
- Data Collection Rules
- Azure Activity Log
- Azure Alerts
- Windows Server
- PowerShell
- Process investigation
- CPU troubleshooting
- Memory troubleshooting
- Disk troubleshooting
- VM availability analysis
- Heartbeat troubleshooting
- Windows event analysis
- Alert investigation
- Telemetry correlation
- Timeline analysis
- Remediation validation
- Incident closure
- Evidence preservation
- Change correlation
- Technical documentation

---

# Root-Cause Checklist

```text
[ ] Define symptom
[ ] Define affected resource
[ ] Establish baseline
[ ] Identify timestamp
[ ] Identify recent change
[ ] Validate resource state
[ ] Validate monitoring state
[ ] Query relevant telemetry
[ ] Correlate multiple sources
[ ] Develop root-cause hypothesis
[ ] Validate hypothesis
[ ] Identify corrective action
[ ] Perform remediation
[ ] Validate local recovery
[ ] Validate Azure recovery
[ ] Validate alert recovery
[ ] Write RCA statement
[ ] Record preventive actions
[ ] Preserve evidence
[ ] Close incident
```

---

# RCA Validation by Incident

```text
INC-001:
Root Cause CONFIRMED
Controlled CPU workload

INC-002:
Root Cause CONFIRMED
Monitoring interruption

INC-003:
Root Cause CONFIRMED
Controlled VM deallocation

INC-004:
Root Cause CONFIRMED
Controlled Windows Application event

INC-005:
Root Cause CONFIRMED
Controlled 10 GB file

INC-006:
Root Cause CONFIRMED
Controlled memory worker

INC-007:
Root Cause CONFIRMED
Controlled low-memory workload
```

---

# Final Root-Cause Model

The completed project demonstrates:

```text
Observe
      ↓
Measure
      ↓
Correlate
      ↓
Hypothesize
      ↓
Validate
      ↓
Remediate
      ↓
Re-measure
      ↓
Confirm
      ↓
Document
```

---

# Conclusion

Root-cause analysis is the point where monitoring becomes operational troubleshooting.

Azure Monitor, Log Analytics, KQL, Windows telemetry, and Activity Log provide evidence, but the support engineer must still determine:

```text
What actually changed
Why that change produced the symptom
Which layer failed
What action will remove the cause
Whether the system recovered afterward
```

Across seven incidents, this lab demonstrates confirmed RCA for:

```text
High CPU
Missing heartbeat
VM availability
Windows application events
Disk capacity
Memory pressure
Scheduled query alerting
```

The project therefore demonstrates a complete Azure support lifecycle:

```text
Monitor
      ↓
Detect
      ↓
Investigate
      ↓
Identify Root Cause
      ↓
Remediate
      ↓
Validate Recovery
      ↓
Resolve
      ↓
Document
```
