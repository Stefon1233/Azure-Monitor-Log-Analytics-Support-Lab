# INC-004 — Windows Application Event Investigation

## Azure Monitor + Log Analytics Support Lab

---

## Incident Summary

**Incident ID:** INC-004
**Title:** Windows Application Event Investigation
**Affected Resource:** AZMON-WIN01
**Resource Group:** RG-AZMON-SUPPORT-LAB
**Monitoring Workspace:** LAW-AZMON-SUPPORT-LAB
**Platform:** Microsoft Azure
**Operating System:** Windows Server 2022 Datacenter: Azure Edition
**Severity:** Medium / Application Error Investigation
**Incident Type:** Windows Event Log Monitoring
**Status:** Resolved
**Controlled Event Source:** AZMON-AppLab
**Controlled Event ID:** 200
**Event Level:** Error
**Root Cause:** Controlled Windows Application error generated through Azure VM Run Command
**Resolution:** Event successfully detected, correlated, and validated without VM or monitoring disruption

---

# Executive Summary

INC-004 simulated a Windows application error on the Azure virtual machine:

```text
AZMON-WIN01
```

The purpose of the incident was to demonstrate end-to-end Windows Event Log monitoring using:

```text
Windows Event Log
Azure Monitor Agent
Data Collection Rule
Log Analytics
Kusto Query Language
Azure Activity Log
AzureActivity
Heartbeat
```

Before generating the incident, Windows Event ingestion was validated in Log Analytics.

The environment was already collecting Windows Application and System warning events from sources including:

```text
Microsoft-Windows-DistributedCOM
Microsoft-Windows-PerfOS
Microsoft-Windows-WinRM
Microsoft-Windows-Wininit
```

A controlled Application event was then generated through Azure VM Run Command.

The event was created with:

```text
Event Log:
Application

Event Level:
Error

Source:
AZMON-AppLab

Event ID:
200
```

Local Windows validation confirmed that the event was successfully written to the Application log.

Azure Monitor Agent then collected the event according to the configured Data Collection Rule and delivered it to:

```text
LAW-AZMON-SUPPORT-LAB
```

The event appeared in the Log Analytics:

```text
Event
```

table with the expected:

```text
Computer
EventLog
EventLevelName
Source
EventID
```

fields.

AzureActivity was then used to correlate the Windows event with the Azure Run Command operation that generated it.

The control-plane timeline showed:

```text
Run Command Start:
approximately 03:02:58 UTC

Run Command Accept:
approximately 03:02:58 UTC

Windows Event:
approximately 03:03:09 UTC

Run Command Success:
approximately 03:03:28 UTC
```

This placed the Application Error directly inside the Azure Run Command execution window.

Finally, Heartbeat telemetry was checked after the event.

Heartbeat remained healthy:

```text
MinutesSinceHeartbeat:
1

HeartbeatCount:
14
```

This confirmed that the Windows Application error represented a monitored application-level event and did not cause a VM availability or Azure Monitor Agent outage.

INC-004 demonstrates controlled Windows event generation, event ingestion, KQL investigation, Azure control-plane correlation, and post-event health validation.

---

# Incident Objectives

The objectives of INC-004 were to:

- Establish a Windows Event ingestion baseline
- Verify Application and System event collection
- Review existing event sources and severity levels
- Generate a controlled Windows Application Error
- Validate the error locally on the Windows guest
- Confirm Event ID 200
- Confirm source AZMON-AppLab
- Confirm Error severity
- Verify ingestion into Log Analytics
- Isolate the controlled event with KQL
- Review the surrounding Application Event timeline
- Correlate the event with Azure VM Run Command
- Confirm Run Command Start, Accept, and Success lifecycle
- Verify the VM continued reporting Heartbeat
- Demonstrate the difference between application errors and infrastructure outages
- Store reusable Windows Event KQL queries
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

The monitoring path relevant to this incident was:

```text
Windows Server
AZMON-WIN01
      ↓
Windows Application Event Log
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Event Table
      ↓
KQL Investigation
```

