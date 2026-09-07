# INC-007 — Log Analytics Scheduled Query Alert / Low Memory

## Azure Monitor + Log Analytics Support Lab

---

## Incident Summary

**Incident ID:** INC-007
**Title:** Log Analytics Scheduled Query Alert / Low Memory
**Affected Resource:** AZMON-WIN01
**Resource Group:** RG-AZMON-SUPPORT-LAB
**Log Analytics Workspace:** LAW-AZMON-SUPPORT-LAB
**Data Collection Rule:** DCR-AZMON-WINDOWS
**Alert Rule:** ALRT-AZMON-LOW-MEMORY-KQL
**Action Group:** AG-AZMON-SUPPORT
**Platform:** Microsoft Azure
**Operating System:** Windows Server 2022 Datacenter: Azure Edition
**Severity:** Severity 2 — Warning
**Incident Type:** Performance Monitoring / Automated Detection
**Alert Type:** Log Analytics Scheduled Query Rule
**Memory Counter:** Memory\Available Bytes
**Alert Threshold:** Available memory below 2048 MB
**Evaluation Frequency:** 1 minute
**Automatic Resolution:** Enabled
**Status:** Resolved

---

# Executive Summary

INC-007 extended the Azure Monitor + Log Analytics Support Lab from manual monitoring into automated incident detection.

Previous incidents demonstrated how support engineers can manually investigate performance data stored in Log Analytics.

INC-007 added:

```text
KQL-based automated monitoring
Scheduled query alerting
Action group integration
Controlled incident generation
Alert firing
Incident remediation
Automatic alert resolution
```

The monitored resource was:

```text
AZMON-WIN01
```

The memory counter used was:

```text
Memory\Available Bytes
```

A healthy baseline query showed approximately:

```text
2911 MB available
2.84 GB available
```

A KQL detection condition was designed to return a row when available physical memory fell below:

```text
2048 MB
```

Before the incident, the alert query returned:

```text
No results found
```

This confirmed:

```text
Current memory state:
HEALTHY

Alert condition:
FALSE
```

A scheduled query rule was then configured:

```text
ALRT-AZMON-LOW-MEMORY-KQL
```

The rule monitored:

```text
LAW-AZMON-SUPPORT-LAB
```

and used:

```text
Table rows > 0
```

as the alert condition.

The rule evaluated the KQL query approximately every:

```text
1 minute
```

The existing Azure Monitor action group:

```text
AG-AZMON-SUPPORT
```

was associated with the rule.

Automatic alert resolution was enabled.

A controlled memory-pressure workload was then started on:

```text
AZMON-WIN01
```

The workload reduced available memory below the configured threshold.

Log Analytics reported:

```text
AvailableMB:
1691
```

The detection query changed from:

```text
0 rows
```

to:

```text
1 matching row
```

and the scheduled query alert fired.

The alert instance was verified in Azure Monitor and its fired-state details were captured.

The controlled memory worker was then stopped.

Windows immediately reported:

```text
WorkerStopped:
True

AvailableMemoryMB:
2696

AvailableMemoryGB:
2.63

Above2048MB:
True

Status:
MEMORY PRESSURE CLEARED
```

A subsequent Log Analytics sample reported:

```text
2764 MB
HEALTHY
```

The complete monitored sequence showed:

```text
Healthy:
2719 MB

Healthy:
2680 MB

Healthy:
2692 MB

Low Memory:
1691 MB

Low Memory:
1696 MB

Low Memory:
1702 MB

Low Memory:
1664 MB

Low Memory:
1673 MB

Low Memory:
1676 MB

Recovered:
2764 MB
```

The exact scheduled-query condition later returned:

```text
No results found
```

confirming that the alert condition was no longer true.

Azure Monitor subsequently transitioned the current alert from:

```text
Fired
```

to:

```text
Resolved
```

This demonstrated the complete monitoring lifecycle:

```text
Healthy telemetry
        ↓
KQL condition false
        ↓
Scheduled query rule enabled
        ↓
Controlled memory pressure
        ↓
Available memory < 2048 MB
        ↓
KQL condition true
        ↓
Scheduled query alert fired
        ↓
Support investigation
        ↓
Memory workload stopped
        ↓
Available memory recovered
        ↓
KQL condition false
        ↓
Alert automatically resolved
```

INC-007 demonstrates how Azure Monitor and Log Analytics can move from reactive investigation to automated operational monitoring.

---

# Incident Objectives

The objectives of INC-007 were to:

- Establish a healthy memory baseline
- Reuse the Memory\Available Bytes counter identified in INC-006
- Build a KQL query capable of detecting low available memory
- Verify the detection query returned no rows during healthy operation
- Create a Log Analytics scheduled query alert
- Configure the correct monitoring scope
- Configure a row-count alert condition
- Configure a meaningful memory threshold
- Configure alert evaluation frequency
- Associate an Azure Monitor action group
- Set an operational severity
- Enable automatic alert resolution
- Verify the alert rule was enabled
- Generate a controlled low-memory condition
- Confirm memory crossed the configured threshold
- Confirm KQL changed from no results to a matching result
- Confirm Azure Monitor fired the scheduled query alert
- Inspect the fired alert instance
- Remediate the underlying condition
- Verify Windows memory recovery
- Verify Log Analytics memory recovery
- Verify the KQL alert condition cleared
- Verify Azure Monitor automatically resolved the current alert
- Document the full alert lifecycle
- Build reusable scheduled-query KQL examples

