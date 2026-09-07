# 08 — Incident Investigation

## Azure Monitor + Log Analytics Support Lab

---

## Overview

This section documents the incident-investigation methodology used throughout the Azure Monitor + Log Analytics Support Lab.

The lab was designed around a practical cloud support workflow:

```text
Detect
      ↓
Validate
      ↓
Investigate
      ↓
Correlate
      ↓
Identify Root Cause
      ↓
Remediate
      ↓
Validate Recovery
      ↓
Close Incident
```

The monitored system was:

```text
AZMON-WIN01
```

with telemetry collected into:

```text
LAW-AZMON-SUPPORT-LAB
```

using:

```text
Azure Monitor Agent
DCR-AZMON-WINDOWS
Azure Monitor
Log Analytics
KQL
Azure Metrics
Azure Activity Log
Windows performance counters
Windows Event Log
```

Seven controlled support incidents were completed.

Each incident was designed to demonstrate a different monitoring or troubleshooting problem while using a consistent investigation process.

---

# Investigation Objectives

The primary objectives of the incident investigation phase were to demonstrate how a support engineer can:

- Establish a known-good baseline
- Validate whether a reported symptom is real
- Identify the affected monitoring layer
- Select the correct Azure data source
- Query recent telemetry
- Compare guest and Azure-side observations
- Determine whether the issue is resource-related or monitoring-related
- Identify the underlying cause
- Perform safe remediation
- Validate recovery
- Confirm alert resolution when applicable
- Preserve evidence
- Document the incident clearly

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

## Azure Monitor Agent

Azure Monitor Agent provided guest operating-system telemetry.

The collection path was:

```text
AZMON-WIN01
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
```

---

# Investigation Data Sources

The project used several monitoring sources because no single source answers every troubleshooting question.

---

# Azure Platform Metrics

Azure platform metrics were used for:

```text
CPU utilization
VM resource performance
Metric alert evaluation
Performance trend analysis
```

INC-001 used Azure Percentage CPU as the primary alert signal.

---

# Log Analytics Perf Table

The `Perf` table was used for guest-level performance investigation.

Examples included:

```text
Processor
Memory
LogicalDisk
Network
```

Typical fields included:

```text
TimeGenerated
Computer
ObjectName
CounterName
InstanceName
CounterValue
```

---

# Heartbeat

The `Heartbeat` table was used to determine whether Azure Monitor Agent was reporting.

This was especially important when distinguishing:

```text
Monitoring interruption
```

from:

```text
Actual VM outage
```

---

# Azure Activity Log

Azure Activity Log was used to investigate control-plane activity.

Examples included:

```text
VM deallocation
VM start operations
Administrative changes
Resource operations
Operation status
```

---

# Windows Events

Windows Application event telemetry was used during INC-004.

The controlled event included:

```text
Event ID:
200

Source:
AZMON-AppLab

Log:
Application

Level:
Error
```

---

# Azure Monitor Alerts

Alerts were used as both:

```text
Incident detection
```

and:

```text
Recovery validation
```

The lab included:

```text
Metric alerts
Scheduled query alerts
```

---

# PowerShell

PowerShell was used for:

```text
Controlled incident generation
Local validation
Process inspection
Memory validation
Storage validation
Event generation
Remediation
Recovery verification
```

---

# Investigation Methodology

A repeatable methodology was used across the seven incidents.

---

# Step 1 — Establish a Baseline

Before generating or troubleshooting a controlled incident, the normal state was established.

Examples included:

```text
Normal CPU
Recent heartbeat
Running VM
Normal disk capacity
Normal available memory
No matching alert query rows
```

This provided a reference point for determining whether a later value was abnormal.

---

# Why Baselines Matter

Without a baseline, a support engineer may see a value such as:

```text
2700 MB available memory
```

but not know whether that value is:

```text
Normal
Degraded
Critical
```

A baseline provides operational context.

---