The Azure control-plane path was:

```text
Azure Portal
      ↓
VM Run Command
      ↓
Azure Resource Manager
      ↓
Azure Activity Log
      ↓
DIAG-AZMON-ACTIVITY-LOG
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
AzureActivity
```

Together, these paths allowed the guest event to be correlated with the Azure administrative action that generated it.

---

# Why Windows Event Monitoring Matters

Windows Event Logs provide operating-system and application-level telemetry.

Common use cases include investigating:

```text
Application failures
Service failures
Authentication problems
Driver issues
System startup problems
Windows Update failures
Distributed COM issues
Application crashes
Performance warnings
Security events
Administrative changes
```

Azure Monitor allows these logs to be centralized instead of requiring administrators to manually inspect each server.

---

# Event Collection Layers

INC-004 involved several layers.

## Layer 1 — Windows Guest

The original event was generated inside:

```text
AZMON-WIN01
```

and written to:

```text
Windows Application Event Log
```

---

## Layer 2 — Azure Monitor Agent

Azure Monitor Agent collected the configured Windows events.

---

## Layer 3 — Data Collection Rule

The Data Collection Rule defined which Windows event levels were collected.

The lab configuration included Application events such as:

```text
Critical
Error
Warning
```

---

## Layer 4 — Log Analytics

Collected events were written into:

```text
Event
```

---

## Layer 5 — KQL

Kusto Query Language was used to:

```text
Validate ingestion
Filter by computer
Filter by event log
Filter by severity
Filter by source
Filter by Event ID
Build timelines
```

---

## Layer 6 — AzureActivity

AzureActivity was used to identify the Run Command operation used to generate the test event.

---

# Baseline Validation

Before generating the controlled error, Windows Event collection was validated.

This ensured the test began from a known-good monitoring state.

---

# Windows Event Ingestion Baseline

The following query was executed:

```kusto
Event
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| summarize EventCount=count(),
            LastEvent=max(TimeGenerated)
            by Computer
```

The query returned:

```text
Computer:
AZMON-WIN01

EventCount:
4
```

A recent event timestamp was also present.

---

# Baseline Interpretation

The baseline proved:

```text
Azure Monitor Agent:
COLLECTING WINDOWS EVENTS

Data Collection Rule:
ACTIVE

Log Analytics:
RECEIVING EVENTS

Event Table:
AVAILABLE
```

This was important because a failure to detect the later controlled event could otherwise have been caused by a pre-existing ingestion problem.

---

# Windows Event Source Summary

The next query reviewed recent Windows event sources:

```kusto
Event
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| summarize Count=count()
    by EventLog, EventLevelName, Source
| order by Count desc
```

The result included events from both:

```text
Application
System
```

logs.

---

# Baseline Event Sources

Observed sources included:

```text
Microsoft-Windows-DistributedCOM
Microsoft-Windows-PerfOS
Microsoft-Windows-WinRM
Microsoft-Windows-Wininit
```

All visible baseline events were:

```text
Warning
```

level.

---

# Baseline Event Logs

The baseline demonstrated that the Data Collection Rule was successfully collecting from:

```text
Application
```

and:

```text
System
```

event logs.

This established a broader event-monitoring baseline before the controlled Application Error was created.

---

# Controlled Incident Design

The test required a Windows event that could be uniquely identified later.

The following attributes were selected:

```text
Event Log:
Application

Event Level:
Error

Source:
AZMON-AppLab

Event ID:
200
```

The source name was intentionally unique to the lab.

This made the event easy to isolate in Log Analytics.

---

# Why a Unique Event Source Was Used

A generic Windows source could produce unrelated events.

Using:

```text
AZMON-AppLab
```

allowed the investigation to filter on an exact source.

This reduced ambiguity.

---

# Why Event ID 200 Was Used

The test also used:

```text
Event ID:
200
```

This provided a second unique filter.

Together:

```text
Source = AZMON-AppLab
+
EventID = 200
```