---

# Environment

## Azure Virtual Machine

```text
Name:
AZMON-WIN01

Operating System:
Windows Server 2022 Datacenter: Azure Edition

Region:
North Central US

VM Size:
Standard_D2als_v6

vCPU:
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

## Azure Monitor Agent

Azure Monitor Agent was already installed and collecting guest performance telemetry from:

```text
AZMON-WIN01
```

---

## Alert Rule

```text
ALRT-AZMON-LOW-MEMORY-KQL
```

---

## Action Group

```text
AG-AZMON-SUPPORT
```

---

# Monitoring Architecture

The monitoring path used for INC-007 was:

```text
AZMON-WIN01
      ↓
Windows Performance Counters
      ↓
Memory\Available Bytes
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Perf Table
      ↓
KQL Scheduled Query
      ↓
Alert Evaluation
      ↓
ALRT-AZMON-LOW-MEMORY-KQL
      ↓
AG-AZMON-SUPPORT
      ↓
Azure Monitor Alert
```

---

# Why Scheduled Query Alerts Matter

Azure environments generate large volumes of telemetry.

Support engineers cannot continuously review every Log Analytics table manually.

Scheduled query rules allow administrators to convert:

```text
KQL investigation logic
```

into:

```text
continuous automated monitoring
```

This enables Azure Monitor to detect conditions without waiting for an engineer to manually run the query.

---

# Metric Alerts vs Scheduled Query Alerts

The lab previously used Azure Monitor metric alerting for high CPU.

INC-007 demonstrates a different detection model.

## Metric Alert

A metric alert evaluates Azure metrics directly.

Example:

```text
Percentage CPU > threshold
```

---

## Scheduled Query Alert

A scheduled query alert evaluates the results of a KQL query.

Example:

```text
Run KQL
      ↓
Return rows when memory is below threshold
      ↓
Alert when row count > 0
```

---

# Why KQL-Based Alerting Is Valuable

Scheduled query rules can detect conditions that require:

```text
Filtering
Aggregation
Multiple columns
Custom calculations
State classification
Historical correlation
Log-based telemetry
Security events
Application events
Performance events
Custom logs
```

This provides more flexibility than many simple metric thresholds.

---

# Memory Counter

INC-006 previously established that the collected memory counter in this environment was:

```text
ObjectName:
Memory

CounterName:
Available Bytes
```

The alert therefore reused:

```text
Memory\Available Bytes
```

instead of assuming:

```text
Memory\Available MBytes
```

---

# Healthy Memory Baseline

Before configuring the scheduled query rule, the current available-memory state was validated.

The query returned approximately:

```text
Computer:
AZMON-WIN01

AvailableMB:
2911

AvailableGB:
2.84
```

This established:

```text
Healthy operating state
```

before creating the alert.

---

# Baseline Interpretation

At approximately:

```text
2911 MB
```

available memory, the VM had substantially more memory available than the selected alert threshold.

The threshold was configured at:

```text
2048 MB
```

Therefore:

```text
2911 MB > 2048 MB
```

and the environment should not alert.

---

# Detection Query Design

The scheduled query was designed to evaluate only the newest memory sample.

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

# Query Logic

The query performs the following operations:

```text
1. Search the Perf table
2. Limit results to the last 5 minutes
3. Filter to AZMON-WIN01
4. Filter to the Memory object
5. Filter to Available Bytes
6. Select the newest sample
7. Convert bytes to megabytes
8. Keep the result only when memory is below 2048 MB
9. Return the timestamp, computer, and available memory
```

---

# Why arg_max Was Used

The scheduled query used:

```kusto
arg_max(TimeGenerated, CounterValue)
```

to select the newest relevant performance record.

This prevents older low-memory records within the query window from being treated as the current state when a newer sample exists.

---

# Healthy Query Validation

Before creating the alert, the query was executed while memory was healthy.

The result was:

```text
No results found
```

This was expected.

The query was intentionally designed to return a row only when:

```text
AvailableMB < 2048
```

Because available memory was approximately:

```text
2911 MB
```

the query correctly returned:

```text
0 rows
```

---

# Healthy Alert Logic

```text
Latest Memory:
2911 MB
      ↓
Threshold:
2048 MB
      ↓
2911 < 2048?
NO
      ↓
Query rows:
0
      ↓