# Baseline Sources

Depending on the incident, baselines were collected from:

```text
Azure Monitor Metrics
Log Analytics
Windows PowerShell
Heartbeat
VM resource state
Alert state
```

---

# Step 2 — Confirm the Symptom

The next step was to verify that the problem actually existed.

Examples:

```text
CPU exceeded 80%
Heartbeat stopped
VM was deallocated
Windows error event appeared
Disk free space decreased
Available memory decreased
Low-memory KQL condition returned a row
```

This prevented remediation from being performed based only on assumptions.

---

# Step 3 — Determine the Affected Layer

A central troubleshooting question was:

```text
Where is the failure occurring?
```

Possible layers included:

```text
Azure resource
Guest operating system
Azure Monitor Agent
Data Collection Rule
Log Analytics
Alert rule
Telemetry query
Application / process
```

---

# Layered Troubleshooting Model

```text
Azure Resource Layer
      ↓
Guest OS Layer
      ↓
Monitoring Agent Layer
      ↓
Collection Configuration Layer
      ↓
Log Analytics Layer
      ↓
KQL / Detection Layer
      ↓
Alert Layer
```

Each layer can fail independently.

---

# Step 4 — Select the Correct Evidence Source

Different symptoms require different evidence.

## High CPU

Use:

```text
Azure Percentage CPU
Perf
PowerShell process validation
Azure Monitor alert state
```

---

## Missing Heartbeat

Use:

```text
Heartbeat
VM resource state
AMA validation
DCR state
Recent telemetry timestamps
```

---

## VM Availability

Use:

```text
VM state
Heartbeat
Activity Log
Azure resource operations
```

---

## Windows Application Event

Use:

```text
Windows Event Log
Log Analytics event data
Event ID
Event source
Event severity
```

---

## Disk Space

Use:

```text
Windows disk state
LogicalDisk\Free Megabytes
Perf
KQL timeline
```

---

## Memory Pressure

Use:

```text
Win32_OperatingSystem
Process working set
Memory\Available Bytes
Perf
KQL timeline
```

---

## Scheduled Query Alert

Use:

```text
Perf
KQL alert condition
Scheduled query rule
Azure Monitor alert state
Action group
```

---

# Step 5 — Narrow the Time Range

Every investigation used a time range appropriate to the incident.

Examples:

```text
Last 5 minutes
Last 15 minutes
Last 30 minutes
Last 60 minutes
Last 2 hours
```

This helped isolate the controlled incident from unrelated historical telemetry.

---

# Step 6 — Validate the Resource Identity

Queries were typically filtered to:

```text
AZMON-WIN01
```

Example:

```kusto
| where Computer =~ "AZMON-WIN01"
```

This prevents unrelated systems from affecting the investigation.

---

# Step 7 — Inspect the Latest Telemetry

Recent values were often the most useful first check.

Example:

```kusto
| top 1 by TimeGenerated desc
```

or:

```kusto
| summarize arg_max(TimeGenerated, CounterValue) by Computer
```

This answered:

```text
What is the current monitored state?
```

---

# Step 8 — Review the Timeline

After identifying the latest state, the investigation expanded to historical values.

This allowed comparison of:

```text
Baseline
Incident onset
Sustained failure
Remediation
Recovery
```

---

# Timeline Model

```text
Healthy
      ↓
Change Begins
      ↓
Failure Condition
      ↓
Investigation
      ↓
Remediation
      ↓
Recovery
```

---

# Step 9 — Correlate Independent Evidence

The strongest incidents used more than one data source.

For example:

```text
Windows reports memory reduction
+
Perf reports memory reduction
+
KQL timeline shows memory reduction
```

This is stronger than relying on one source.

---

# Step 10 — Identify Root Cause

A symptom is not necessarily the root cause.

Example:

```text
Symptom:
Available memory decreased

Root Cause:
Controlled PowerShell process consuming approximately 1 GB
```