created a high-confidence match.

---

# Controlled Event Generation

Azure VM Run Command was used to execute PowerShell inside AZMON-WIN01.

The test used:

```text
RunPowerShellScript
```

The Windows utility:

```text
eventcreate.exe
```

generated the controlled Application Error.

---

# Controlled Error Command

The core event creation command was:

```powershell
eventcreate.exe `
    /T ERROR `
    /ID 200 `
    /L APPLICATION `
    /SO "AZMON-AppLab" `
    /D "Controlled application error generated for Azure Monitor Log Analytics incident validation."
```

---

# Command Parameter Interpretation

## Event Type

```text
/T ERROR
```

created an Error-level event.

---

## Event ID

```text
/ID 200
```

created Event ID 200.

---

## Event Log

```text
/L APPLICATION
```

sent the event to the Windows Application Event Log.

---

## Event Source

```text
/SO AZMON-AppLab
```

created the unique event source.

---

## Event Description

The description identified the event as a controlled lab test.

---

# Local Validation

After the event was generated, PowerShell immediately queried the Windows Application Event Log.

The validation used:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName   = 'Application'
    Id        = 200
    StartTime = (Get-Date).AddMinutes(-10)
}
```

The result was filtered for:

```text
ProviderName = AZMON-AppLab
```

---

# Local Event Result

The local Windows result showed:

```text
TimeCreated:
9/7/2026 3:03:09 AM

Id:
200

LevelDisplayName:
Error

ProviderName:
AZMON-AppLab
```

This confirmed that the event existed on the guest before Log Analytics ingestion was evaluated.

---

# Why Local Validation Matters

If the event had failed to appear in Log Analytics, the troubleshooting path would depend on whether the event existed locally.

For example:

```text
Event missing locally
      ↓
Event generation problem
```

versus:

```text
Event exists locally
but
Event missing from Log Analytics
      ↓
AMA / DCR / ingestion problem
```

Local validation therefore isolates the source layer from the collection layer.

---

# Event Creation Status

The Run Command output also reported:

```text
SUCCESS
```

for the event creation operation.

This provided immediate confirmation that Windows accepted the event.

---

# Log Analytics Ingestion Investigation

After local validation, Log Analytics was queried for the exact controlled event.

The query used:

```kusto
Event
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where EventID == 200
| where Source =~ "AZMON-AppLab"
| project TimeGenerated,
          Computer,
          EventLog,
          EventLevelName,
          Source,
          EventID
| order by TimeGenerated desc
```

---

# Controlled Event Ingestion Result

The result showed:

```text
Computer:
AZMON-WIN01

EventLog:
Application

EventLevelName:
Error

Source:
AZMON-AppLab

EventID:
200
```

The event timestamp was approximately:

```text
03:03:09 UTC
```

---

# End-to-End Event Validation

The same event was now proven at two separate layers.

## Windows Guest

```text
Application
Error
AZMON-AppLab
Event ID 200
```

## Log Analytics

```text
Application
Error
AZMON-AppLab
Event ID 200
```

This confirmed successful collection and ingestion.

---

# Event Ingestion Path Validation

The following path was therefore validated:

```text
Windows Event Log
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Event Table
```

---

# Application Event Timeline

A broader Application Event query was then executed:

```kusto
Event
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where EventLog =~ "Application"
| project TimeGenerated,
          EventLevelName,
          Source,
          EventID
| order by TimeGenerated desc
```

---

# Timeline Result

The controlled event appeared at the top of the recent Application Event timeline.

Observed:

```text
03:03:09 UTC
Error
AZMON-AppLab
200
```

A previous Application warning from:

```text
Microsoft-Windows-PerfOS
```

was also visible.

---

# Why the Timeline Query Matters

Filtering only by Event ID identifies the known incident.

The broader timeline provides context.

It helps answer:

```text
Were other events occurring nearby?
Was this event isolated?
Were related errors generated?
Was there a sequence of application failures?
```

This is a common troubleshooting technique.

---

