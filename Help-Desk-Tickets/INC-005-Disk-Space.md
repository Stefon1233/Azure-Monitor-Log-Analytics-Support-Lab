# INC-005 — Disk Space / Storage Capacity Monitoring

## Azure Monitor + Log Analytics Support Lab

---

## Incident Summary

**Incident ID:** INC-005
**Title:** Disk Space / Storage Capacity Monitoring
**Affected Resource:** AZMON-WIN01
**Resource Group:** RG-AZMON-SUPPORT-LAB
**Monitoring Workspace:** LAW-AZMON-SUPPORT-LAB
**Platform:** Microsoft Azure
**Operating System:** Windows Server 2022 Datacenter: Azure Edition
**Severity:** Medium / Storage Capacity Degradation
**Incident Type:** Disk Capacity Monitoring
**Status:** Resolved
**Affected Disk:** C:
**Controlled Test File:** C:\AZMON-Disk-Test.bin
**Controlled Allocation:** 10 GB
**Root Cause:** Controlled creation of a 10 GB test file on the Windows system disk
**Resolution:** Test file removed and disk free-space telemetry returned to baseline

---

# Executive Summary

INC-005 simulated a disk-capacity incident on the Azure virtual machine:

```text
AZMON-WIN01
```

The objective was to demonstrate how Windows storage utilization can be monitored and investigated using:

```text
Windows Server
PowerShell
CIM
Windows Performance Counters
Azure Monitor Agent
Data Collection Rules
Log Analytics
Perf table
Kusto Query Language
Time-series analysis
```

Before generating the incident, disk capacity was measured from both the Windows guest and Log Analytics.

The Windows baseline showed:

```text
Drive:
C:

Size:
126.45 GB

Free Space:
112.26 GB

Free Percent:
88.78%
```

Log Analytics reported approximately:

```text
115,459 MB free
```

through the:

```text
LogicalDisk
Free Megabytes
```

performance counter.

A controlled 10 GB test file was then created:

```text
C:\AZMON-Disk-Test.bin
```

The Windows guest immediately showed:

```text
Free Space:
102.26 GB

Free Percent:
80.87%
```

Local performance-counter validation showed approximately:

```text
105,218 MB free
```

Log Analytics later ingested the same reduced free-space condition.

The disk-space timeline clearly showed a reduction from approximately:

```text
115,459 MB
```

to:

```text
105,218 MB
```

representing approximately:

```text
10 GB
```

of controlled disk consumption.

The incident was remediated by deleting:

```text
C:\AZMON-Disk-Test.bin
```

Windows free space immediately recovered to:

```text
112.26 GB
88.78%
```

The local performance counter returned to approximately:

```text
115,454 MB
```

and Log Analytics subsequently reported approximately:

```text
115,458 MB
```

The final time-series chart displayed the complete incident lifecycle:

```text
Healthy Baseline
      ↓
10 GB Controlled Consumption
      ↓
Reduced Free Space
      ↓
Remediation
      ↓
Recovered Free Space
```

INC-005 demonstrates disk-capacity monitoring, controlled incident generation, guest-level validation, Azure Monitor telemetry analysis, ingestion-latency troubleshooting, remediation, and
recovery verification.

---

# Incident Objectives

The objectives of INC-005 were to:

- Establish a Windows disk-capacity baseline
- Establish a Log Analytics free-space baseline
- Confirm LogicalDisk performance-counter collection
- Measure disk capacity from the Windows guest
- Safely generate a controlled storage-capacity change
- Avoid placing the operating system at risk
- Create a reversible 10 GB test file
- Validate the test file locally
- Confirm reduced free space through CIM
- Confirm reduced free space through Windows performance counters
- Verify the changed value reached Log Analytics
- Compare collection time with ingestion time
- Visualize the disk-space decline through KQL
- Remove the controlled test file
- Verify Windows storage recovery
- Verify local performance-counter recovery
- Verify Log Analytics recovery
- Visualize the complete incident timeline
- Build reusable KQL queries for disk investigations
- Document the complete troubleshooting workflow

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

```text
AzureMonitorWindowsAgent
```

---

# Monitoring Architecture

The storage-monitoring path used in INC-005 was:

```text
AZMON-WIN01
      ↓
Windows LogicalDisk Counter
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
Disk Capacity Investigation
```

Windows-side validation used:

```text
Win32_LogicalDisk
Get-Counter
PowerShell
```

This allowed the incident to be verified independently at both the guest and monitoring layers.

---

# Why Disk Capacity Monitoring Matters

Low disk space is a common infrastructure and application support problem.

Insufficient disk capacity can cause:

```text
Application failures
Database failures
Log-writing failures
Windows Update failures
Temporary file failures
Service crashes
Backup failures
Performance degradation
Application installation failures
Operating system instability
```

Monitoring free disk capacity allows support teams to identify storage pressure before it becomes a service outage.

---

# Disk Monitoring Layers

INC-005 used several different monitoring layers.

## Windows CIM

```text
Win32_LogicalDisk
```

provided direct guest operating-system information about:

```text
Disk size
Free bytes
Free GB
Free percentage
```

---

## Windows Performance Counter

```text
\LogicalDisk(_Total)\Free Megabytes
```

provided the metric sampled by the Azure monitoring pipeline.

---

## Azure Monitor Agent

Azure Monitor Agent collected the configured performance counter.

---

## Data Collection Rule

```text
DCR-AZMON-WINDOWS
```

defined the performance counters collected from the VM.

---

## Log Analytics

Performance telemetry was stored in:

```text
Perf
```

---

## KQL

Kusto Query Language was used to:

```text
Find latest disk capacity
Review historical samples
Measure ingestion latency
Calculate minimum free space
Calculate maximum free space
Calculate disk-space change
Visualize the incident timeline
Verify recovery
```

---

# Baseline Validation

Before modifying disk capacity, the environment was validated from two independent sources.

The first source was:

```text
Log Analytics
```

The second source was:

```text
Windows Server
```

This created a known-good baseline.

---

# Log Analytics Disk Baseline

The following query was used:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| summarize arg_max(TimeGenerated, CounterValue)
    by Computer, InstanceName
| project Computer,
          Disk=InstanceName,
          FreeMB=round(CounterValue, 0)
| order by Disk asc
```

The result showed approximately:

```text
Computer:
AZMON-WIN01

Disk:
_Total

FreeMB:
115459
```

---

# Log Analytics Baseline Interpretation

The result established:

```text
LogicalDisk performance telemetry:
AVAILABLE

Azure Monitor Agent:
REPORTING

Data Collection Rule:
ACTIVE

Perf table:
RECEIVING DATA
```

This was important because the later change could be compared against a known monitoring baseline.

---

# Windows Disk Baseline

Windows disk capacity was then measured directly.

PowerShell used:

```powershell
Get-CimInstance Win32_LogicalDisk -Filter "DriveType=3"
```

The baseline showed:

```text
DeviceID:
C:

SizeGB:
126.45

FreeGB:
112.26

FreePercent:
88.78
```

---

# Windows Baseline Status

```text
System Disk:
C:

Capacity:
126.45 GB

Free Space:
112.26 GB

Used Space:
Approximately 14 GB

Free Percentage:
88.78%
```

The operating system had substantial free capacity.

---

# Safe Incident Design

The incident needed to produce a visible change without putting the VM at risk.

Because the system had approximately:

```text
112 GB
```

free, a controlled allocation of:

```text
10 GB
```

was selected.

This was large enough to produce a clear monitoring change while leaving more than:

```text
100 GB
```

free.

---

# Safety Criteria

The incident design followed these requirements:

```text
Do not fill the disk
Do not consume most available storage
Do not alter Windows system files
Do not delete production data
Use a clearly named test file
Ensure remediation is simple
Verify free capacity before testing
Delete test data after monitoring validation
```

---

# Controlled Test File

The following file was used:

```text
C:\AZMON-Disk-Test.bin
```

Target size:

```text
10 GB
```

---

# Controlled Disk Consumption

PowerShell created the controlled test file using:

```powershell
fsutil file createnew C:\AZMON-Disk-Test.bin 10737418240
```

The actual script calculated the byte value programmatically.

---

# Disk Consumption Result

Immediately after the test file was created, Windows showed:

```text
Test file:
C:\AZMON-Disk-Test.bin

Allocated:
10 GB

Free space:
102.26 GB

Free percent:
80.87%

Status:
CONTROLLED DISK TEST ACTIVE
```

---

# Capacity Change

The guest-level free-space change was approximately:

```text
Before:
112.26 GB

After:
102.26 GB

Difference:
10 GB
```

This matched the intended controlled allocation.

---

# Test File Validation

The file was then independently validated using:

```powershell
Get-Item "C:\AZMON-Disk-Test.bin"
```

The result showed:

```text
TestFile:
C:\AZMON-Disk-Test.bin

FileSizeGB:
10

FreeGB:
102.26