Another example:

```text
Symptom:
Heartbeat missing

Root Cause:
Monitoring interruption rather than VM failure
```

---

# Step 11 — Remediate the Cause

Remediation targeted the identified cause rather than only the visible symptom.

Examples included:

```text
Allow CPU workload to terminate
Restore monitoring
Start VM
Remove controlled event condition
Delete temporary 10 GB test file
Stop memory worker
Stop controlled low-memory workload
```

---

# Step 12 — Validate Recovery Locally

Where possible, Windows-side recovery was checked immediately.

Examples:

```text
Worker process absent
Disk capacity restored
Available memory restored
VM operational
```

---

# Step 13 — Validate Recovery in Azure

A local fix was not enough.

The investigation waited for Azure monitoring to reflect the recovered state.

Examples:

```text
New Perf sample
New Heartbeat
Healthy VM state
Resolved metric alert
Resolved scheduled query alert
```

---

# Step 14 — Close the Incident

An incident was considered complete only after:

```text
Root cause identified
Remediation completed
Recovery confirmed
Monitoring confirmed healthy
Evidence captured
Documentation updated
```

---

# Incident Investigation Matrix

| Incident | Primary Symptom | Main Evidence | Root Cause | Recovery Validation |
|---|---|---|---|---|
| INC-001 | High CPU | Azure Metrics / Alert | Controlled CPU workload | CPU recovery + resolved alert |
| INC-002 | Missing Heartbeat | Heartbeat / AMA | Monitoring interruption | Heartbeat restored |
| INC-003 | VM unavailable | VM state / Activity Log / Heartbeat | Controlled deallocation | VM online + telemetry restored |
| INC-004 | Windows error event | Event telemetry | Controlled Application event | Event investigated and incident closed |
| INC-005 | Reduced disk capacity | Windows + Perf | Controlled 10 GB file | Disk capacity restored |
| INC-006 | Reduced available memory | Windows + Perf | Controlled memory worker | Memory returned to baseline |
| INC-007 | Low-memory alert | KQL + Azure Alerts | Controlled memory workload | Query cleared + alert resolved |

---

# INC-001 — High CPU Investigation

Incident:

```text
INC-001 — High CPU Utilization
```

Affected resource:

```text
AZMON-WIN01
```

Primary detection:

```text
Azure Monitor metric alert
```

Alert:

```text
ALRT-AZMON-HIGH-CPU
```

---

# INC-001 Baseline

Before generating the condition:

```text
VM operational
Monitoring healthy
AMA reporting
DCR associated
Log Analytics receiving telemetry
CPU below threshold
```

---

# INC-001 Detection

The metric alert was configured for:

```text
Signal:
Percentage CPU

Operator:
Greater Than

Threshold:
80%

Evaluation Frequency:
1 minute

Lookback:
5 minutes
```

---

# INC-001 Controlled Workload

A controlled PowerShell workload was launched.

Two worker processes performed repeated mathematical operations.

The workload was intentionally temporary.

Maximum runtime:

```text
Approximately 10 minutes
```

---

# INC-001 Symptoms

Observed:

```text
CPU utilization increased
Percentage CPU exceeded 80%
Alert condition became true
ALRT-AZMON-HIGH-CPU fired
```

---

# INC-001 Investigation

Evidence came from:

```text
Azure Monitor Percentage CPU
Azure Monitor alert
Guest Perf telemetry
PowerShell worker validation
```

---

# INC-001 Root Cause

```text
Controlled PowerShell CPU workload
```

---

# INC-001 Recovery

The workload reached its configured timeout.

Post-test validation showed:

```text
Controlled workers:
0
```

Azure Monitor later confirmed:

```text
CPU recovered
Alert resolved
```

Final incident state:

```text
CLOSED
```

---

# INC-001 Lesson

The key lesson was the difference between:

```text
Metric detection
Root-cause identification
Remediation
Recovery
Alert resolution
```