# AzureActivity Correlation

The event was generated through Azure VM Run Command.

AzureActivity was therefore queried to correlate the guest event with the Azure administrative operation.

---

# Run Command AzureActivity Query

The following query was used:

```kusto
AzureActivity
| where TimeGenerated > ago(4h)
| where ResourceGroup =~ "RG-AZMON-SUPPORT-LAB"
| where OperationNameValue =~ "MICROSOFT.COMPUTE/VIRTUALMACHINES/RUNCOMMAND/ACTION"
| where TimeGenerated > datetime(2026-09-07 03:00:00)
| project TimeGenerated,
          OperationNameValue,
          ActivityStatusValue,
          ResourceGroup
| order by TimeGenerated asc
```

---

# Run Command Operation

AzureActivity identified:

```text
MICROSOFT.COMPUTE/VIRTUALMACHINES/RUNCOMMAND/ACTION
```

---

# Run Command Lifecycle

Observed lifecycle:

```text
Start
Accept
Success
```

Approximate timestamps:

```text
03:02:58 UTC — Start
03:02:58 UTC — Accept
03:03:28 UTC — Success
```

---

# Windows Event Timestamp

The controlled Windows event occurred at approximately:

```text
03:03:09 UTC
```

---

# Timeline Correlation

The combined timeline was:

```text
03:02:58 UTC
Azure Run Command Start

03:02:58 UTC
Azure Run Command Accept

03:03:09 UTC
Windows Application Error 200 created

03:03:28 UTC
Azure Run Command Success
```

This placed the guest event directly inside the Run Command execution window.

---

# Why This Correlation Is Useful

A support engineer may need to determine whether a guest event was associated with an Azure administrative action.

The relationship may look like:

```text
Azure Administrative Operation
      ↓
Guest OS Change
      ↓
Windows Event
```

AzureActivity can provide the control-plane context while the Event table provides the guest context.

---

# Guest Plane vs Control Plane

INC-004 demonstrates two different telemetry planes.

## Guest Plane

```text
Windows Event Log
Azure Monitor Agent
Event table
```

Answers:

```text
What happened inside Windows?
```

---

## Control Plane

```text
Azure Activity Log
AzureActivity
```

Answers:

```text
What Azure administrative operation occurred?
```

---

# Correlation Value

When both sources are available, administrators can build a more complete incident timeline.

```text
Control Plane Action
+
Guest OS Event
=
Higher-confidence investigation
```

---

# Post-Event Health Validation

A Windows Application Error does not automatically mean the VM is unavailable.

The system therefore required a post-event Heartbeat check.

---

# Heartbeat Validation Query

The following query was executed:

```kusto
Heartbeat
| where Computer =~ "AZMON-WIN01"
| where TimeGenerated > ago(15m)
| summarize LastHeartbeat=max(TimeGenerated),
            HeartbeatCount=count()
| extend MinutesSinceHeartbeat=datetime_diff("minute", now(), LastHeartbeat)
| project LastHeartbeat,
          MinutesSinceHeartbeat,
          HeartbeatCount
```

---

# Heartbeat Result

Observed:

```text
LastHeartbeat:
approximately 03:06:16 UTC

MinutesSinceHeartbeat:
1

HeartbeatCount:
14
```

This confirmed the VM continued reporting normally.

---

# Post-Incident Infrastructure Status

```text
VM:
RUNNING

Azure Monitor Agent:
REPORTING

Heartbeat:
HEALTHY

Event ingestion:
HEALTHY

Log Analytics:
HEALTHY
```

---

# Application Event vs Infrastructure Failure

INC-004 intentionally generated:

```text
Application Error
```

but did not create:

```text
VM outage
AMA outage
Heartbeat outage
Network outage
Platform availability outage
```

This distinction is important.

---

# Comparison with Previous Incidents

## INC-001

```text
High CPU
```

demonstrated metric-based performance monitoring and alerting.

---

## INC-002

```text
Missing Heartbeat
```

