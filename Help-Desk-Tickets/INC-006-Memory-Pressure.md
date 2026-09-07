# INC-006 — Memory Pressure / High Memory Usage

## Azure Monitor + Log Analytics Support Lab

---

## Incident Summary

**Incident ID:** INC-006
**Title:** Memory Pressure / High Memory Usage
**Affected Resource:** AZMON-WIN01
**Resource Group:** RG-AZMON-SUPPORT-LAB
**Monitoring Workspace:** LAW-AZMON-SUPPORT-LAB
**Platform:** Microsoft Azure
**Operating System:** Windows Server 2022 Datacenter: Azure Edition
**Severity:** Medium / Performance Degradation
**Incident Type:** Memory Capacity / Performance Monitoring
**Status:** Resolved
**Controlled Workload:** PowerShell memory allocation process
**Target Allocation:** 1024 MB
**Observed Worker Memory:** Approximately 1105 MB
**Root Cause:** Controlled PowerShell process intentionally consuming approximately 1 GB of physical memory
**Resolution:** Controlled memory worker stopped and memory availability returned to baseline

---

# Executive Summary

INC-006 simulated a controlled memory-pressure condition on the Azure virtual machine:

```text
AZMON-WIN01
```

The objective was to demonstrate how memory availability can be monitored, investigated, correlated, remediated, and validated using:

```text
Windows Server
PowerShell
Win32_OperatingSystem
Windows performance counters
Azure Monitor Agent
Data Collection Rules
Log Analytics
Perf table
Kusto Query Language
Time-series visualization
```

Before the incident was generated, available memory was measured from both Windows and Log Analytics.

The Windows baseline showed:

```text
Total Memory:
3.99 GB

Available Memory:
2.79 GB

Used Memory:
1.21 GB

Memory Used:
30.23%
```

Log Analytics reported approximately:

```text
2933 MB
2.86 GB
```

available through the collected Windows performance counter:

```text
Object:
Memory

Counter:
Available Bytes
```

An initial query attempted to use:

```text
Available MBytes
```

but returned no results.

A counter inventory query was used to identify the actual memory counter being collected by the Data Collection Rule.

The investigation confirmed:

```text
ObjectName:
Memory

CounterName:
Available Bytes
```

with more than 100 recent samples.

The KQL query was corrected to use:

```text
Available Bytes
```

and convert the value to MB and GB.

A controlled PowerShell worker was then started to allocate approximately:

```text
1024 MB
```

of memory.

The memory workload produced the expected change.

Windows showed:

```text
Available Memory:
1.73 GB

Memory Used:
56.59%

Worker Running:
True

Worker Memory:
1105 MB
```

Log Analytics later showed approximately:

```text
1809–1827 MB
```

available memory during the pressure condition.

The time-series chart clearly displayed the reduction from approximately:

```text
2.9 GB available
```

to:

```text
1.8 GB available
```

The controlled workload was then stopped.

Windows immediately recovered to approximately:

```text
2.77 GB available

30.78% memory used
```

and confirmed:

```text
WorkerPresent:
False
```

Log Analytics subsequently reported approximately:

```text
2915 MB
2.85 GB
```

available.

The final KQL timechart displayed the complete incident lifecycle:

```text
Healthy Memory Baseline
        ↓
Controlled 1 GB Allocation
        ↓
Available Memory Drops
        ↓
Memory Pressure Maintained
        ↓
Worker Process Stopped
        ↓
Available Memory Recovers
```

INC-006 demonstrates memory-performance monitoring, counter discovery, controlled fault generation, guest-level validation, Log Analytics investigation, telemetry-latency awareness, remediation,
and end-to-end recovery verification.

---

# Incident Objectives

The objectives of INC-006 were to:

- Establish a Windows memory baseline
- Establish a Log Analytics memory baseline
- Verify Azure Monitor memory telemetry
- Identify the exact collected memory counter
- Troubleshoot an initial KQL no-results condition
- Validate Data Collection Rule performance-counter coverage
- Safely generate controlled memory pressure
- Avoid destabilizing the 4 GB virtual machine
- Allocate approximately 1 GB of memory
- Confirm the workload process remained active
- Measure workload memory consumption
- Confirm available memory decreased in Windows
- Confirm memory utilization increased
- Confirm the decrease reached Log Analytics
- Compare collection and ingestion timestamps
- Visualize available-memory reduction
- Stop the controlled memory workload
- Verify the worker process was removed
- Verify Windows memory recovery
- Verify Log Analytics memory recovery
- Visualize the full incident lifecycle
- Build reusable KQL memory queries
- Document the troubleshooting workflow

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

## Azure Monitor Agent

The VM was already onboarded to Azure Monitor Agent and sending guest telemetry to:

```text
LAW-AZMON-SUPPORT-LAB
```

---

# Monitoring Architecture