Alert condition:
FALSE
```

---

# Threshold Selection

The threshold chosen for the lab was:

```text
2048 MB
```

This threshold was selected because the VM normally reported approximately:

```text
2.7–2.9 GB available
```

while a controlled 1 GB allocation had previously reduced available memory to approximately:

```text
1.7–1.8 GB
```

This provided a safe and predictable testing range.

---

# Alert Scope

The alert scope was configured for:

```text
LAW-AZMON-SUPPORT-LAB
```

Resource type:

```text
Log Analytics workspace
```

---

# Why Workspace Scope Was Appropriate

The scheduled query operates against data stored in:

```text
LAW-AZMON-SUPPORT-LAB
```

The query itself narrows the monitoring target to:

```text
AZMON-WIN01
```

using:

```kusto
| where Computer =~ "AZMON-WIN01"
```

---

# Alert Rule Condition

The rule used:

```text
Measurement:
Table rows / Count
```

The alert logic was:

```text
Operator:
Greater than

Threshold:
0
```

This means:

```text
0 rows:
Healthy

1 or more rows:
Alert
```

---

# Alert Condition Model

```text
Scheduled KQL query runs
        ↓
Does query return a row?
        ↓
No
        ↓
Table row count = 0
        ↓
Healthy
```

or:

```text
Scheduled KQL query runs
        ↓
Does query return a row?
        ↓
Yes
        ↓
Table row count > 0
        ↓
Alert fires
```

---

# Evaluation Frequency

The rule was configured to evaluate approximately every:

```text
1 minute
```

This allowed the controlled incident to be detected quickly.

---

# Query Lookback

The detection query used:

```kusto
ago(5m)
```

This allowed the scheduled query to examine recent performance telemetry while still selecting the newest sample.

---

# Action Group

The existing Azure Monitor action group used was:

```text
AG-AZMON-SUPPORT
```

This demonstrated reuse of centralized Azure Monitor notification infrastructure instead of creating duplicate action groups for each alert.

---

# Why Action Groups Matter

Action groups can be used for:

```text
Email notifications
SMS
Push notifications
Voice notifications
Webhooks
Azure Functions
Logic Apps
Automation runbooks
ITSM integrations
Event Hubs
```

They allow multiple alert rules to share common response actions.

---

# Alert Details

The scheduled query rule was named:

```text
ALRT-AZMON-LOW-MEMORY-KQL
```

Severity:

```text
2 — Warning
```

Description:

```text
Detects AZMON-WIN01 when available physical memory falls below 2048 MB using Log Analytics Perf telemetry.
```

---

# Severity Selection

Severity 2 was appropriate because the simulated condition represented:

```text
performance degradation risk
```

rather than:

```text
complete service outage
```

---

# Automatic Resolution

Automatic alert resolution was enabled.

This allowed Azure Monitor to automatically transition the alert from:

```text
Fired
```

to:

```text
Resolved
```

after the scheduled query stopped matching the low-memory condition.

---

# Alert Review

Before creation, the configuration was reviewed to confirm:

```text
Correct workspace scope
Correct KQL query
Correct row-count logic
Correct threshold
Correct evaluation frequency
Correct action group
Correct severity
Correct rule name
Automatic resolution enabled
```

---

# Alert Rule Creation

The rule was created as:

```text
ALRT-AZMON-LOW-MEMORY-KQL
```

---

# Enabled State

After deployment, the Azure Monitor alert rule list showed:

```text
ALRT-AZMON-LOW-MEMORY-KQL
```

in an enabled state.

---

# Rule Overview

The individual alert-rule overview was opened to verify that the scheduled query rule was successfully deployed and active.

This provided evidence that automated monitoring was ready before generating the incident.

---

# Incident Generation

A controlled PowerShell memory workload was used to create a low-memory condition on:

```text
AZMON-WIN01
```

The workload was designed to remain temporary and reversible.

---

# Controlled Test Safety

The VM had only:

```text
4 GiB RAM
```

The test therefore avoided intentionally exhausting all physical memory.

The goal was only to cross the alert threshold:

```text
Available memory < 2048 MB
```

---

# Controlled Workload State

The corrected controlled memory worker was successfully started and kept active long enough for Azure Monitor Agent to collect the low-memory performance state.

---

# Local Memory Validation

The low-memory test was independently checked from Windows.

The worker remained active while available memory decreased below the threshold.

This confirmed the condition existed on the guest before relying on cloud-side telemetry.

---

# Low-Memory KQL Result

The scheduled-query detection logic later returned:

```text
TimeGenerated:
2026-09-07T04:24:40.3074787Z

Computer:
AZMON-WIN01

AvailableMB:
1691
```

---

# Threshold Evaluation

```text
Available memory:
1691 MB

Threshold:
2048 MB

1691 < 2048:
TRUE
```

---

# KQL State Change

Before pressure:

```text
Query result:
0 rows
```

During pressure:

```text
Query result:
1 row
```

This confirmed the exact query used by the alert rule successfully detected the controlled incident.

---

# Detection Lifecycle

```text
Healthy Memory:
~2911 MB
      ↓
Query:
0 rows
      ↓
Controlled workload starts
      ↓
Available memory falls
      ↓
1691 MB
      ↓
1691 < 2048
      ↓