demonstrated an Azure Monitor Agent interruption while the VM remained available.

---

## INC-003

```text
VM Availability
```

demonstrated a full VM deallocation where both availability and Heartbeat were affected.

---

## INC-004

```text
Windows Application Event
```

demonstrates guest operating-system event investigation while infrastructure remains healthy.

---

# Monitoring Layer Comparison

```text
INC-001:
Performance layer

INC-002:
Monitoring agent layer

INC-003:
Infrastructure availability layer

INC-004:
Guest operating system / application event layer
```

This expands the project beyond a single monitoring method.

---

# Root Cause

Root cause:

```text
Controlled Windows Application Error generated through Azure VM Run Command
```

The event was intentionally created for monitoring and troubleshooting validation.

---

# Root Cause Classification

```text
Category:
Windows Application Event

Component:
Windows Application Event Log

Source:
AZMON-AppLab

Event ID:
200

Severity:
Error

Cause Type:
Controlled lab generation
```

---

# Incident Impact

The event represented an application-level error condition.

Observed impact:

```text
VM availability:
No impact

Heartbeat:
No impact

Azure Monitor Agent:
No impact

Log Analytics:
No impact

Windows event collection:
Operational
```

---

# Production Interpretation

In a production environment, the same investigative workflow could apply to a genuine Application Error.

The event source might represent:

```text
Application service
Database service
Web server
Backup agent
Middleware
Custom application
Windows component
Third-party software
```

The administrator would then evaluate the event description and surrounding telemetry.

---

# Production Investigation Questions

Useful questions include:

```text
Which computer generated the event?
Which Event Log contains it?
What is the severity?
What is the Event ID?
What is the source?
When did it occur?
How often has it occurred?
Did other events happen nearby?
Did an Azure administrative action occur?
Did CPU or memory change?
Did the application stop?
Did Heartbeat remain healthy?
Did the VM remain available?
Did the event repeat?
```

---

# Severity Filtering

The Event table can be filtered by severity.

Common values include:

```text
Critical
Error
Warning
Information
```

The lab DCR was configured to collect selected severity levels.

INC-004 specifically validated:

```text
Error
```

collection.

---

# Event Source Filtering

Filtering by source can isolate a particular component.

Example:

```kusto
Event
| where Source =~ "AZMON-AppLab"
```

This is useful when an application generates multiple Event IDs.

---

# Event ID Filtering

Filtering by Event ID provides another level of precision.

Example:

```kusto
Event
| where EventID == 200
```

---

# Combined Filtering

The strongest controlled-event filter used both:

```text
Source = AZMON-AppLab
EventID = 200
```

This avoided unrelated records.

---

# Error Search Query

The KQL repository also includes a reusable query for recent Windows errors:

```kusto
Event
| where TimeGenerated > ago(24h)
| where Computer =~ "AZMON-WIN01"
| where EventLevelName =~ "Error"
| project TimeGenerated,
          Computer,
          EventLog,
          Source,
          EventID
| order by TimeGenerated desc
```

This can be used as a general Windows troubleshooting query.

---

# Event Investigation Workflow

A practical workflow is:

```text
User or Monitoring Reports Error
      ↓
Identify Affected Computer
      ↓
Query Recent Events
      ↓
Filter by Severity
      ↓
Identify Source
      ↓
Identify Event ID
      ↓
Review Event Timeline
      ↓
Check AzureActivity
      ↓
Check Heartbeat
      ↓
Check Performance Metrics
      ↓
Determine Root Cause
      ↓
Remediate Application
      ↓
Validate Recovery
```

---

# Event Ingestion Troubleshooting Workflow

If a known local Windows event does not appear in Log Analytics:

```text
1. Confirm the event exists locally
2. Confirm event log name
3. Confirm severity
4. Confirm DCR collects that severity
5. Confirm DCR resource association
6. Confirm Azure Monitor Agent health
7. Confirm Heartbeat
8. Confirm Log Analytics workspace destination
9. Check ingestion delay
10. Review AMA logs if necessary
```