These are separate phases and should be validated separately.

---

# INC-002 — Missing Heartbeat Investigation

Incident:

```text
INC-002 — Missing Heartbeat
```

Primary question:

```text
Is the VM offline, or is monitoring interrupted?
```

---

# Heartbeat Importance

A heartbeat indicates monitoring-agent communication.

Missing heartbeat can indicate:

```text
Agent stopped
Agent communication failure
VM shutdown
VM deallocation
Network issue
Monitoring configuration issue
```

---

# INC-002 Investigation Strategy

The incident used:

```text
Heartbeat
VM state
Azure Monitor Agent
DCR association
Recent telemetry
```

to isolate the fault.

---

# INC-002 Key Distinction

```text
Missing heartbeat
≠
Automatically VM outage
```

The resource state must be checked independently.

---

# INC-002 Root Cause

The controlled incident represented:

```text
Monitoring interruption
```

rather than:

```text
Infrastructure unavailability
```

---

# INC-002 Recovery

Recovery required:

```text
Heartbeat returned
Monitoring pipeline healthy
Recent telemetry visible
```

---

# INC-002 Lesson

Always separate:

```text
Resource availability
```

from:

```text
Monitoring availability
```

---

# INC-003 — VM Availability Investigation

Incident:

```text
INC-003 — Virtual Machine Availability
```

The VM was intentionally deallocated.

---

# INC-003 Symptom

Observed behavior included:

```text
VM unavailable
Heartbeat stopped
Monitoring telemetry interrupted
Azure control-plane operation recorded
```

---

# INC-003 Investigation Sources

```text
VM resource state
Azure Activity Log
Heartbeat
Log Analytics
```

---

# Activity Log Correlation

Activity Log provided evidence that the availability change was caused by a control-plane operation.

This helped distinguish:

```text
Unexpected crash
```

from:

```text
Administrative VM deallocation
```

---

# INC-003 Root Cause

```text
Controlled VM deallocation
```

---

# INC-003 Recovery

The VM was started again.

Recovery validation included:

```text
VM running
Heartbeat resumed
Telemetry resumed
Monitoring healthy
```

---

# INC-003 Lesson

Resource state and monitoring state should be correlated.

A missing heartbeat caused by a deallocated VM is fundamentally different from an AMA-only failure.

---

# INC-004 — Windows Application Event Investigation

Incident:

```text
INC-004 — Windows Application Event Investigation
```

---

# Controlled Event

The event was generated with:

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

# INC-004 Detection

The event was first generated on Windows.

The investigation then waited for the event to appear in the monitoring environment.

---

# INC-004 Investigation

The event was filtered using:

```text
Computer
Event ID
Source
Severity
Time
```

This confirmed the Azure monitoring pipeline received the expected guest event.

---

# INC-004 Root Cause

```text
Controlled application event generated for monitoring validation
```

---

# INC-004 Lesson

Windows event investigations should correlate:

```text
Local event
Collection pipeline
Log Analytics record
Timestamp
Event metadata
```

---

# INC-005 — Disk Space Investigation

Incident:

```text
INC-005 — Disk Space / Storage Capacity Monitoring
```

---

# INC-005 Baseline

Windows baseline:

```text
C: Total:
126.45 GB

C: Free:
112.26 GB

Free:
88.78%
```

Log Analytics also showed the healthy disk baseline through:

```text
LogicalDisk\Free Megabytes
```

---

# INC-005 Controlled Fault

A file was created:

```text
C:\AZMON-Disk-Test.bin
```

Size:

```text
10 GB
```

---

# INC-005 Symptom

Windows showed:

```text
Free:
102.26 GB

Free Percentage:
80.87%
```

The approximately 10 GB reduction was also observed in Log Analytics.

---

# INC-005 Correlation

```text
10 GB file created
      ↓
Windows free disk decreases
      ↓
LogicalDisk\Free Megabytes decreases
      ↓
KQL timeline shows reduction
```