The memory-monitoring path used in INC-006 was:

```text
AZMON-WIN01
      ↓
Windows Memory Performance Counter
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Perf Table
      ↓
KQL
      ↓
Memory Investigation
```

Windows-side validation used:

```text
Win32_OperatingSystem
PowerShell
Process Working Set
```

This allowed the memory condition to be verified independently at the operating-system and monitoring layers.

---

# Why Memory Monitoring Matters

Memory pressure can cause significant performance degradation.

Common symptoms include:

```text
Slow applications
Increased response times
Paging
High disk activity
Application hangs
Service instability
Process termination
Failed deployments
Poor user experience
Operating system responsiveness issues
```

Monitoring memory availability helps administrators identify resource exhaustion before a server becomes unstable.

---

# Memory Monitoring Layers

INC-006 used multiple monitoring layers.

## Windows Operating System

```text
Win32_OperatingSystem
```

provided direct information about:

```text
Total physical memory
Available physical memory
Used memory
Memory utilization percentage
```

---

## Windows Process Layer

The controlled worker process was inspected for:

```text
Process ID
Running state
Working set
```

This provided direct evidence that the memory allocation was caused by the intended lab workload.

---

## Windows Performance Counter

The actual counter collected by the environment was:

```text
\Memory\Available Bytes
```

---

## Azure Monitor Agent

Azure Monitor Agent collected the configured memory performance telemetry.

---

## Data Collection Rule

```text
DCR-AZMON-WINDOWS
```

controlled which guest performance counters were collected.

---

## Log Analytics

Memory telemetry was stored in:

```text
Perf
```

---

## KQL

Kusto Query Language was used to:

```text
Identify collected counters
Find latest available memory
Review memory samples
Compare collection and ingestion timestamps
Calculate minimum memory availability
Calculate maximum memory availability
Measure memory reduction
Detect low available memory
Verify recovery
Visualize the incident
```

---

# Initial Log Analytics Query

The first memory query attempted to use:

```text
ObjectName:
Memory

CounterName:
Available MBytes
```

The query returned:

```text
No results found from the specified time range
```

This required investigation before the controlled workload could be started.

---

# Initial Troubleshooting Decision

The absence of results could have indicated:

```text
Memory counter not configured
Incorrect counter name
Data Collection Rule issue
Azure Monitor Agent issue
Telemetry delay
KQL filter mismatch
```

Rather than immediately modifying the DCR, the existing performance-counter inventory was queried.

---

# Performance Counter Inventory

The following type of query was used:

```kusto
Perf
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| summarize SampleCount=count(),
            LastSample=max(TimeGenerated)
    by ObjectName, CounterName, InstanceName
| order by ObjectName asc, CounterName asc
```

The results showed several actively collected counters, including:

```text
LogicalDisk
Avg. Disk Queue Length

LogicalDisk
Free Megabytes

Memory
Available Bytes

Network Interface
Bytes Total/sec

Processor Information
% Processor Time
```

---

# Memory Counter Discovery

The inventory confirmed:

```text
ObjectName:
Memory

CounterName:
Available Bytes
```

The environment already had recent memory samples.

Therefore:

```text
DCR modification:
NOT REQUIRED
```

---

# Counter Name Root Cause

The original query failed because it searched for:

```text
Available MBytes
```

while the actual collected counter was:

```text
Available Bytes
```

This was a query-definition issue rather than a monitoring failure.

---

# Corrected Memory Query

The query was corrected to:

```kusto
Perf
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| top 1 by TimeGenerated desc
| project TimeGenerated,
          Computer,
          AvailableMB=round(CounterValue / 1024.0 / 1024.0, 0),
          AvailableGB=round(CounterValue / 1024.0 / 1024.0 / 1024.0, 2)
```

---

# Log Analytics Memory Baseline

The corrected query returned approximately:

```text
Computer:
AZMON-WIN01

AvailableMB:
2933

AvailableGB:
2.86
```

This confirmed:

```text
Memory telemetry:
AVAILABLE

Azure Monitor Agent:
REPORTING

DCR:
COLLECTING MEMORY

Perf table:
RECEIVING MEMORY DATA
```

---

# Windows Memory Baseline

The Windows baseline was captured using:

```powershell
Get-CimInstance Win32_OperatingSystem
```

Observed:

```text
TotalMemoryGB:
3.99

AvailableMemoryGB:
2.79

UsedMemoryGB:
1.21

MemoryUsedPercent:
30.23
```

---

# Baseline Interpretation

The VM had approximately:

```text
4 GB total memory
```

with roughly:

```text
2.8–2.9 GB available
```

before the test.

The server was not experiencing memory pressure.

---

# Baseline Cross-Validation

Windows reported:

```text
2.79 GB available
```

Log Analytics reported:

```text
2.86 GB available
```

The small difference was expected because the values were collected at different times.