---

# Why the Baseline Was Important

The baseline showed that Windows events were already reaching Log Analytics.

Therefore, when Event ID 200 appeared successfully, the incident validated the existing pipeline rather than relying on assumptions.

---

# Ingestion Latency

Windows events may not appear in Log Analytics at the exact instant they are created.

The workflow allows time for:

```text
Agent collection
DCR processing
Network transmission
Log Analytics ingestion
Query visibility
```

A short delay does not automatically indicate failure.

---

# AzureActivity Ingestion Latency

AzureActivity also exhibited delayed visibility during previous phases.

The Run Command record appeared after the guest event had already been ingested.

This reinforces the lesson that different Azure telemetry sources can have different ingestion timelines.

---

# Cross-Source Timestamp Comparison

The event and AzureActivity records used UTC timestamps in Log Analytics.

This made it possible to compare:

```text
Run Command Start
Windows Event Time
Run Command Success
```

within a common timeline.

---

# Time Zone Awareness

Azure portal views may display:

```text
Local time
```

while Log Analytics often displays:

```text
UTC
```

Administrators should verify the time zone before correlating incidents.

---

# Reusable KQL Repository

INC-004 queries are stored in:

```text
KQL/Windows-Event-Incident-Queries.kql
```

The file contains:

```text
7 reusable queries
```

---

# Query 01 — Event Ingestion Baseline

```kusto
Event
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| summarize EventCount=count(),
            LastEvent=max(TimeGenerated)
            by Computer
```

Purpose:

```text
Verify Windows Event collection is currently active.
```

---

# Query 02 — Event Source Summary

```kusto
Event
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| summarize Count=count()
    by EventLog, EventLevelName, Source
| order by Count desc
```

Purpose:

```text
Identify recent event logs, severity levels, and event sources.
```

---

# Query 03 — Controlled Application Error

```kusto
Event
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where EventID == 200
| where Source =~ "AZMON-AppLab"
| project TimeGenerated,
          Computer,
          EventLog,
          EventLevelName,
          Source,
          EventID
| order by TimeGenerated desc
```

Purpose:

```text
Isolate the exact controlled incident.
```

---

# Query 04 — Application Event Timeline

```kusto
Event
| where TimeGenerated > ago(2h)
| where Computer =~ "AZMON-WIN01"
| where EventLog =~ "Application"
| project TimeGenerated,
          EventLevelName,
          Source,
          EventID
| order by TimeGenerated desc
```

Purpose:

```text
Review surrounding Application events for context.
```

---

# Query 05 — Run Command Correlation

```kusto
AzureActivity
| where TimeGenerated > ago(4h)
| where ResourceGroup =~ "RG-AZMON-SUPPORT-LAB"
| where OperationNameValue =~ "MICROSOFT.COMPUTE/VIRTUALMACHINES/RUNCOMMAND/ACTION"
| where TimeGenerated > datetime(2026-09-07 03:00:00)
| project TimeGenerated,
          OperationNameValue,
          ActivityStatusValue,
          ResourceGroup
| order by TimeGenerated asc
```

Purpose:

```text
Correlate the guest event with the Azure administrative operation.
```

---

# Query 06 — Recent Windows Errors

```kusto
Event
| where TimeGenerated > ago(24h)
| where Computer =~ "AZMON-WIN01"
| where EventLevelName =~ "Error"
| project TimeGenerated,
          Computer,
          EventLog,
          Source,
          EventID
| order by TimeGenerated desc
```

Purpose:

```text
Find recent Error-level Windows events.
```

---

# Query 07 — Heartbeat Health

```kusto
Heartbeat
| where Computer =~ "AZMON-WIN01"
| where TimeGenerated > ago(15m)
| summarize LastHeartbeat=max(TimeGenerated),
            HeartbeatCount=count()
| extend MinutesSinceHeartbeat=datetime_diff("minute", now(), LastHeartbeat)
| project LastHeartbeat,
          MinutesSinceHeartbeat,
          HeartbeatCount
```