Query:
1 row
```

---

# Alert Firing

After the scheduled rule evaluated the matching query result, Azure Monitor created a fired alert instance.

The alert list showed:

```text
ALRT-AZMON-LOW-MEMORY-KQL
```

with:

```text
Fired
```

status.

---

# Fired Alert Details

The fired alert instance was opened to validate the incident details.

This confirmed the scheduled query rule had progressed from:

```text
configured
```

to:

```text
actively detecting an incident
```

---

# Low-Memory Timeline

Log Analytics later showed the following progression:

```text
04:21 UTC
2719 MB
HEALTHY

04:22 UTC
2680 MB
HEALTHY

04:23 UTC
2692 MB
HEALTHY

04:24 UTC
1691 MB
LOW MEMORY

04:25 UTC
1696 MB
LOW MEMORY

04:26 UTC
1702 MB
LOW MEMORY

04:27 UTC
1664 MB
LOW MEMORY

04:28 UTC
1673 MB
LOW MEMORY

04:29 UTC
1676 MB
LOW MEMORY

04:30 UTC
2764 MB
HEALTHY
```

---

# Sustained Condition

The memory condition remained below:

```text
2048 MB
```

for several consecutive collected samples.

This provided stable alerting evidence rather than a single transient value.

---

# Minimum Observed Memory

During the captured interval, the lowest listed value was approximately:

```text
1664 MB
```

---

# Controlled Incident Characteristics

The incident pattern was:

```text
Healthy baseline
      ↓
Sudden memory reduction
      ↓
Several consecutive low-memory samples
      ↓
Remediation
      ↓
Immediate recovery
```

This pattern was consistent with the controlled memory worker.

---

# Root Cause

The direct root cause of the low-memory condition was:

```text
Controlled PowerShell memory allocation workload
```

The workload intentionally consumed enough physical memory to reduce available memory below:

```text
2048 MB
```

---

# Root Cause Classification

```text
Category:
Performance

Subcategory:
Memory

Detection:
KQL Scheduled Query Alert

Affected Resource:
AZMON-WIN01

Cause:
Controlled test process

Result:
Available memory below configured threshold
```

---

# Remediation

The controlled worker was stopped.

Temporary workload artifacts were removed.

Windows then reported:

```text
WorkerStopped:
True

AvailableMemoryMB:
2696

AvailableMemoryGB:
2.63

Above2048MB:
True

Status:
MEMORY PRESSURE CLEARED
```

---

# Immediate Remediation Result

The operating system confirmed:

```text
2696 MB > 2048 MB
```

Therefore the guest had returned above the scheduled-query alert threshold.

---

# Why Local Recovery Was Not Enough

Stopping the worker proved the operating system had recovered.

However, the alert could not be considered fully resolved until:

```text
Azure Monitor Agent collected the recovery
Log Analytics ingested the recovery
KQL returned healthy state
Alert condition cleared
Azure Monitor resolved the alert
```

---

# Log Analytics Recovery

The next Log Analytics recovery sample showed:

```text
Time:
04:30 UTC

Computer:
AZMON-WIN01

AvailableMB:
2764

State:
HEALTHY
```

---

# Recovery Sequence

```text
04:29 UTC:
1676 MB
LOW MEMORY

04:30 UTC:
2764 MB
HEALTHY
```

This provided strong before-and-after telemetry evidence.

---

# Recovery Threshold Evaluation

```text
Available memory:
2764 MB

Threshold:
2048 MB

2764 < 2048:
FALSE
```

---

# Exact Alert Query Revalidation

After the recovery sample arrived, the exact scheduled-query condition was executed again.

The result was:

```text
No results found
```

This confirmed:

```text
Query rows:
0

Alert condition:
FALSE
```

---

# Condition Cleared Model

```text
Recovered Memory:
2764 MB
      ↓
2764 < 2048?
NO
      ↓
KQL returns:
0 rows
      ↓
Table row count:
0
      ↓
Alert condition:
CLEARED
```

---

# Post-Remediation Alert State

Immediately after remediation, Azure Monitor still showed the current alert as fired while the alerting platform completed subsequent evaluations.

This was expected.

The alert state can lag behind the underlying telemetry because:

```text
Guest state changes
      ↓
AMA collects new sample
      ↓
Data ingested
      ↓
Scheduled query evaluates
      ↓
Alert engine processes state
      ↓
Portal reflects new state
```

---

# Prior Alert Instance

The alert list also displayed an earlier resolved instance from approximately:

```text
11:08 PM local time
```

That older resolved instance was not used as proof of closure for the current controlled incident.

The current incident was correlated with the alert that fired at approximately:

```text
11:25 PM local time
```

---

# Alert Correlation

INC-007 closure evidence was intentionally tied to the same current alert lifecycle:

```text
Current alert fired
      ↓
Underlying memory remediated
      ↓
Current alert later resolved
```

rather than incorrectly using a previous resolved instance.

---

# Current Alert Resolution

After subsequent scheduled-query evaluations detected that the condition was no longer true, the current alert transitioned to:

```text
Resolved
```

---

# Resolved Alert Summary

The resolved alert instance was opened and its summary captured.

This completed the automated detection lifecycle.

---

# Full Incident Lifecycle

```text
Healthy Windows Memory
2911 MB
      ↓