FreePercent:
80.87
```

This confirmed that the disk-capacity change was caused by the intended lab file.

---

# Root Cause Evidence

The direct cause of the reduced disk capacity was:

```text
C:\AZMON-Disk-Test.bin
```

with size:

```text
10 GB
```

This provided a clear and reversible root cause.

---

# Initial Monitoring Delay

After the test file was created, the first Log Analytics queries continued to show:

```text
FreeMB:
115459
```

This appeared inconsistent with the current Windows value.

Windows was already showing:

```text
102.26 GB free
```

while Log Analytics still showed the older free-space sample.

---

# Troubleshooting Question

The discrepancy required determining whether:

```text
Windows performance counter was stale
```

or:

```text
Log Analytics had not yet ingested a newer sample
```

This created a useful monitoring-troubleshooting scenario.

---

# Local Counter Validation

The following local PowerShell check was performed:

```powershell
$Disk = Get-CimInstance Win32_LogicalDisk -Filter "DeviceID='C:'"

$Counter = (
    Get-Counter '\LogicalDisk(_Total)\Free Megabytes'
).CounterSamples[0]
```

The output showed approximately:

```text
CIM_FreeMB:
104719

Perf_FreeMB:
105218

FreeGB:
102.26

TestFilePresent:
True
```

---

# Local Counter Interpretation

Both local sources confirmed reduced storage capacity.

```text
CIM:
Reduced

Windows Performance Counter:
Reduced