Purpose:

```text
Determine whether the event coincided with monitoring or VM availability loss.
```

---

# Potential Event-Based Alert

A future scheduled query alert could detect application errors.

Example:

```text
ALRT-AZMON-APPLICATION-ERROR
```

Possible KQL:

```kusto
Event
| where TimeGenerated > ago(5m)
| where EventLevelName =~ "Error"
| summarize ErrorCount=count() by Computer, Source
| where ErrorCount > 0
```

---

# Alert Design Considerations

Production event alerts should avoid generating excessive noise.

Administrators should consider:

```text
Specific Event IDs
Specific applications
Known benign errors
Repeated event thresholds
Severity
Business impact
Time window
Suppression
Maintenance windows
```

Alerting on every Error event may be too noisy.

---

# Example Targeted Alert

A more specific alert could monitor:

```text
Source:
CriticalBusinessApp

Event ID:
5001
```

instead of all Windows errors.

---

# Evidence Directory

Incident evidence is stored in:

```text
Screenshots/09-Windows-Events/
```

---

# Evidence 01

```text
01-AZMON-WIN01-Windows-Event-Baseline.png
```

Demonstrates:

```text
Event table active
AZMON-WIN01 generating recent Windows events
Event ingestion baseline established
```

---

# Evidence 02

```text
02-AZMON-WIN01-Windows-Event-Summary.png
```

Demonstrates:

```text
Application and System logs collected
Warning-level events ingested
Multiple Windows sources visible
```

---

# Evidence 03

```text
03-AZMON-WIN01-Controlled-Application-Error-Generated.png
```

Demonstrates:

```text
Controlled Windows Error created
Application log
AZMON-AppLab source
Event ID 200
Error severity
Local validation successful
```

---

# Evidence 04

```text
04-AZMON-WIN01-Controlled-Application-Error-Ingested.png
```

Demonstrates:

```text
Event ID 200 reached Log Analytics
Application Event Log
Error severity
AZMON-AppLab source
```

---

# Evidence 05

```text
05-AZMON-WIN01-Run-Command-AzureActivity.png
```

Demonstrates:

```text
Azure Run Command Start
Azure Run Command Accept
Azure Run Command Success
```

surrounding the controlled Windows event.

---

# Evidence 06

```text
06-AZMON-WIN01-Application-Event-Timeline.png
```

Demonstrates:

```text
Controlled Error 200 in recent Application timeline
Nearby Windows Application events
Chronological event context
```

---

# Evidence 07

```text
07-AZMON-WIN01-Heartbeat-Healthy-Post-Event.png
```

Demonstrates:

```text
Heartbeat remained current
MinutesSinceHeartbeat = 1
Monitoring remained operational
```

---

# Evidence Progression

The evidence progression is:

```text
01 — Windows Event Pipeline Healthy
      ↓
02 — Existing Event Sources Identified
      ↓
03 — Controlled Error Generated Locally
      ↓
04 — Controlled Error Reaches Log Analytics
      ↓
05 — Azure Run Command Correlated
      ↓
06 — Application Timeline Reviewed
      ↓
07 — VM Monitoring Health Verified
```

---

# Resolution Criteria

INC-004 was considered resolved after:

```text
Controlled Error successfully generated
Local event validated
Event ingested into Log Analytics
Event ID confirmed
Source confirmed
Severity confirmed
Azure Run Command correlation completed
Application timeline reviewed
Heartbeat remained healthy
```

All criteria were met.

---

# Root Cause Analysis

## Symptom

```text
Windows Application Error detected on AZMON-WIN01
```

---

## Evidence

```text
Application Event Log
Error
AZMON-AppLab
Event ID 200
```

---

## Root Cause

```text
Controlled event generated through Azure VM Run Command
```

---

## Infrastructure Impact

```text
None
```

---

## Monitoring Impact

```text
None
```

---

## Resolution

```text
Investigation completed
Event pipeline validated
No remediation required
```

---

# Incident Timeline