KQL Detection Query
0 rows
      ↓
Alert Rule Created
ALRT-AZMON-LOW-MEMORY-KQL
      ↓
Alert Rule Enabled
      ↓
Controlled Memory Pressure
      ↓
1691 MB Available
      ↓
KQL Query
1 row
      ↓
Scheduled Query Alert
FIRED
      ↓
Alert Details Reviewed
      ↓
Controlled Worker Stopped
      ↓
Windows Recovery
2696 MB
      ↓
Log Analytics Recovery
2764 MB
      ↓
KQL Detection Query
0 rows
      ↓
Alert Condition Cleared
      ↓
Azure Monitor Alert
RESOLVED
```

---

# Healthy vs Incident vs Recovery

## Healthy Baseline

```text
Available Memory:
2911 MB

Available Memory:
2.84 GB

KQL Alert Rows:
0

State:
HEALTHY
```

---

## Incident State

```text
Available Memory:
1691 MB

Threshold:
2048 MB

KQL Alert Rows:
1

State:
LOW MEMORY

Alert:
FIRED
```

---

## Remediation State

```text
WorkerStopped:
True

AvailableMemoryMB:
2696

AvailableMemoryGB:
2.63

Above2048MB:
True
```

---

## Log Analytics Recovery

```text
Available Memory:
2764 MB

State:
HEALTHY
```

---

## Final Monitoring State

```text
KQL Alert Rows:
0

Alert Condition:
FALSE

Azure Monitor Alert:
RESOLVED
```

---

# Alert State Lifecycle

```text
Configured
      ↓
Enabled
      ↓
Healthy
      ↓
Condition True
      ↓
Fired
      ↓
Condition Cleared
      ↓
Resolved
```

---

# Scheduled Query Architecture

```text
Perf
      ↓
Memory\Available Bytes
      ↓
KQL
      ↓
Latest sample selected
      ↓
Convert bytes to MB
      ↓
AvailableMB < 2048?
      ↓
TRUE
      ↓
Return row
      ↓
Table rows > 0
      ↓
Alert fired
```

---

# Recovery Architecture

```text
Memory worker stopped
      ↓
Available memory increases
      ↓
Perf receives healthy value
      ↓
KQL checks latest sample
      ↓
AvailableMB < 2048?
      ↓
FALSE
      ↓
No rows returned
      ↓
Alert condition clears
      ↓