Test File:
Present
```

Therefore the guest-side monitoring counter was functioning correctly.

---

# Monitoring Pipeline Diagnosis

Because:

```text
Windows CIM
```

and:

```text
Windows Performance Counter
```

both reflected the reduced capacity, the old Log Analytics value was not caused by a guest counter problem.

The likely explanation was:

```text
normal collection and ingestion latency
```

---

# Collection vs Ingestion Investigation

A KQL query was used to expose both timestamps.

```kusto
Perf
| where TimeGenerated > ago(45m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| project TimeGenerated,
          IngestedAt=ingestion_time(),
          InstanceName,
          FreeMB=round(CounterValue, 0)
| order by TimeGenerated desc
```

---

# Reduced Free Space Reaches Log Analytics

A newer performance record eventually appeared with:

```text
FreeMB:
105218
```

This matched the local Windows performance counter.

The older records showed approximately:

```text
115459 MB
```

The newer records showed approximately:

```text
105218 MB
```

---

# End-to-End Storage Monitoring Validation

The complete monitoring path was now verified:

```text
10 GB file created
      ↓
Windows free space decreases
      ↓
LogicalDisk counter decreases
      ↓
Azure Monitor Agent collects counter
      ↓
Data Collection Rule processes telemetry
      ↓
Log Analytics ingests sample
      ↓
Perf table reports 105218 MB
```

---

# Disk Capacity Difference

Observed values:

```text
Baseline:
approximately 115459 MB

Reduced:
approximately 105218 MB
```

Difference:

```text
approximately 10241 MB
```

This is approximately:

```text
10 GB
```

and matches the controlled test.

---

# Disk Free-Space Timeline

A time-series query was used:

```kusto
Perf
| where TimeGenerated > ago(45m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| summarize FreeMB=avg(CounterValue)
    by bin(TimeGenerated, 1m)
| order by TimeGenerated asc
| render timechart
```

The chart clearly showed:

```text
~115459 MB
      ↓
~105218 MB
```

---

# Timeline Interpretation

The sharp drop corresponded directly with creation of:

```text
C:\AZMON-Disk-Test.bin
```

The time-series view made the capacity change immediately visible.

---

# Why Time-Series Visualization Matters

Tabular queries are useful for exact values.

Charts are useful for identifying:

```text
Sudden drops
Gradual disk growth
Repeated capacity changes
Recovery after cleanup
Capacity trends
```

Disk-space incidents often become easier to understand when viewed over time.

---

# Incident Condition

At the lowest point of the controlled incident:

```text
Windows Free Space:
102.26 GB

Windows Free Percentage:
80.87%

Perf Free Megabytes:
approximately 105218 MB

Test File:
10 GB

Test File Present:
True
```

The disk was not critically low.

This was intentionally a capacity-monitoring simulation rather than a dangerous near-full-disk condition.

---

# Important Lab Distinction

INC-005 demonstrates:

```text
disk capacity degradation
```

not:

```text
critical low disk space
```

The test was designed to demonstrate monitoring behavior safely.

A production alert would normally trigger at a much lower free-space threshold.

---

# Root Cause

Root cause:

```text
Controlled creation of a 10 GB file:
C:\AZMON-Disk-Test.bin
```

---

# Root Cause Classification

```text
Category:
Storage Capacity

Affected Disk:
C:

Cause Type:
Controlled file allocation

File:
C:\AZMON-Disk-Test.bin

Size:
10 GB

Result:
Reduced free disk space
```

---

# Remediation Plan

Because the capacity reduction was caused by the controlled test file, remediation was straightforward.

The remediation action was:

```text
Delete C:\AZMON-Disk-Test.bin
```

---

# Disk Space Remediation

PowerShell checked whether the file existed.

If present:

```powershell
Remove-Item "C:\AZMON-Disk-Test.bin" -Force
```

The script then recalculated disk free space.

---

# Remediation Result

The output showed:

```text
Test file removed:
YES

Free space:
112.26 GB

Free percent:
88.78%

Status:
DISK SPACE RECOVERED
```

This matched the original baseline.

---

# Immediate Windows Recovery

Windows showed an immediate increase from:

```text
102.26 GB
```

back to:

```text
112.26 GB
```

The 10 GB test allocation was fully recovered.

---

# Recovery Validation

A second independent PowerShell check was performed.

It measured:

```text
FreeGB
FreePercent
Perf_FreeMB
TestFilePresent
```

---

# Recovery Result

Observed:

```text
FreeGB:
112.26

FreePercent:
88.78

Perf_FreeMB:
115454

TestFilePresent:
False
```

---

# Recovery Interpretation

All local indicators confirmed remediation.

```text
Test file:
REMOVED

CIM free space:
RECOVERED

Windows performance counter:
RECOVERED

Free percentage:
RECOVERED
```

---

# Log Analytics Recovery

Log Analytics was then monitored for a new performance sample.

A new record eventually showed approximately:

```text
FreeMB:
115458
```

The immediately preceding low-capacity records still showed:

```text
105218 MB
```

This provided clear before-and-after evidence.

---

# Recovery Telemetry Path

The recovery followed the same telemetry path:

```text
Test file deleted
      ↓
Windows free space increases
      ↓
LogicalDisk counter increases
      ↓
AMA collects new value
      ↓
DCR processes value
      ↓
Log Analytics ingests value
      ↓
Perf reports ~115458 MB
```

---

# Final Incident Timeline

The final KQL timechart showed:

```text
Healthy Baseline
~115459 MB
      ↓
Controlled 10 GB File Created
      ↓
Free Space Drops
~105218 MB
      ↓
Controlled Incident Maintained
      ↓
Test File Deleted
      ↓
Free Space Returns
~115458 MB
```

This single chart visualized the complete incident lifecycle.

---

# Baseline vs Incident vs Recovery

## Baseline

```text
Windows FreeGB:
112.26

Windows FreePercent:
88.78

Perf FreeMB:
~115459
```

---

## Incident

```text
Windows FreeGB:
102.26

Windows FreePercent:
80.87

Perf FreeMB:
~105218

Test File:
10 GB
```

---

## Recovery

```text
Windows FreeGB:
112.26

Windows FreePercent:
88.78

Perf FreeMB:
~115458

Test File:
Absent
```

---

# Capacity Delta

Approximate monitored delta:

```text
115459 MB - 105218 MB
=
10241 MB
```

Converted to GB:

```text
10241 / 1024
≈
10 GB
```

This matched the intended incident design.

---

# Incident Impact

The controlled capacity reduction did not cause:

```text
VM outage
Application outage
Heartbeat outage
Azure Monitor Agent outage
Windows instability
```

The objective was storage telemetry validation.

---

# Production Low-Disk Impact

In a real environment, low disk space could affect:

```text
Windows Update
IIS
SQL Server
Application logs
Temporary files
Page files
Backup software
Antivirus
System services
Software deployment
User profiles
```

A support engineer should investigate before the disk becomes critically full.

---

# Disk Capacity Investigation Workflow

A practical workflow is:

```text
Disk Alert or User Report
      ↓
Identify Affected Server
      ↓
Check Current Free Space
      ↓
Check Free Percentage
      ↓
Review Perf History
      ↓
Determine Rate of Consumption
      ↓
Identify Large Files or Directories
      ↓
Identify Recent Changes
      ↓
Determine Root Cause
      ↓
Clean or Expand Storage
      ↓
Validate Windows Free Space
      ↓
Validate Monitoring Recovery
      ↓
Close Incident
```

---

# Production Investigation Questions

Useful questions include:

```text
Which disk is affected?
How much free space remains?
What percentage is free?
When did space begin decreasing?
Was the change sudden or gradual?
Which files are largest?
Did an application begin generating logs?
Was software recently installed?
Did Windows Update download content?
Did backups fail?
Is temporary data accumulating?
Is the disk expected to grow?
Can files be removed safely?
Should the disk be expanded?
```

---

# Large File Investigation

In a real incident, PowerShell could be used to locate large files.

Example:

```powershell
Get-ChildItem C:\ -File -Recurse -ErrorAction SilentlyContinue |
    Sort-Object Length -Descending |
    Select-Object -First 20 FullName,
        @{Name="SizeGB";Expression={[math]::Round($_.Length / 1GB,2)}}
```

This should be used carefully because recursive scans can be resource intensive.

---

# Directory Investigation

Common locations to investigate include:

```text
C:\Windows\Temp
C:\Temp
Application log directories
IIS log directories
User profile directories
Software distribution caches
Backup staging directories
Crash dump directories
```

The correct remediation depends on the root cause.

---

# Storage Remediation Options

Possible production remediations include:

```text
Delete unnecessary temporary files
Archive logs
Configure log rotation
Remove obsolete installers
Clean Windows Update files
Move data to another volume
Expand the managed disk
Extend the Windows partition
Correct application logging
Correct backup retention
Add monitoring alerts
```

---

# Why Deleting Files Requires Care

Support engineers should not remove unknown files solely to increase free space.

Before deletion, determine:

```text
File ownership
Application dependency
Retention requirement
Backup requirement
Business impact
Security implications
```

INC-005 used a known controlled file, so deletion was safe.

---

# Performance Counter Used

The lab monitored:

```text
ObjectName:
LogicalDisk

CounterName:
Free Megabytes

InstanceName:
_Total
```

---

# _Total Instance

The collected performance data used:

```text
_Total
```

as the disk instance.

For environments with multiple disks, production monitoring may need per-volume counters such as:

```text
C:
D:
E:
```

depending on the Data Collection Rule configuration.

---

# Free MB vs Free Percentage

INC-005 monitored:

```text
Free Megabytes
```

Production monitoring often benefits from also collecting:

```text
% Free Space
```

because a fixed MB threshold may not scale well across disks of different sizes.

---

# Example Threshold Problem

Consider:

```text
Disk A:
64 GB

Disk B:
4 TB
```

A threshold of:

```text
20 GB free
```

means very different things on those disks.

Percentage-based monitoring provides additional context.

---

# Example Free-Space Alert

A future alert could detect a fixed threshold such as:

```text
FreeMB < 20480
```

which represents approximately:

```text
20 GB
```

free.

The reusable KQL repository contains an example.

---

# Threshold Query

```kusto
Perf
| where TimeGenerated > ago(15m)
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| summarize arg_max(TimeGenerated, CounterValue)
    by Computer, InstanceName
| where CounterValue < 20480
| project TimeGenerated,
          Computer,
          Disk=InstanceName,
          FreeMB=round(CounterValue,0)
| order by FreeMB asc
```

---

# Production Alert Design

A real disk-space alert might use:

```text
Warning:
Less than 20% free

Critical:
Less than 10% free

Emergency:
Less than 5% free
```

Exact thresholds depend on workload requirements.

---

# Alert Evaluation Window

A production alert should consider whether the low-space state persists.

Example:

```text
Evaluation:
Every 5 minutes

Condition:
Below threshold for 10 minutes
```

This can reduce noise from short-lived disk activity.

---

# Ingestion Latency Lesson

One of the strongest troubleshooting lessons from INC-005 was the difference between:

```text
current guest state
```

and:

```text
latest cloud telemetry
```

Immediately after the disk change:

```text
Windows:
102.26 GB free
```

while Log Analytics still showed:

```text
115459 MB free
```

This was not a monitoring failure.

A newer sample had not yet arrived.

---

# Collection Timestamp

```text
TimeGenerated
```

represents the event or metric collection timestamp.

---

# Ingestion Timestamp

```text
ingestion_time()
```

shows approximately when Log Analytics ingested the record.

Comparing the two helps investigate telemetry delay.

---

# Monitoring Troubleshooting Model

When a metric appears stale:

```text
Check local resource state
      ↓
Check local performance counter
      ↓
Check AMA health
      ↓
Check DCR
      ↓
Check Heartbeat
      ↓
Check latest TimeGenerated
      ↓
Check ingestion_time()
      ↓
Wait for next collection interval if appropriate
      ↓
Escalate only if telemetry remains stale
```

---

# Why Local Counter Validation Was Important

Without local counter validation, it would have been unclear whether:

```text
Windows counter failed to update
```

or:

```text
Azure telemetry was delayed
```

The local result:

```text
Perf_FreeMB:
105218
```

proved the counter itself was correct.

---

# Cross-Layer Validation

INC-005 validated the storage condition through:

```text
Win32_LogicalDisk
+
Windows Performance Counter
+
Azure Monitor Agent
+
Log Analytics Perf
+
KQL timechart
```

This creates higher confidence than relying on one monitoring source.

---

# KQL Repository

Reusable INC-005 queries are stored in:

```text
KQL/Disk-Space-Queries.kql
```

The file contains:

```text
8 reusable queries
```

---

# Query 01 — Latest Disk Free-Space Baseline

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| summarize arg_max(TimeGenerated, CounterValue)
    by Computer, InstanceName
| project TimeGenerated,
          Computer,
          Disk=InstanceName,
          FreeMB=round(CounterValue, 0)
| order by Disk asc
```

Purpose:

```text
Determine the newest available disk free-space value.
```

---

# Query 02 — Recent Free-Space Samples

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| project TimeGenerated,
          Computer,
          Disk=InstanceName,
          FreeMB=round(CounterValue, 0)
| order by TimeGenerated desc
```

Purpose:

```text
Review disk-capacity changes sample by sample.
```

---

# Query 03 — Collection vs Ingestion

```kusto
Perf
| where TimeGenerated > ago(45m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| project TimeGenerated,
          IngestedAt=ingestion_time(),
          Disk=InstanceName,
          FreeMB=round(CounterValue, 0)
| order by TimeGenerated desc
```

Purpose:

```text
Troubleshoot delays between metric collection and Log Analytics visibility.
```

---

# Query 04 — Disk Free-Space Timeline

```kusto
Perf
| where TimeGenerated > ago(60m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| summarize FreeMB=avg(CounterValue)
    by bin(TimeGenerated, 1m)
| order by TimeGenerated asc
| render timechart
```

Purpose:

```text
Visualize baseline, capacity reduction, and recovery.
```

---

# Query 05 — Minimum and Maximum Free Space

```kusto
Perf
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| summarize MinimumFreeMB=min(CounterValue),
            MaximumFreeMB=max(CounterValue)
            by Computer, InstanceName
| extend ChangeMB=MaximumFreeMB-MinimumFreeMB
| project Computer,
          Disk=InstanceName,
          MinimumFreeMB=round(MinimumFreeMB,0),
          MaximumFreeMB=round(MaximumFreeMB,0),
          ChangeMB=round(ChangeMB,0)
```

Purpose:

```text
Quantify the largest observed free-space change.
```

---

# Query 06 — Low Disk Threshold Detection

```kusto
Perf
| where TimeGenerated > ago(15m)
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| summarize arg_max(TimeGenerated, CounterValue)
    by Computer, InstanceName
| where CounterValue < 20480
| project TimeGenerated,
          Computer,
          Disk=InstanceName,
          FreeMB=round(CounterValue,0)
| order by FreeMB asc
```

Purpose:

```text
Identify monitored disks below 20 GB free.
```

---

# Query 07 — Recovery Verification

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| top 10 by TimeGenerated desc
| project TimeGenerated,
          IngestedAt=ingestion_time(),
          Disk=InstanceName,
          FreeMB=round(CounterValue,0)
```

Purpose:

```text
Verify the newest samples returned to the expected free-space baseline.
```

---

# Query 08 — Incident Free-Space Change

```kusto
Perf
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where ObjectName =~ "LogicalDisk"
| where CounterName =~ "Free Megabytes"
| summarize BaselineMaxMB=max(CounterValue),
            IncidentMinMB=min(CounterValue)
            by Computer
| extend ConsumedMB=BaselineMaxMB-IncidentMinMB
| extend ConsumedGB=ConsumedMB/1024
| project Computer,
          BaselineMaxMB=round(BaselineMaxMB,0),
          IncidentMinMB=round(IncidentMinMB,0),
          ConsumedMB=round(ConsumedMB,0),
          ConsumedGB=round(ConsumedGB,2)
```

Purpose:

```text
Calculate the approximate disk capacity consumed during the incident.
```

---

# Evidence Directory

INC-005 evidence is stored in:

```text
Screenshots/10-Disk-Space/
```

---

# Evidence 01

```text
01-AZMON-WIN01-Disk-Free-Space-Baseline.png
```

Demonstrates:

```text
Log Analytics LogicalDisk telemetry active
Baseline FreeMB approximately 115459
Perf data available before incident
```

---

# Evidence 02

```text
02-AZMON-WIN01-Disk-Space-Windows-Baseline.png
```

Demonstrates:

```text
C: capacity 126.45 GB
Free space 112.26 GB
Free percentage 88.78%
Known-good Windows baseline
```

---

# Evidence 03

```text
03-AZMON-WIN01-Controlled-Disk-Consumption.png
```

Demonstrates:

```text
10 GB controlled file created
Free space reduced to 102.26 GB
Free percentage reduced to 80.87%
Controlled test active
```

---

# Evidence 04

```text
04-AZMON-WIN01-Disk-Space-Reduced.png
```

Demonstrates:

```text
C:\AZMON-Disk-Test.bin exists
File size = 10 GB
Reduced Windows disk capacity confirmed
```

---

# Evidence 05

```text
05-AZMON-WIN01-Local-Disk-Counter-Reduced.png
```

Demonstrates:

```text
CIM reduced free-space value
Local LogicalDisk performance counter reduced
Test file present
Guest telemetry source operating correctly
```

---

# Evidence 06

```text
06-AZMON-WIN01-Disk-Free-Space-Reduced-Verified.png
```

Demonstrates:

```text
Reduced LogicalDisk performance value reached Log Analytics
Approximate FreeMB = 105218
Older baseline samples still visible
Collection and ingestion timestamps available
```

---

# Evidence 07

```text
07-AZMON-WIN01-Disk-Free-Space-Drop-Timeline.png
```

Demonstrates:

```text
Clear time-series drop
Baseline approximately 115459 MB
Incident approximately 105218 MB
```

---

# Evidence 08

```text
08-AZMON-WIN01-Disk-Test-File-Removed.png
```

Demonstrates:

```text
Controlled test file removed
Free space returned to 112.26 GB
Free percentage returned to 88.78%
Remediation successful
```

---

# Evidence 09

```text
09-AZMON-WIN01-Disk-Space-Recovery-Validated.png
```

Demonstrates:

```text
FreeGB recovered
FreePercent recovered
Local Perf_FreeMB recovered
TestFilePresent = False
```

---

# Evidence 10

```text
10-AZMON-WIN01-Disk-Free-Space-Recovery-Verified.png
```

Demonstrates:

```text
Newest Log Analytics sample approximately 115458 MB
Previous reduced samples approximately 105218 MB
Monitoring recovery verified
```

---

# Evidence 11

```text
11-AZMON-WIN01-Disk-Space-Full-Incident-Timeline.png
```

Demonstrates the complete incident lifecycle:

```text
Healthy
↓
10 GB Reduction
↓
Reduced Capacity
↓
Remediation
↓
Recovered Capacity
```

---

# Evidence Progression

```text
01 — Log Analytics Storage Baseline
      ↓
02 — Windows Storage Baseline
      ↓
03 — Controlled 10 GB Consumption
      ↓
04 — Test File and Reduced Space Validated
      ↓
05 — Local Performance Counter Reduced
      ↓
06 — Reduced Capacity Reaches Log Analytics
      ↓
07 — Disk-Space Drop Visualized
      ↓
08 — Test File Removed
      ↓
09 — Windows Recovery Validated
      ↓
10 — Log Analytics Recovery Verified
      ↓
11 — Complete Incident Timeline
```

---

# Incident Timeline

```text
Initial State:
AZMON-WIN01 healthy

Windows Free Space:
112.26 GB

Windows Free Percent:
88.78%

Log Analytics FreeMB:
~115459

Controlled Incident:
10 GB test file created

Test File:
C:\AZMON-Disk-Test.bin

Windows Free Space:
102.26 GB

Windows Free Percent:
80.87%

Local Perf Counter:
~105218 MB

Initial Log Analytics Result:
Old baseline sample still visible

Troubleshooting:
Local CIM checked

Troubleshooting:
Local LogicalDisk counter checked

Finding:
Local counter correctly reflected reduced space

Diagnosis:
Monitoring pipeline awaiting newer sample

New Perf Sample:
~105218 MB

Disk Timeline:
Drop verified

Remediation:
C:\AZMON-Disk-Test.bin deleted

Windows Free Space:
112.26 GB

Windows Free Percent:
88.78%

Local Perf Counter:
~115454 MB

TestFilePresent:
False

Log Analytics Recovery:
~115458 MB

Full Timeline:
Healthy → Reduced → Recovered

Incident:
RESOLVED
```

---

# Incident Correlation

The evidence chain was:

```text
Known-Good Disk Baseline
      ↓
Known 10 GB File Created
      ↓
Windows Capacity Decreased
      ↓
Windows Counter Decreased
      ↓
Log Analytics Perf Decreased
      ↓
Timechart Confirmed Decrease
      ↓
Known File Deleted
      ↓
Windows Capacity Recovered
      ↓
Windows Counter Recovered
      ↓
Log Analytics Perf Recovered
      ↓
Timechart Confirmed Recovery
```

---

# Root Cause Analysis

## Symptom

```text
Reduced free disk space on AZMON-WIN01
```

---

## Affected Resource

```text
AZMON-WIN01
```

---

## Affected Storage

```text
C:
```

---

## Root Cause

```text
Controlled 10 GB test file:
C:\AZMON-Disk-Test.bin
```

---

## Evidence

```text
FileSizeGB:
10

FreeGB before:
112.26

FreeGB during:
102.26
```

---

## Monitoring Confirmation

```text
Perf before:
~115459 MB

Perf during:
~105218 MB
```

---

## Remediation

```text
Delete C:\AZMON-Disk-Test.bin
```

---

## Recovery

```text
Windows:
112.26 GB free

Perf:
~115458 MB
```

---

# Resolution Criteria

INC-005 was not closed until the following were verified:

```text
Controlled test file removed:
YES

Windows free space recovered:
YES

Windows free percentage recovered:
YES

Local LogicalDisk counter recovered:
YES

Log Analytics Perf recovered:
YES

Final timechart shows recovery:
YES
```

---

# Operational Lessons Learned

## Lesson 1 — Validate Disk Capacity from More Than One Layer

Windows CIM and performance counters provided direct guest evidence.

Log Analytics provided centralized monitoring evidence.

Using both improved confidence.

---

## Lesson 2 — Monitoring Data Can Lag Behind Current State

Immediately after disk allocation, Windows showed the new value before Log Analytics did.

This did not indicate monitoring failure.

---

## Lesson 3 — Check the Source Counter Before Troubleshooting Azure

The local:

```text
LogicalDisk(_Total)\Free Megabytes
```

counter showed the correct reduced value.

This prevented unnecessary DCR or AMA troubleshooting.

---

## Lesson 4 — Collection and Ingestion Timestamps Matter

Comparing:

```text
TimeGenerated
```

with:

```text
ingestion_time()
```

helps identify telemetry delay.

---

## Lesson 5 — Time-Series Charts Make Capacity Problems Obvious

The free-space chart clearly showed both the controlled reduction and recovery.

---

## Lesson 6 — Safe Failure Simulation Is Important

The test used only 10 GB while more than 100 GB remained available.

The lab did not intentionally approach a dangerous full-disk condition.

---

## Lesson 7 — Incident Closure Requires Recovery Validation

Deleting the file was not enough.

Recovery was validated through:

```text
Windows
Local Perf
Log Analytics
KQL chart
```

---

# Skills Demonstrated

INC-005 demonstrates:

- Microsoft Azure
- Azure Monitor
- Log Analytics
- Azure Monitor Agent
- Data Collection Rules
- Windows Server
- PowerShell
- Win32_LogicalDisk
- Get-CimInstance
- Get-Counter
- Windows performance counters
- LogicalDisk monitoring
- Storage capacity troubleshooting
- Free-space monitoring
- Kusto Query Language
- Perf table analysis
- Time-series visualization
- Collection timestamp analysis
- Ingestion timestamp analysis
- Monitoring latency troubleshooting
- Controlled fault simulation
- Disk-capacity remediation
- Recovery validation
- Root-cause analysis
- Incident documentation
- Evidence management
- Cloud support troubleshooting

---

# Final Validation

```text
Windows Disk Baseline:
VERIFIED

Log Analytics Disk Baseline:
VERIFIED

Controlled 10 GB Consumption:
COMPLETE

Controlled File:
VERIFIED

Windows Free-Space Reduction:
VERIFIED

Local Performance Counter Reduction:
VERIFIED

Log Analytics Reduction:
VERIFIED

Disk Timeline:
VERIFIED

Monitoring Latency Investigation:
COMPLETE

Root Cause:
VERIFIED

Controlled File Removal:
COMPLETE

Windows Free-Space Recovery:
VERIFIED

Local Counter Recovery:
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
INC-005

Incident:
Disk Space / Storage Capacity Monitoring

Affected Resource:
AZMON-WIN01

Affected Disk:
C:

Baseline Free Space:
112.26 GB

Baseline Free Percentage:
88.78%

Controlled Allocation:
10 GB

Controlled Test File:
C:\AZMON-Disk-Test.bin

Incident Free Space:
102.26 GB

Incident Free Percentage:
80.87%

Baseline Perf:
~115459 MB

Incident Perf:
~105218 MB

Root Cause:
Controlled 10 GB test file allocation

Remediation:
Test file deleted

Recovered Free Space:
112.26 GB

Recovered Free Percentage:
88.78%

Recovered Perf:
~115458 MB

Test File Present:
False

Monitoring Recovery:
VERIFIED

Incident Status:
CLOSED
```

INC-005 successfully demonstrates storage-capacity monitoring, controlled disk consumption, Windows and Azure Monitor performance-counter validation, telemetry-latency troubleshooting,
remediation, and end-to-end recovery verification.