```text
Windows Event Pipeline Baseline:
VERIFIED

Controlled Event Generation:
STARTED

03:02:58 UTC:
Run Command Start

03:02:58 UTC:
Run Command Accept

03:03:09 UTC:
Application Error Event ID 200 generated

03:03:09 UTC:
Event locally validated

Shortly afterward:
Event ingested into Log Analytics

03:03:28 UTC:
Run Command Success

Application timeline:
REVIEWED

03:06:16 UTC:
Fresh Heartbeat verified

Incident:
RESOLVED
```

---

# Lessons Learned

## Lesson 1 — Validate the Event Locally First

Local validation separates:

```text
Event generation failure
```

from:

```text
Monitoring ingestion failure
```

---

## Lesson 2 — Use Unique Sources and Event IDs

A unique source and Event ID make controlled testing easy to identify.

---

## Lesson 3 — Guest and Control-Plane Logs Complement Each Other

The Windows Event identified what occurred inside the guest.

AzureActivity identified the Azure operation associated with it.

---

## Lesson 4 — An Application Error Does Not Automatically Mean Outage

Heartbeat remained healthy after the event.

The VM and monitoring pipeline continued functioning.

---

## Lesson 5 — Context Matters

A single Error event may not indicate a major incident.

Administrators should examine:

```text
Frequency
Source
Event ID
Timeline
Application health
VM health
Performance
Related Azure operations
```

---

## Lesson 6 — Centralized Logs Simplify Investigation

Log Analytics allowed Windows events to be investigated without manually connecting to the server.

---

## Lesson 7 — Different Telemetry Sources May Arrive at Different Times

The Windows Event record became visible before the matching AzureActivity record.

Ingestion latency should be considered during investigations.

---

# Skills Demonstrated

INC-004 demonstrates:

- Microsoft Azure
- Azure Monitor
- Log Analytics
- Azure Monitor Agent
- Data Collection Rules
- Windows Server
- Windows Event Logs
- Application Event Log
- Event ID analysis
- Event source analysis
- Windows Error monitoring
- Azure VM Run Command
- PowerShell
- Get-WinEvent
- eventcreate.exe
- Kusto Query Language
- Event table querying
- Azure Activity Log
- AzureActivity
- Control-plane correlation
- Guest telemetry investigation
- Timeline analysis
- Root-cause analysis
- Health validation
- Incident documentation
- Cloud support troubleshooting

---

# Final Validation

```text
Windows Event Baseline:
VERIFIED

Event Source Summary:
VERIFIED

Controlled Application Error:
GENERATED

Local Windows Validation:
VERIFIED

Event ID 200:
VERIFIED

Source AZMON-AppLab:
VERIFIED

Error Severity:
VERIFIED

Log Analytics Ingestion:
VERIFIED

Application Event Timeline:
VERIFIED

Azure Run Command Correlation:
VERIFIED

Run Command Start:
VERIFIED

Run Command Accept:
VERIFIED

Run Command Success:
VERIFIED

Heartbeat Post-Event:
HEALTHY

VM Availability Impact:
NONE

Monitoring Impact:
NONE

Root Cause:
VERIFIED

Incident Status:
CLOSED
```

---

# Incident Closure

```text
Incident ID:
INC-004

Incident:
Windows Application Event Investigation

Affected Resource:
AZMON-WIN01

Windows Event Log:
Application

Event Level:
Error

Event Source:
AZMON-AppLab

Event ID:
200

Event Generation:
SUCCESS

Local Event Validation:
SUCCESS

Log Analytics Ingestion:
SUCCESS

AzureActivity Correlation:
SUCCESS

Heartbeat:
HEALTHY

Infrastructure Impact:
NONE

Root Cause:
Controlled Windows Application Error generated through Azure VM Run Command

Incident Status:
CLOSED
```

INC-004 successfully demonstrates end-to-end Windows Application Event generation, collection, ingestion, KQL investigation, Azure control-plane correlation, and post-event monitoring validation.