The two sources were sufficiently close to establish a known-good baseline.

---

# Safe Incident Design

The VM had only:

```text
4 GiB RAM
```

so the memory workload needed to remain controlled.

The baseline available memory was approximately:

```text
2.8 GB
```

A target workload of:

```text
1 GB
```

was selected.

This was large enough to produce a visible monitoring change while still leaving substantial memory available to Windows.

---

# Safety Criteria

The controlled test followed these requirements:

```text
Do not exhaust physical memory
Do not intentionally trigger severe paging
Do not destabilize Windows
Do not allocate all available memory
Use a temporary worker process
Use an automatic timeout
Record the worker PID
Allow immediate manual termination
Remove temporary scripts after testing
Verify recovery after workload termination
```

---

# Controlled Memory Workload

The test created a PowerShell worker that allocated memory in blocks.

Target:

```text
1024 MB
```

The worker was configured to remain active long enough for Azure Monitor to collect the reduced available-memory condition.

---

# Worker Process

The controlled test produced:

```text
Worker PID:
2572
```

The worker was recorded so it could be safely identified and stopped during remediation.

---

# Memory Pressure Start

After the controlled workload started, Windows reported:

```text
Memory workload:
STARTED

Target allocation:
1024 MB

Worker PID:
2572

Available memory:
1.73 GB

Status:
CONTROLLED MEMORY PRESSURE ACTIVE
```

---

# Immediate Memory Change

Available memory changed from approximately:

```text
2.79 GB
```

to:

```text
1.73 GB
```

This represented a reduction of approximately:

```text
1.06 GB
```

which closely matched the controlled allocation target.

---

# Worker Validation

The memory worker was independently validated.

Observed:

```text
WorkerRunning:
True

WorkerPID:
2572

WorkerMemoryMB:
1105

AvailableMemoryGB:
1.73

MemoryUsedPercent:
56.59
```

---

# Worker Interpretation

The process working set of approximately:

```text
1105 MB
```

closely matched the intended:

```text
1024 MB
```

memory-pressure workload.

The difference can result from:

```text
PowerShell process overhead
Runtime allocations
Object overhead
Memory-management behavior
```

---

# Memory Utilization Change

Baseline memory utilization:

```text
30.23%
```

During pressure:

```text
56.59%
```

Change:

```text
approximately 26 percentage points
```

This confirmed the workload had a meaningful but controlled effect.

---

# Windows Pressure State

During the incident:

```text
Total Memory:
3.99 GB

Available Memory:
1.73 GB

Memory Used:
56.59%

Worker:
Running

Worker Working Set:
~1105 MB
```

The VM remained stable.

---

# Log Analytics Pressure Verification

After the workload remained active, newer `Perf` records appeared.

Observed values included:

```text
AvailableMB:
1809

AvailableGB:
1.77
```

and:

```text
AvailableMB:
1827

AvailableGB:
1.78
```

---

# Monitoring Comparison

Before the workload:

```text
Available memory:
~2929–2937 MB
```

During the workload:

```text
Available memory:
~1809–1827 MB
```

Approximate monitored reduction:

```text
~1100 MB
```

This closely matched the controlled worker memory usage.

---

# End-to-End Memory Monitoring Validation

The complete monitoring path was now verified:

```text
1 GB worker started
      ↓
Windows available memory decreases
      ↓
Memory\Available Bytes decreases
      ↓
Azure Monitor Agent collects value
      ↓
DCR processes telemetry
      ↓
Log Analytics ingests sample
      ↓
Perf table reports ~1.8 GB available
```

---

# Available Memory Timeline

The following query was used:

```kusto
Perf
| where TimeGenerated > ago(45m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| summarize AvailableMB=avg(CounterValue) / 1024.0 / 1024.0
    by bin(TimeGenerated, 1m)
| order by TimeGenerated asc
| render timechart
```

---

# Pressure Timeline Result

The chart showed a sharp drop from approximately:

```text
~2900 MB
```

to approximately:

```text
~1800 MB
```

at the time of the controlled workload.

---

# Timeline Interpretation

The sharp reduction corresponded with the controlled PowerShell memory worker.

The timeline provided immediate visual evidence of:

```text
Healthy memory state
      ↓
Controlled pressure
```

---

# Why Time-Series Visualization Matters

Memory tables show exact values.

Charts help identify:

```text
Sudden allocation spikes
Gradual memory leaks
Repeated workload patterns
Sustained memory pressure
Recovery after process termination
```

---

# Incident Condition

At the controlled pressure state:

```text
Windows Available:
1.73 GB

Windows Memory Used:
56.59%

Worker Memory:
1105 MB

Log Analytics Available:
~1.8 GB

Worker Running:
True
```

---

# Important Lab Distinction

INC-006 demonstrates:

```text
controlled memory pressure
```

not:

```text
critical memory exhaustion
```

The VM was intentionally kept within a safe operating range.

---

# Root Cause

The direct root cause was:

```text
Controlled PowerShell memory worker
```

with:

```text
PID:
2572

Working Set:
approximately 1105 MB
```

---

# Root Cause Classification

```text
Category:
Performance / Memory

Affected Resource:
AZMON-WIN01

Cause Type:
Controlled process memory allocation

Target:
1024 MB

Observed Worker Memory:
~1105 MB

Result:
Reduced available physical memory
```

---

# Remediation Plan

The remediation action was:

```text
Stop PID 2572
```

The temporary script and PID-tracking file were also removed.

---

# Memory Pressure Remediation

PowerShell checked whether the worker process was still running.

If present:

```powershell
Stop-Process -Id 2572 -Force
```

The temporary test artifacts were then removed.

---

# Remediation Result

Observed:

```text
Memory worker stopped:
YES

Available memory:
2.77 GB

Memory used:
30.7%

Status:
MEMORY PRESSURE CLEARED
```

---

# Immediate Windows Recovery

Available memory recovered from:

```text
1.73 GB
```

to:

```text
2.77 GB
```

Memory utilization returned from:

```text
56.59%
```

to approximately:

```text
30.7%
```

---

# Recovery Validation

A separate Windows validation was performed.

Observed:

```text
TotalMemoryGB:
3.99

AvailableMemoryGB:
2.77

UsedMemoryGB:
1.23

MemoryUsedPercent:
30.78

WorkerPresent:
False
```

---

# Recovery Interpretation

All guest-level indicators confirmed successful remediation.

```text
Worker:
STOPPED

Available Memory:
RECOVERED

Memory Utilization:
RECOVERED

Worker Process:
ABSENT
```

---

# Log Analytics Recovery

A newer Log Analytics sample later appeared with:

```text
AvailableMB:
2915

AvailableGB:
2.85
```

The preceding records still showed:

```text
1809
1827
1832
1833 MB
```

during the controlled workload.

---

# Recovery Telemetry Path

The recovery followed:

```text
Worker terminated
      ↓
Allocated memory released
      ↓
Windows available memory increases
      ↓
Memory\Available Bytes increases
      ↓
AMA collects recovered value
      ↓
DCR processes telemetry
      ↓
Log Analytics ingests recovered sample
      ↓
Perf reports ~2915 MB
```

---

# Final Memory Timeline

The final timechart displayed:

```text
Healthy Baseline
~2.9 GB available
      ↓
Controlled Worker Starts
      ↓
Available Memory Drops
~1.8 GB
      ↓
Controlled Pressure Maintained
      ↓
Worker Stopped
      ↓
Available Memory Recovers
~2.9 GB
```

---

# Baseline vs Pressure vs Recovery

## Baseline

```text
Windows Available:
2.79 GB

Windows Used:
1.21 GB

Windows Memory Used:
30.23%

Log Analytics Available:
~2933 MB
```

---

## Pressure

```text
Windows Available:
1.73 GB

Windows Memory Used:
56.59%

Worker Memory:
~1105 MB

Log Analytics Available:
~1809–1827 MB
```

---

## Recovery

```text
Windows Available:
2.77 GB

Windows Memory Used:
30.78%

WorkerPresent:
False

Log Analytics Available:
2915 MB
```

---

# Memory Delta

Approximate monitored baseline:

```text
2933 MB
```

Approximate controlled pressure:

```text
1809 MB
```

Difference:

```text
2933 - 1809
=
1124 MB
```

This aligns closely with the observed worker working set:

```text
1105 MB
```

---

# Incident Impact

The controlled test did not cause:

```text
VM outage
Heartbeat outage
Azure Monitor Agent failure
Application failure
System instability
Critical paging condition
```

The purpose was controlled memory telemetry validation.

---

# Production Memory Pressure Impact

In a production environment, sustained memory pressure can affect:

```text
Application performance
Database performance
Web servers
Remote desktop sessions
Backup processes
Security software
Monitoring agents
Windows services
Virtual-machine responsiveness
```

---

# Production Investigation Workflow

A practical memory-pressure workflow is:

```text
Alert or Performance Complaint
      ↓
Identify Affected Server
      ↓
Check Available Memory
      ↓
Check Memory Utilization
      ↓
Review Historical Perf Data
      ↓
Identify High-Memory Processes
      ↓
Check for Memory Leak Patterns
      ↓
Review Recent Changes
      ↓
Determine Root Cause
      ↓
Stop / Restart / Tune Application
      ↓
Increase RAM if Required
      ↓
Validate Recovery
      ↓
Close Incident
```

---

# Production Investigation Questions

Useful questions include:

```text
How much memory is installed?
How much memory is currently available?
Which process is consuming the most RAM?
Did memory usage increase suddenly?
Is usage gradually increasing?
Is paging increasing?
Was software recently deployed?
Did a service restart?
Is the workload expected?
Is the application leaking memory?
Is the VM undersized?
Did the condition recover automatically?
```

---

# High-Memory Process Investigation

PowerShell can identify large memory consumers.

Example:

```powershell
Get-Process |
    Sort-Object WorkingSet64 -Descending |
    Select-Object -First 10 Name,
        Id,
        @{Name="WorkingSetMB";Expression={[math]::Round($_.WorkingSet64 / 1MB,0)}}
```

---

# Additional Windows Memory Counters

Useful production counters can include:

```text
Memory\Available Bytes
Memory\Available MBytes
Memory\% Committed Bytes In Use
Memory\Committed Bytes
Memory\Pages/sec
Paging File\% Usage
Process\Working Set
Process\Private Bytes
```

Availability depends on the Data Collection Rule configuration.

---

# Counter Used in INC-006

The actual collected memory counter was:

```text
ObjectName:
Memory

CounterName:
Available Bytes
```

---

# Bytes Conversion

Because the counter returned bytes, KQL converted the value to megabytes using:

```text
CounterValue / 1024 / 1024
```

and gigabytes using:

```text
CounterValue / 1024 / 1024 / 1024
```

---

# Available Bytes vs Available MBytes

A key lesson from INC-006 was that similar Windows performance counters may use different names.

The initial assumption:

```text
Available MBytes
```

was not correct for the current DCR.

The actual collected counter was:

```text
Available Bytes
```

Always inspect collected data before assuming a counter name.

---

# Monitoring Troubleshooting Model

When a performance query returns no data:

```text
Check query filters
      ↓
Inventory current Perf counters
      ↓
Confirm ObjectName
      ↓
Confirm CounterName
      ↓
Confirm InstanceName
      ↓
Check recent TimeGenerated
      ↓
Confirm AMA Heartbeat
      ↓
Check DCR only if counter is actually missing
```

---

# Why Counter Inventory Was Important

Without querying the available counters, the no-results condition could have led to an unnecessary DCR modification.

The inventory proved:

```text
Memory telemetry already existed
```

The problem was only:

```text
wrong KQL counter name
```

---

# Collection and Ingestion Timing

INC-006 also used:

```text
TimeGenerated
```

and:

```text
ingestion_time()
```

to compare when the performance sample was created with when Log Analytics ingested it.

---

# Why Telemetry Timing Matters

Immediately after a resource change, the latest cloud record may still represent the previous state.

This does not automatically indicate:

```text
Agent failure
DCR failure
Workspace failure
```

A newer sample may simply not have reached the workspace yet.

---

# Cross-Layer Validation

INC-006 validated the memory incident through:

```text
Win32_OperatingSystem
+
PowerShell process state
+
Process working set
+
Windows Memory performance counter
+
Azure Monitor Agent
+
Log Analytics Perf
+
KQL timechart
```

This provides stronger evidence than relying on a single monitoring source.

---

# Alerting Considerations

A production memory alert could be based on:

```text
Low Available Memory
```

rather than simply:

```text
High Memory Usage
```

because `Available Bytes` was the collected counter in this environment.

---

# Example Low-Memory Threshold

The reusable KQL file contains an example condition such as:

```text
AvailableMB < 1024
```

This would identify servers with less than approximately:

```text
1 GB
```

available memory.

---

# Threshold Caution

The correct production threshold depends on:

```text
VM size
Application workload
Expected cache behavior
Paging configuration
Database requirements
Operating system requirements
Historical usage
```

One fixed threshold is not appropriate for every server.

---

# Alert Evaluation Design

A production alert might use:

```text
Condition:
Available memory below threshold

Evaluation:
Every 5 minutes

Persistence:
Condition remains low for 10–15 minutes
```

This can reduce noise from short-term allocation spikes.

---

# Memory Leak Detection

A true memory leak typically appears as:

```text
Available memory gradually decreasing
```

while a process working set continues to increase.

A timechart is especially useful for identifying this pattern.

---

# Controlled Incident Pattern

INC-006 produced a different pattern:

```text
Healthy baseline
      ↓
Immediate allocation
      ↓
Stable pressure plateau
      ↓
Immediate recovery
```

This matched a known temporary workload rather than a leak.

---

# Remediation Options in Production

Possible production responses include:

```text
Stop runaway process
Restart affected service
Restart application pool
Restart application
Correct memory leak
Tune application configuration
Reduce workload
Scale out service
Resize Azure VM
Add RAM
Review page file configuration
Apply software update
Escalate to application owner
```

---

# Why Process Termination Requires Care

In a production environment, support engineers should not terminate an unfamiliar high-memory process without understanding:

```text
Application ownership
Business impact
Data-loss risk
Service dependencies
High-availability design
Maintenance requirements
```