---

# INC-005 Root Cause

```text
Controlled 10 GB test file
```

---

# INC-005 Remediation

The test file was deleted.

---

# INC-005 Recovery

Windows returned to approximately:

```text
112.26 GB free
88.78%
```

Log Analytics later showed approximately the same restored capacity.

---

# INC-005 Lesson

Capacity incidents should be validated using:

```text
Local storage state
+
Monitoring telemetry
+
Historical trend
```

---

# INC-006 — Memory Pressure Investigation

Incident:

```text
INC-006 — Memory Pressure / High Memory Usage
```

---

# INC-006 Baseline

Windows:

```text
Total:
3.99 GB

Available:
2.79 GB

Memory Used:
30.23%
```

Log Analytics:

```text
Approximately 2933 MB available
```

---

# Initial Query Problem

The original query searched for:

```text
Available MBytes
```

and returned no results.

---

# Investigation Decision

Instead of changing the Data Collection Rule immediately, the available performance counters were inventoried.

The actual counter was:

```text
Memory\Available Bytes
```

---

# INC-006 Troubleshooting Lesson

The problem was:

```text
Incorrect KQL counter name
```

not:

```text
Monitoring failure
```

---

# INC-006 Controlled Workload

A PowerShell worker allocated approximately:

```text
1024 MB
```

Observed process working set:

```text
Approximately 1105 MB
```

---

# INC-006 Symptom

During pressure:

```text
Windows Available:
1.73 GB

Memory Used:
56.59%

Log Analytics:
Approximately 1.8 GB available
```

---

# INC-006 Correlation

```text
Worker memory:
~1105 MB
      ↓
Windows available memory falls
      ↓
Memory\Available Bytes falls
      ↓
Log Analytics confirms change
```

---

# INC-006 Root Cause

```text
Controlled PowerShell memory worker
```

---

# INC-006 Remediation

The worker was stopped.

Temporary test files were removed.

---

# INC-006 Recovery

Windows:

```text
Available:
2.77 GB

Memory Used:
30.78%

WorkerPresent:
False
```

Log Analytics:

```text
Approximately 2915 MB available
```

---

# INC-006 Lesson

Before modifying monitoring configuration:

```text
Inventory the telemetry that is already being collected.
```

---

# INC-007 — Scheduled Query Alert Investigation

Incident:

```text
INC-007 — Log Analytics Scheduled Query Alert / Low Memory
```

---

# INC-007 Objective

The goal was to turn manual KQL logic into:

```text
Automated incident detection
```

---

# Alert Rule

```text
ALRT-AZMON-LOW-MEMORY-KQL
```

Action group:

```text
AG-AZMON-SUPPORT
```

---

# Alert Threshold

```text
Available memory < 2048 MB
```

---

# Healthy Baseline

Before the incident:

```text
2911 MB available
```

The scheduled query returned:

```text
0 rows
```

State:

```text
HEALTHY
```

---

# Detection Logic

The KQL query returned a row only when:

```text
AvailableMB < 2048
```

The scheduled query rule used:

```text
Table rows > 0
```

as its alert condition.

---

# INC-007 Controlled Incident

During memory pressure:

```text
AvailableMB:
1691
```

The query changed from:

```text
0 rows
```

to:

```text
1 row
```

---

# INC-007 Alert State

Azure Monitor transitioned to:

```text
FIRED
```

This proved the scheduled query rule was operational.

---

# INC-007 Remediation

The controlled low-memory process was stopped.

Windows immediately reported:

```text
AvailableMemoryMB:
2696

Above2048MB:
True

Status:
MEMORY PRESSURE CLEARED
```

---

# INC-007 Log Analytics Recovery

The recovery sample showed:

```text
AvailableMB:
2764

State:
HEALTHY
```

---

# INC-007 Query Recovery

The exact scheduled query returned:

```text
No results
```

after recovery.

Therefore:

```text
Alert condition:
FALSE
```

---

# INC-007 Alert Resolution

The current alert later transitioned from:

```text
FIRED
```

to:

```text
RESOLVED
```

---

# INC-007 Lesson

Alert resolution can lag behind resource recovery because of:

```text
Telemetry collection
Ingestion
Scheduled query evaluation
Alert state processing
Portal refresh
```

---

# Cross-Incident Investigation Patterns

Several patterns appeared repeatedly across the incidents.

---

# Pattern 1 — Establish Healthy State First

Before generating an incident:

```text
Confirm monitoring works.
```

This reduces uncertainty later.

---

# Pattern 2 — Never Assume No Data Means Failure

A query returning no results may indicate:

```text
Wrong filter
Wrong counter
Wrong time range
Wrong resource
```

rather than:

```text
Monitoring outage
```

---

# Pattern 3 — Validate Multiple Layers

Strong evidence often combines:

```text
Windows
Azure Monitor
Log Analytics
Alert state
```

---

# Pattern 4 — Use Time Correlation

The timing of events is critical.

Example:

```text
Workload started
      ↓
Telemetry changed
      ↓
Alert fired
```

This helps establish causality.

---

# Pattern 5 — Remediation Must Target the Cause

Example:

```text
High memory symptom
```

was not remediated by altering the alert threshold.

The cause was:

```text
Memory worker
```

so the worker was stopped.

---

# Pattern 6 — Verify Monitoring Recovery

A local recovery does not prove the monitoring environment has recovered.

The investigation must wait for:

```text
New Azure telemetry
```

---

# Pattern 7 — Validate the Correct Alert Instance

INC-007 showed multiple alert instances for the same rule.

The investigation correlated the correct alert using:

```text
Fired timestamp
Incident timestamp
Remediation timestamp
Resolved timestamp
```

---

# Pattern 8 — Avoid Unnecessary Configuration Changes

INC-006 could have led to an unnecessary DCR modification.

Counter inventory prevented that.

---

# Investigation Questions

A support engineer should repeatedly ask:

```text
What changed?
When did it change?
Which resource is affected?
Is the VM available?
Is the monitoring agent reporting?
Is telemetry current?
Which counter is being collected?
Does local Windows state agree with Azure?
What process or operation caused the condition?
Did remediation actually remove the cause?
Did Azure observe the recovery?
Did the alert resolve?
```

---

# Evidence Quality

Evidence should demonstrate:

```text
Before
During
After
```

A screenshot of only the failure is less useful than a sequence showing:

```text
Healthy
      ↓
Failure
      ↓
Recovery
```

---

# Screenshot Organization

Screenshots were organized by phase and incident.

Examples:

```text
Screenshots/06-Alerts/
Screenshots/07-Missing-Heartbeat/
Screenshots/08-VM-Availability/
Screenshots/09-Windows-Events/
Screenshots/10-Disk-Space/
Screenshots/11-Memory-Pressure/
Screenshots/12-Log-Query-Alert/
```

---

# Evidence Naming

Screenshot filenames describe:

```text
Resource
Condition
Monitoring state
Investigation phase
```

Examples:

```text
AZMON-WIN01-Memory-Recovery-Validated
ALRT-AZMON-LOW-MEMORY-KQL-Fired
Low-Memory-KQL-Condition-Cleared
```

This makes evidence easy to locate.

---

# Incident Documentation Structure

Each incident generally includes:

```text
Incident Summary
Environment
Objective
Baseline
Symptoms
Investigation
KQL
Root Cause
Remediation
Recovery
Evidence
Lessons Learned
Skills Demonstrated
Closure
```

---

# Why Detailed Documentation Matters

Technical investigation is only part of support work.

A strong incident record should explain:

```text
What happened
What was affected
How it was detected
What evidence was reviewed
What caused the issue
What remediation occurred
How recovery was proven
```