Alert resolves
```

---

# KQL Repository

Reusable queries for INC-007 are stored in:

```text
KQL/Log-Query-Alert-Queries.kql
```

The file contains:

```text
8 queries
```

---

# Query 01 — Latest Available Memory

Purpose:

```text
Return the latest available-memory measurement for AZMON-WIN01.
```

---

# Query 02 — Scheduled Query Alert Condition

Purpose:

```text
Return a row only when current available memory is below 2048 MB.
```

This is the core alert-detection query.

---

# Query 03 — Recent Memory State Transitions

Purpose:

```text
Classify recent performance samples as HEALTHY or LOW MEMORY.
```

---

# Query 04 — Full Incident Timeline

Purpose:

```text
Visualize available memory over time.
```

---

# Query 05 — Minimum and Maximum Available Memory

Purpose:

```text
Quantify memory change during the investigation.
```

---

# Query 06 — Current Alert Evaluation State

Purpose:

```text
Represent the current KQL alert condition as 0 or 1.
```

---

# Query 07 — Post-Remediation Recovery Verification

Purpose:

```text
Review the newest memory samples and confirm recovery.
```

---

# Query 08 — Healthy vs Low-Memory Sample Counts

Purpose:

```text
Summarize how many samples were healthy versus below the threshold.
```

---

# Evidence Directory

INC-007 screenshots are stored in:

```text
Screenshots/12-Log-Query-Alert/
```

Total screenshots:

```text
20
```

---

# Evidence 01

```text
01-AZMON-WIN01-Memory-Healthy-Baseline.png
```

Demonstrates:

```text
AZMON-WIN01 healthy memory baseline
Approximately 2911 MB available
Approximately 2.84 GB available
```

---

# Evidence 02

```text
02-Low-Memory-KQL-Healthy-No-Results.png
```

Demonstrates:

```text
Low-memory detection query
No results while memory is healthy
Alert condition false
```

---

# Evidence 03

```text
03-ALRT-AZMON-LOW-MEMORY-KQL-Scope.png
```

Demonstrates:

```text
Alert scope configuration
LAW-AZMON-SUPPORT-LAB selected
```

---

# Evidence 04

```text
04-ALRT-AZMON-LOW-MEMORY-KQL-Condition.png
```

Demonstrates:

```text
Scheduled-query condition
Table rows / Count
Greater than 0
KQL detection logic
```

---

# Evidence 05

```text
05-ALRT-AZMON-LOW-MEMORY-KQL-Action-Group.png
```

Demonstrates:

```text
AG-AZMON-SUPPORT associated with the alert
```

---

# Evidence 06

```text
06-ALRT-AZMON-LOW-MEMORY-KQL-Details.png
```

Demonstrates:

```text
Alert rule name
Severity
Description
Automatic resolution configuration
```

---

# Evidence 07A

```text
07A-ALRT-AZMON-LOW-MEMORY-KQL-Review-Create-Condition.png
```

Demonstrates:

```text
Final condition configuration before deployment
```

---

# Evidence 07B

```text
07B-ALRT-AZMON-LOW-MEMORY-KQL-Review-Create-Details-Tags.png
```

Demonstrates:

```text
Final alert details and deployment configuration
```

---

# Evidence 08

```text
08-ALRT-AZMON-LOW-MEMORY-KQL-Alert-Rule-Enabled.png
```

Demonstrates:

```text
ALRT-AZMON-LOW-MEMORY-KQL created
Rule enabled
```

---

# Evidence 09

```text
09-ALRT-AZMON-LOW-MEMORY-KQL-Overview.png
```

Demonstrates:

```text
Individual alert-rule overview
Scheduled query rule active
```

---

# Evidence 10

```text
10-AZMON-WIN01-Controlled-Low-Memory-Test-Started.png
```

Demonstrates:

```text
Controlled low-memory test
Memory worker active
Available memory reduced below threshold
```

---

# Evidence 11

```text
11-Low-Memory-KQL-Condition-Triggered.png
```

Demonstrates:

```text
AvailableMB approximately 1691
1691 MB below 2048 MB
KQL condition true
```

---

# Evidence 12

```text
12-ALRT-AZMON-LOW-MEMORY-KQL-Fired.png
```

Demonstrates:

```text
Azure Monitor scheduled query alert fired
```

---

# Evidence 13

```text
13-ALRT-AZMON-LOW-MEMORY-KQL-Fired-Details.png
```

Demonstrates:

```text
Details of the fired alert instance
Current low-memory incident detected
```

---

# Evidence 14

```text
14-AZMON-WIN01-Low-Memory-Remediation.png
```

Demonstrates:

```text
Worker stopped
2696 MB available
2.63 GB available
Above2048MB = True
Memory pressure cleared
```

---

# Evidence 15

```text
15-Low-Memory-Log-Analytics-Recovery.png
```

Demonstrates:

```text
Healthy samples before incident
Multiple LOW MEMORY samples
2764 MB HEALTHY recovery sample
```

---

# Evidence 16

```text
16-Low-Memory-KQL-Condition-Cleared.png
```

Demonstrates:

```text
Exact alert query returned no results after recovery
Alert condition false
```

---

# Evidence 17

```text
17-ALRT-AZMON-LOW-MEMORY-KQL-Post-Remediation-Alert-State.png
```

Demonstrates:

```text
Alert state immediately after remediation
Current fired alert awaiting state transition
Prior resolved instance visible separately
```

---

# Evidence 18

```text
18-ALRT-AZMON-LOW-MEMORY-KQL-Current-Alert-Resolved.png
```

Demonstrates:

```text
Current controlled low-memory alert successfully resolved
```

---

# Evidence 19

```text
19-ALRT-AZMON-LOW-MEMORY-KQL-Resolved-Summary.png
```

Demonstrates:

```text
Resolved summary for the current alert instance
```

---

# Evidence Progression

```text
01 — Healthy Memory Baseline
      ↓
02 — KQL Condition False
      ↓
03 — Alert Scope
      ↓
04 — Alert Condition
      ↓
05 — Action Group
      ↓
06 — Alert Details
      ↓
07A — Review Condition
      ↓
07B — Review Details
      ↓
08 — Alert Rule Enabled
      ↓
09 — Alert Rule Overview
      ↓
10 — Controlled Low-Memory Test
      ↓
11 — KQL Condition Triggered
      ↓
12 — Alert Fired
      ↓
13 — Fired Alert Details
      ↓
14 — Remediation
      ↓
15 — Log Analytics Recovery
      ↓
16 — KQL Condition Cleared
      ↓
17 — Post-Remediation Alert State
      ↓
18 — Current Alert Resolved
      ↓
19 — Resolved Alert Summary
```

---

# Incident Timeline

```text
Initial memory baseline:
2911 MB

Initial KQL alert condition:
FALSE

Initial query result:
0 rows

Alert rule:
ALRT-AZMON-LOW-MEMORY-KQL

Alert scope:
LAW-AZMON-SUPPORT-LAB

Action group:
AG-AZMON-SUPPORT

Severity:
2 — Warning

Evaluation frequency:
1 minute

Automatic resolution:
Enabled

Alert rule state:
Enabled

Controlled memory test:
Started

First captured low-memory KQL result:
1691 MB

Threshold:
2048 MB

Alert condition:
TRUE

Alert state:
FIRED

Controlled worker:
Stopped

Immediate Windows recovery:
2696 MB

Recovery threshold:
Above 2048 MB