INC-006 used a known controlled worker, so termination was safe.

---

# KQL Repository

Reusable queries are stored in:

```text
KQL/Memory-Pressure-Queries.kql
```

The file contains:

```text
8 reusable memory queries
```

---

# Query 01 — Latest Available Memory

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

Purpose:

```text
Determine the newest available-memory value.
```

---

# Query 02 — Recent Memory Samples

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| project TimeGenerated,
          Computer,
          AvailableMB=round(CounterValue / 1024.0 / 1024.0,0),
          AvailableGB=round(CounterValue / 1024.0 / 1024.0 / 1024.0,2)
| order by TimeGenerated desc
```

Purpose:

```text
Review available-memory changes over time.
```

---

# Query 03 — Collection vs Ingestion

```kusto
Perf
| where TimeGenerated > ago(45m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| project TimeGenerated,
          IngestedAt=ingestion_time(),
          AvailableMB=round(CounterValue / 1024.0 / 1024.0,0),
          AvailableGB=round(CounterValue / 1024.0 / 1024.0 / 1024.0,2)
| order by TimeGenerated desc
```

Purpose:

```text
Troubleshoot memory telemetry collection and ingestion timing.
```

---

# Query 04 — Available Memory Timeline

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

Purpose:

```text
Visualize baseline, memory pressure, and recovery.
```

---

# Query 05 — Minimum and Maximum Available Memory

```kusto
Perf
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| summarize MinimumBytes=min(CounterValue),
            MaximumBytes=max(CounterValue)
            by Computer
| extend MinimumMB=MinimumBytes / 1024.0 / 1024.0,
         MaximumMB=MaximumBytes / 1024.0 / 1024.0,
         ChangeMB=(MaximumBytes-MinimumBytes) / 1024.0 / 1024.0
| project Computer,
          MinimumMB=round(MinimumMB,0),
          MaximumMB=round(MaximumMB,0),
          ChangeMB=round(ChangeMB,0)
```

Purpose:

```text
Quantify the largest available-memory change.
```

---

# Query 06 — Detect Low Available Memory

```kusto
Perf
| where TimeGenerated > ago(15m)
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| summarize arg_max(TimeGenerated,CounterValue) by Computer
| extend AvailableMB=CounterValue / 1024.0 / 1024.0
| where AvailableMB < 1024
| project TimeGenerated,
          Computer,
          AvailableMB=round(AvailableMB,0)
| order by AvailableMB asc
```

Purpose:

```text
Identify monitored computers with less than approximately 1 GB available.
```

---

# Query 07 — Recovery Verification

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| top 10 by TimeGenerated desc
| project TimeGenerated,
          IngestedAt=ingestion_time(),
          AvailableMB=round(CounterValue / 1024.0 / 1024.0,0),
          AvailableGB=round(CounterValue / 1024.0 / 1024.0 / 1024.0,2)
```

Purpose:

```text
Verify the newest samples returned to the expected memory baseline.
```

---

# Query 08 — Quantify Memory Pressure

```kusto
Perf
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "Memory"
| where CounterName =~ "Available Bytes"
| summarize BaselineMaxBytes=max(CounterValue),
            PressureMinBytes=min(CounterValue)
            by Computer
| extend BaselineMaxMB=BaselineMaxBytes / 1024.0 / 1024.0,
         PressureMinMB=PressureMinBytes / 1024.0 / 1024.0,
         ReductionMB=(BaselineMaxBytes-PressureMinBytes) / 1024.0 / 1024.0
| project Computer,
          BaselineMaxMB=round(BaselineMaxMB,0),
          PressureMinMB=round(PressureMinMB,0),
          ReductionMB=round(ReductionMB,0)
```

Purpose:

```text
Measure the observed reduction in available memory during the incident.
```

---

# Evidence Directory

INC-006 evidence is stored in:

```text
Screenshots/11-Memory-Pressure/
```

---

# Evidence 01

```text
01-AZMON-WIN01-Available-Memory-Baseline.png
```

Demonstrates:

```text
Correct Memory counter located
Available Bytes converted to MB and GB
Baseline available memory approximately 2933 MB / 2.86 GB
```

---

# Evidence 02

```text
02-AZMON-WIN01-Windows-Memory-Baseline.png
```

Demonstrates:

```text
Total memory 3.99 GB
Available memory 2.79 GB
Used memory 1.21 GB
Memory utilization 30.23%
```

---

# Evidence 03

```text
03-AZMON-WIN01-Controlled-Memory-Pressure-Started.png
```

Demonstrates:

```text
Controlled memory workload started
Target allocation 1024 MB
Worker PID 2572
Available memory reduced to 1.73 GB
```

---

# Evidence 04

```text
04-AZMON-WIN01-Memory-Pressure-Validated.png
```

Demonstrates:

```text
Worker running = True
Worker PID 2572
Worker memory approximately 1105 MB
Available memory 1.73 GB
Memory used 56.59%
```

---

# Evidence 05

```text
05-AZMON-WIN01-Memory-Pressure-Log-Analytics-Verified.png
```

Demonstrates:

```text
Reduced available memory reached Log Analytics
Available memory approximately 1809–1827 MB
Baseline records approximately 2929–2937 MB remain visible
```

---

# Evidence 06

```text
06-AZMON-WIN01-Available-Memory-Drop-Timeline.png
```

Demonstrates:

```text
Clear available-memory decline
Baseline approximately 2.9 GB
Pressure approximately 1.8 GB
```

---

# Evidence 07

```text
07-AZMON-WIN01-Memory-Pressure-Cleared.png
```

Demonstrates:

```text
Memory worker stopped successfully
Available memory recovered to approximately 2.77 GB
Memory utilization returned to approximately 30.7%
```

---

# Evidence 08

```text
08-AZMON-WIN01-Memory-Recovery-Validated.png
```

Demonstrates:

```text
Total memory 3.99 GB
Available memory 2.77 GB
Used memory 1.23 GB
Memory utilization 30.78%
WorkerPresent = False
```

---

# Evidence 09

```text
09-AZMON-WIN01-Memory-Recovery-Log-Analytics-Verified.png
```

Demonstrates:

```text
Newest Log Analytics sample approximately 2915 MB / 2.85 GB
Earlier pressure-state samples approximately 1.8 GB
Monitoring recovery verified
```

---

# Evidence 10

```text
10-AZMON-WIN01-Memory-Pressure-Full-Incident-Timeline.png
```

Demonstrates the complete lifecycle:

```text
Healthy
↓
Controlled Memory Allocation
↓
Reduced Available Memory
↓
Remediation
↓
Recovered Available Memory
```

---

# Evidence Progression

```text
01 — Log Analytics Memory Baseline
      ↓
02 — Windows Memory Baseline
      ↓
03 — Controlled Memory Pressure Started
      ↓
04 — Worker and Memory Pressure Validated
      ↓
05 — Reduced Memory Reaches Log Analytics
      ↓
06 — Available Memory Drop Visualized
      ↓
07 — Memory Worker Stopped
      ↓
08 — Windows Memory Recovery Validated
      ↓
09 — Log Analytics Recovery Verified
      ↓
10 — Complete Incident Timeline
```

---

# Incident Timeline

```text
Initial State:
AZMON-WIN01 healthy

Windows Available Memory:
2.79 GB

Windows Memory Used:
30.23%

Log Analytics Available:
~2933 MB

Initial KQL Query:
No results for Available MBytes

Troubleshooting:
Perf counter inventory performed

Finding:
Memory\Available Bytes is collected

DCR Change:
Not required

Corrected KQL:
Available Bytes converted to MB / GB

Controlled Incident:
1 GB memory worker started

Worker PID:
2572

Worker Memory:
~1105 MB

Windows Available Memory:
1.73 GB

Windows Memory Used:
56.59%

Log Analytics Pressure:
~1809–1827 MB

Memory Timeline:
Drop verified

Remediation:
Worker PID 2572 stopped

Temporary Test Files:
Removed

Windows Available Memory:
2.77 GB

Windows Memory Used:
30.78%

WorkerPresent:
False

Log Analytics Recovery:
2915 MB / 2.85 GB

Final Timeline:
Healthy → Pressure → Recovered

Incident:
RESOLVED
```

---

# Incident Correlation

The evidence chain was:

```text
Known-Good Memory Baseline
      ↓
Counter Inventory
      ↓
Correct Memory Counter Identified
      ↓
Controlled Worker Started
      ↓
Windows Available Memory Decreased
      ↓
Worker Working Set Confirmed
      ↓
Log Analytics Memory Decreased
      ↓
Timechart Confirmed Pressure
      ↓
Worker Stopped
      ↓
Windows Memory Recovered
      ↓
Worker Absence Confirmed
      ↓
Log Analytics Memory Recovered
      ↓
Timechart Confirmed Recovery
```

---

# Root Cause Analysis

## Symptom

```text
Reduced available physical memory on AZMON-WIN01
```

---

## Affected Resource

```text
AZMON-WIN01
```

---

## Root Cause

```text
Controlled PowerShell memory allocation process
```

---

## Worker Evidence

```text
PID:
2572

Observed Working Set:
1105 MB
```

---

## Memory Evidence

```text
Windows baseline:
2.79 GB available

Windows pressure:
1.73 GB available
```

---

## Monitoring Evidence

```text
Log Analytics baseline:
~2933 MB available

Log Analytics pressure:
~1809 MB available
```

---

## Remediation

```text
Stop controlled worker process
```

---

## Recovery

```text
Windows:
2.77 GB available

Log Analytics:
2915 MB available

WorkerPresent:
False
```

---

# Resolution Criteria

INC-006 was not closed until the following were verified:

```text
Correct memory counter identified:
YES

Controlled workload created:
YES

Worker process verified:
YES

Windows available memory reduced:
YES

Log Analytics available memory reduced:
YES

Memory drop visualized:
YES

Worker stopped:
YES

Temporary workload removed:
YES

Windows memory recovered:
YES

Worker process absent:
YES

Log Analytics memory recovered:
YES

Final chart shows recovery:
YES
```

---

# Operational Lessons Learned

## Lesson 1 — Never Assume the Counter Name

The first query searched for:

```text
Available MBytes
```

but the DCR was collecting:

```text
Available Bytes
```

Inventory the actual `Perf` data before changing monitoring configuration.

---

## Lesson 2 — A No-Results Query Does Not Automatically Mean Monitoring Is Broken

The Azure Monitor Agent and DCR were healthy.

The query filter was incorrect.

---

## Lesson 3 — Validate Memory at Multiple Layers

INC-006 compared:

```text
Windows OS memory
Process working set
Windows performance counter
Log Analytics
```

This provided strong incident correlation.

---

## Lesson 4 — Size Controlled Tests Conservatively

The VM had only 4 GiB RAM.

The workload was limited to approximately 1 GB to avoid destabilizing Windows.

---

## Lesson 5 — Process-Level Evidence Helps Confirm Root Cause

The controlled worker showed approximately:

```text
1105 MB
```

of memory consumption.

This closely matched the observed available-memory reduction.

---

## Lesson 6 — Time-Series Analysis Makes Memory Pressure Easy to See

The timeline showed the exact period when available memory declined and recovered.

---

## Lesson 7 — Remediation Must Be Validated

Stopping the worker was only the first step.

Recovery was verified through:

```text
Windows
Worker state
Log Analytics
Final timechart
```

---

# Skills Demonstrated

INC-006 demonstrates:

- Microsoft Azure
- Azure Monitor
- Log Analytics
- Azure Monitor Agent
- Data Collection Rules
- Windows Server
- PowerShell
- Win32_OperatingSystem
- Get-CimInstance
- Get-Process
- Stop-Process
- Process working-set analysis
- Windows performance counters
- Memory monitoring
- Available Bytes
- Performance troubleshooting
- Kusto Query Language
- Perf table analysis
- Counter discovery
- Query troubleshooting
- Time-series visualization
- Collection timestamp analysis
- Ingestion timestamp analysis
- Controlled fault simulation
- Resource-capacity troubleshooting
- Memory-pressure remediation
- Recovery verification
- Root-cause analysis
- Incident documentation
- Evidence management
- Cloud support troubleshooting

---

# Final Validation

```text
Windows Memory Baseline:
VERIFIED

Log Analytics Memory Baseline:
VERIFIED

Initial No-Results Condition:
TROUBLESHOT

Correct Memory Counter:
IDENTIFIED

DCR Modification:
NOT REQUIRED

Controlled 1 GB Memory Workload:
COMPLETE

Worker Process:
VERIFIED

Windows Memory Reduction:
VERIFIED

Log Analytics Memory Reduction:
VERIFIED

Memory Drop Timeline:
VERIFIED

Root Cause:
VERIFIED

Worker Termination:
COMPLETE

Windows Memory Recovery:
VERIFIED

Worker Absence:
VERIFIED

Log Analytics Recovery:
VERIFIED

Full Incident Timeline:
VERIFIED

Incident Status:
CLOSED
```

---

# Incident Closure

```text
Incident ID:
INC-006

Incident:
Memory Pressure / High Memory Usage

Affected Resource:
AZMON-WIN01

Total Memory:
3.99 GB

Baseline Windows Available:
2.79 GB

Baseline Windows Memory Used:
30.23%

Baseline Log Analytics Available:
~2933 MB

Collected Counter:
Memory\Available Bytes

Controlled Allocation:
1024 MB

Worker PID:
2572

Observed Worker Memory:
~1105 MB

Pressure Windows Available:
1.73 GB

Pressure Windows Memory Used:
56.59%

Pressure Log Analytics Available:
~1809–1827 MB

Root Cause:
Controlled PowerShell memory allocation process

Remediation:
Controlled worker terminated

Recovered Windows Available:
2.77 GB

Recovered Windows Memory Used:
30.78%

WorkerPresent:
False

Recovered Log Analytics Available:
2915 MB / 2.85 GB

Monitoring Recovery:
VERIFIED

Incident Status:
CLOSED
```

INC-006 successfully demonstrates controlled memory-pressure generation, Windows process and memory investigation, Azure Monitor performance-counter discovery, KQL troubleshooting, Log Analytics
validation, remediation, and end-to-end recovery verification.