---

# Production Incident Triage

A production support workflow may begin with:

```text
Monitoring alert
Help desk ticket
User report
Application failure
Performance complaint
Security notification
```

The first goal is to determine:

```text
Impact
Scope
Severity
Urgency
Affected service
```

---

# Production Severity Considerations

Severity can depend on:

```text
Number of users
Business impact
Service availability
Data loss
Security risk
Performance degradation
Redundancy
Workaround availability
```

The lab used controlled incidents and warning-level scenarios for safe testing.

---

# Escalation Criteria

A support engineer may escalate when:

```text
Root cause cannot be isolated
Critical production service affected
Security incident suspected
Application owner required
Networking team required
Azure platform issue suspected
Repeated failure occurs
Remediation carries business risk
```

---

# Safe Remediation

Production remediation should consider:

```text
Business impact
Change approval
Backup state
High availability
Service dependencies
User sessions
Data integrity
Rollback plan
```

The lab used controlled workloads so remediation could be performed safely.

---

# Monitoring Delay Awareness

Support engineers should understand that Azure monitoring is not always instantaneous.

The path may include:

```text
Resource state change
      ↓
Agent collection
      ↓
Transmission
      ↓
Workspace ingestion
      ↓
Query evaluation
      ↓
Alert evaluation
      ↓
Portal update
```

This explains why an alert may remain fired briefly after the resource itself recovers.

---

# False Positive Investigation

If an alert fires but the system appears healthy:

```text
Confirm timestamp
Check latest telemetry
Check evaluation window
Check threshold
Check alert dimensions
Check query logic
Check ingestion delay
Check previous alert instance
```

---

# False Negative Investigation

If a known incident occurs but no alert fires:

```text
Confirm rule enabled
Confirm scope
Confirm telemetry exists
Run KQL manually
Check threshold
Check evaluation frequency
Check lookback
Check action group separately
Check alert processing
```

---

# Monitoring Failure vs Resource Failure

This distinction was central to the project.

## Monitoring Failure

Examples:

```text
Heartbeat missing
AMA stopped
DCR misconfiguration
Telemetry not ingested
```

---

## Resource Failure

Examples:

```text
VM deallocated
Disk capacity reduced
High CPU
Low available memory
```

---

# Alert Failure

A third category is:

```text
Telemetry healthy
Resource condition exists
Alert does not fire
```

Possible causes:

```text
Rule disabled
Wrong threshold
Wrong query
Wrong scope
Evaluation issue
```

---

# Investigation Decision Tree

```text
Alert / symptom reported
      ↓
Is resource available?
      ├── No → investigate resource state / Activity Log
      │
      └── Yes
            ↓
Is telemetry current?
      ├── No → investigate AMA / DCR / ingestion
      │
      └── Yes
            ↓
Does telemetry confirm symptom?
      ├── No → inspect alert/query logic
      │
      └── Yes
            ↓
Identify underlying process or operation
            ↓
Remediate
            ↓
Validate resource
            ↓
Validate telemetry
            ↓
Validate alert state
```

---

# Root-Cause Evidence Standard

A root-cause statement should be supported by a chain such as:

```text
Known action
      ↓
Observable state change
      ↓
Monitoring confirms change
      ↓
Action removed
      ↓
State recovers
```

This pattern was used repeatedly in the lab.

---

# Example — Disk Root Cause

```text
10 GB file created
      ↓
Disk free space decreases ~10 GB
      ↓
Perf confirms reduction
      ↓
File deleted
      ↓
Disk free space increases ~10 GB
      ↓
Perf confirms recovery
```

---

# Example — Memory Root Cause

```text
Memory worker starts
      ↓
Worker consumes ~1105 MB
      ↓
Available memory decreases
      ↓
Perf confirms reduction
      ↓
Worker stopped
      ↓
Available memory recovers
```

---

# Example — VM Availability Root Cause