Log Analytics recovery:
2764 MB

Recovery state:
HEALTHY

Exact alert query:
No results

Alert condition:
FALSE

Current Azure Monitor alert:
RESOLVED
```

---

# Root Cause Analysis

## Symptom

```text
Azure Monitor scheduled query alert reported low available memory on AZMON-WIN01.
```

---

## Detection Source

```text
Log Analytics Perf table
```

---

## Counter

```text
Memory\Available Bytes
```

---

## Detection Threshold

```text
Available memory < 2048 MB
```

---

## Detected Value

```text
1691 MB
```

---

## Root Cause

```text
Controlled PowerShell memory allocation workload
```

---

## Impact

```text
Available physical memory decreased below the configured warning threshold.
```

---

## Remediation

```text
Controlled memory worker terminated
Temporary test artifacts removed
```

---

## Immediate Recovery

```text
2696 MB available
```

---

## Log Analytics Recovery

```text
2764 MB available
HEALTHY
```

---

## Alert Recovery

```text
KQL condition false
Azure Monitor alert resolved
```

---

# Incident Correlation

The incident was correlated through multiple evidence layers.

```text
Guest Operating System
      ↓
Memory dropped

Azure Monitor Agent
      ↓
Collected performance data

Log Analytics
      ↓
1691 MB detected

Scheduled KQL Query
      ↓
Condition true

Azure Monitor Alert
      ↓
Fired

Guest Remediation
      ↓
Memory recovered

Log Analytics
      ↓
2764 MB healthy

Scheduled KQL Query
      ↓
Condition false

Azure Monitor Alert
      ↓
Resolved
```

---

# Resolution Criteria

INC-007 was not considered complete until all of the following were verified:

```text
Healthy memory baseline:
YES

Healthy KQL condition returns no rows:
YES

Scheduled query rule created:
YES

Correct workspace scope:
YES

Correct KQL condition:
YES

Correct row-count logic:
YES

Action group associated:
YES

Severity configured:
YES

Automatic resolution enabled:
YES

Alert rule enabled:
YES

Controlled incident generated:
YES

Memory below 2048 MB:
YES

KQL detection triggered:
YES

Azure Monitor alert fired:
YES

Fired details verified:
YES

Controlled workload stopped:
YES

Windows memory recovered:
YES

Log Analytics memory recovered:
YES

Exact KQL condition cleared:
YES

Current alert resolved:
YES

Resolved alert summary verified:
YES
```

---

# Production Troubleshooting Workflow

A real support engineer responding to a scheduled-query low-memory alert could follow:

```text
Alert received
      ↓
Identify affected server
      ↓
Review alert details
      ↓
Open Log Analytics
      ↓
Validate latest memory sample
      ↓
Review recent memory trend
      ↓
Identify high-memory processes
      ↓
Determine expected vs abnormal usage
      ↓
Review recent deployments or changes
      ↓
Remediate root cause
      ↓
Verify Windows recovery
      ↓
Verify Log Analytics recovery
      ↓
Verify alert condition clears
      ↓
Confirm alert resolution
      ↓