```text
VM deallocation operation
      ↓
VM stops
      ↓
Heartbeat stops
      ↓
Activity Log confirms operation
      ↓
VM started
      ↓
Heartbeat returns
```

---

# Investigation Documentation Requirements

A completed support incident should answer:

```text
What was the symptom?
What was the scope?
What telemetry confirmed it?
What was the root cause?
What remediation was performed?
What evidence proves recovery?
What should be learned or prevented?
```

---

# KQL Investigation Library

Reusable investigation queries are stored in:

```text
KQL/
```

This prevents every incident from requiring query design from scratch.

---

# Benefits of Reusable Queries

Reusable queries improve:

```text
Speed
Consistency
Accuracy
Repeatability
Documentation
Team knowledge
```

---

# Production Improvements

A production implementation could expand the lab with:

```text
Multiple VMs
Application Insights
Azure Service Health
Network Watcher
Azure Workbooks
Dynamic thresholds
ITSM integration
Automation runbooks
Teams notifications
Logic Apps
Azure Functions
Azure Policy
Managed dashboards
```

---

# Operational Skills Demonstrated

This incident-investigation phase demonstrates:

- Azure Monitor
- Log Analytics
- Azure Monitor Agent
- Data Collection Rules
- Azure Metrics
- Azure Activity Log
- Azure Alerts
- Action Groups
- Scheduled Query Rules
- KQL
- Windows Server
- PowerShell
- Performance counters
- Heartbeat investigation
- Event investigation
- CPU troubleshooting
- Disk troubleshooting
- Memory troubleshooting
- VM availability troubleshooting
- Monitoring-agent troubleshooting
- Alert investigation
- Evidence correlation
- Incident triage
- Root-cause analysis
- Remediation
- Recovery validation
- Incident closure
- Technical documentation

---

# Investigation Checklist

```text
[ ] Confirm affected resource
[ ] Confirm incident timestamp
[ ] Confirm current VM state
[ ] Confirm monitoring health
[ ] Confirm Heartbeat
[ ] Confirm current telemetry
[ ] Select relevant table
[ ] Validate query filters
[ ] Establish baseline
[ ] Confirm abnormal state
[ ] Review timeline
[ ] Correlate independent evidence
[ ] Identify likely root cause
[ ] Validate root cause
[ ] Select safe remediation
[ ] Perform remediation
[ ] Validate local recovery
[ ] Validate Azure recovery
[ ] Validate alert recovery
[ ] Capture evidence
[ ] Update incident documentation
[ ] Close incident
```

---

# Completed Incident Validation

```text
INC-001 High CPU:
COMPLETE

INC-002 Missing Heartbeat:
COMPLETE

INC-003 VM Availability:
COMPLETE

INC-004 Windows Event:
COMPLETE

INC-005 Disk Space:
COMPLETE

INC-006 Memory Pressure:
COMPLETE

INC-007 Scheduled Query Alert:
COMPLETE
```

---

# Final Investigation Model

The completed lab demonstrates this end-to-end support process:

```text
Monitor
      ↓
Detect
      ↓
Triage
      ↓
Query
      ↓
Correlate
      ↓
Diagnose
      ↓
Remediate
      ↓
Validate
      ↓
Resolve
      ↓
Document
```

---

# Conclusion

Incident investigation in Azure requires more than simply opening an alert.

A complete support investigation must determine:

```text
Whether the resource is actually affected
Whether monitoring itself is healthy
Which telemetry source is authoritative
What changed
What caused the condition
How to safely remediate it
Whether recovery reached Azure monitoring
Whether alert state returned to healthy
```

The seven completed incidents demonstrate this methodology across:

```text
CPU
Monitoring heartbeat
VM availability
Windows events
Disk capacity
Memory capacity
Automated KQL alerting
```

Together, they demonstrate a repeatable cloud support troubleshooting process that moves from detection through verified incident closure.