Document and close incident
```

---

# Production Root Causes for Low Memory

Possible causes include:

```text
Application memory leak
Runaway service
Database cache growth
Unexpected workload increase
Insufficient VM sizing
Backup process
Security scan
Application deployment
Large file processing
Java heap growth
.NET process growth
Memory-intensive script
Too many concurrent sessions
Misconfigured service
```

---

# Production Remediation Options

Depending on the root cause:

```text
Restart affected service
Restart application pool
Terminate runaway process
Correct application memory leak
Tune application memory settings
Reduce workload
Scale application horizontally
Resize Azure VM
Increase physical memory
Schedule workload differently
Patch affected software
Escalate to application owner
```

---

# Alert Tuning Considerations

Production scheduled query alerts should be tuned carefully.

Consider:

```text
Normal memory baseline
VM size
Expected workload variation
Query lookback period
Evaluation frequency
Threshold
Number of failing periods
Application requirements
Notification urgency
Alert severity
Automatic resolution behavior
```

---

# Avoiding Alert Noise

A threshold that is too aggressive may produce:

```text
Transient alerts
Repeated alerts
Notification fatigue
Unnecessary escalations
```

A threshold that is too relaxed may fail to detect real performance degradation.

---

# Dynamic Operational Context

A fixed:

```text
2048 MB
```

threshold was appropriate for this controlled lab because the VM had approximately:

```text
4 GiB
```

of RAM and a known healthy baseline.

Production thresholds should be selected according to the specific workload.

---

# Alert State Interpretation

Azure Monitor alert state may not change at the exact moment the underlying resource changes.

There can be delay between:

```text
OS recovery
Telemetry collection
Log ingestion
Scheduled evaluation
Alert state processing
Portal display
```

INC-007 directly demonstrated this behavior.

---

# Important Alert-Correlation Lesson

When multiple alert instances exist for the same rule, support engineers must ensure they are viewing the correct incident instance.

During INC-007, an older resolved alert was visible while the current incident was still fired.

The current incident was correctly correlated using:

```text
Current fired timestamp
Current remediation
Current recovery
Current resolved state
```

rather than mistakenly using the prior resolved record.

---

# Operational Lessons Learned

## Lesson 1 — KQL Can Become Automated Monitoring Logic

A useful troubleshooting query can be converted into:

```text
scheduled detection logic
```

This turns reactive investigation into proactive monitoring.

---

## Lesson 2 — Validate the Query Before Creating the Alert

The detection query was tested while the VM was healthy.

Expected:

```text
0 rows
```

Observed:

```text
0 rows
```

This confirmed the query would not immediately generate a false positive.

---

## Lesson 3 — Validate the Alert with a Controlled Incident

The rule was not considered proven merely because it was enabled.

A controlled low-memory condition was generated and Azure Monitor was required to actually fire.

---

## Lesson 4 — Verify the Same Query Manually

The exact alert query returned:

```text
1691 MB
```

during the incident.

This provided direct evidence that alert behavior matched the KQL logic.

---

## Lesson 5 — Automated Detection Has Multiple Layers

The alert depended on:

```text
Windows telemetry
AMA
DCR
Log Analytics
KQL
Scheduled evaluation
Azure Monitor alert engine
```

A failure at any layer could affect detection.

---

## Lesson 6 — Recovery Requires End-to-End Verification

Stopping the workload was not enough.

Recovery was confirmed through:

```text
Windows
Log Analytics
KQL
Azure Monitor alert state
```

---

## Lesson 7 — Alert Resolution Can Lag Resource Recovery

Windows recovered before Azure Monitor displayed:

```text
Resolved
```

This is normal in scheduled-query monitoring.

---

## Lesson 8 — Correlate the Correct Alert Instance

An earlier resolved alert was visible during the current investigation.

Using that older entry would have created inaccurate incident evidence.

The current fired alert was tracked until that same incident resolved.

---

# Skills Demonstrated

INC-007 demonstrates:

- Microsoft Azure
- Azure Monitor
- Azure Alerts
- Log Analytics
- Azure Monitor Agent
- Data Collection Rules
- Scheduled Query Rules
- Log Search Alerts
- Kusto Query Language
- KQL filtering
- KQL aggregation
- arg_max()
- extend
- iff()
- Perf table analysis
- Windows performance counters
- Memory monitoring
- Automated alerting
- Alert thresholds
- Table row measurement
- Evaluation frequency
- Action groups
- Alert severity
- Automatic alert resolution
- Alert lifecycle management
- Controlled fault generation
- Incident simulation
- Performance troubleshooting
- Recovery verification
- Alert correlation
- Root-cause analysis
- Azure support operations
- Incident documentation
- Evidence management

---

# Final Validation

```text
Healthy Baseline:
VERIFIED

Healthy KQL Condition:
VERIFIED

Alert Scope:
VERIFIED

Scheduled Query Condition:
VERIFIED

Action Group:
VERIFIED

Alert Details:
VERIFIED

Alert Review:
VERIFIED

Alert Rule Enabled:
VERIFIED

Alert Rule Overview:
VERIFIED

Controlled Low-Memory Test:
VERIFIED

Available Memory < 2048 MB:
VERIFIED

KQL Condition Triggered:
VERIFIED

Alert Fired:
VERIFIED

Fired Details:
VERIFIED

Remediation:
VERIFIED

Windows Recovery:
VERIFIED

Log Analytics Recovery:
VERIFIED

KQL Condition Cleared:
VERIFIED

Post-Remediation State:
VERIFIED

Current Alert Resolved:
VERIFIED

Resolved Summary:
VERIFIED

Incident Status:
CLOSED
```

---

# Incident Closure

```text
Incident ID:
INC-007

Incident:
Log Analytics Scheduled Query Alert / Low Memory

Affected Resource:
AZMON-WIN01

Workspace:
LAW-AZMON-SUPPORT-LAB

Alert Rule:
ALRT-AZMON-LOW-MEMORY-KQL

Action Group:
AG-AZMON-SUPPORT

Counter:
Memory\Available Bytes

Threshold:
2048 MB

Healthy Baseline:
2911 MB / 2.84 GB

Healthy KQL Rows:
0

Controlled Low-Memory Value:
1691 MB

Alert Condition:
TRUE

Alert State:
FIRED

Immediate Remediation Recovery:
2696 MB / 2.63 GB

Log Analytics Recovery:
2764 MB

Recovery State:
HEALTHY

Post-Recovery KQL Rows:
0

Alert Condition:
FALSE

Final Alert State:
RESOLVED

Automatic Resolution:
VERIFIED

Screenshots:
20

Reusable KQL Queries:
8

Incident Status:
CLOSED
```

INC-007 successfully demonstrates the complete lifecycle of a KQL-driven Azure Monitor scheduled query alert, including healthy baseline validation, rule configuration, controlled threshold
violation, automated alert firing, incident remediation, Log Analytics recovery validation, condition clearing, and automatic alert resolution.
