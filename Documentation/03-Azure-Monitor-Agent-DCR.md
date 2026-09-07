# Azure Monitor + Log Analytics Support Lab

## 03 — Azure Monitor Agent and Data Collection Rules

## Overview

This phase of the project establishes guest operating system monitoring for the Windows Server virtual machine `AZMON-WIN01`.

Azure Monitor Agent was deployed to the virtual machine through a Data Collection Rule association. The Data Collection Rule defines which Windows guest telemetry is collected and where that 
telemetry is delivered.

The monitoring pipeline implemented in this phase is:

```text
AZMON-WIN01
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Heartbeat
Perf
Event
```

The completed configuration provides three important types of monitoring evidence:

- Agent connectivity
- Guest operating system performance telemetry
- Windows Event Log telemetry

This phase also validates the complete data path using Kusto Query Language queries in Log Analytics.

---

## Objectives

The objectives of this phase were to:

- Create a Windows Data Collection Rule
- Associate the rule with AZMON-WIN01
- Deploy Azure Monitor Agent
- Configure performance counter collection
- Configure Windows Event Log collection
- Route guest telemetry to Log Analytics
- Validate Azure Monitor Agent health
- Validate heartbeat data
- Validate performance telemetry
- Validate Windows event ingestion
- Generate a controlled Windows warning event
- Verify the warning reaches Log Analytics
- Store reusable KQL queries in the GitHub repository

---

## Environment

### Virtual Machine

```text
Name:
AZMON-WIN01

Operating System:
Windows Server 2022 Datacenter: Azure Edition

VM Size:
Standard_D2als_v6

vCPUs:
2

Memory:
4 GiB

Region:
North Central US
```

### Resource Group

```text
RG-AZMON-SUPPORT-LAB
```

### Log Analytics Workspace

```text
LAW-AZMON-SUPPORT-LAB
```

### Data Collection Rule

```text
DCR-AZMON-WINDOWS
```

### Monitoring Agent

```text
AzureMonitorWindowsAgent
```

---

## Pre-Configuration Baseline

Before creating the Data Collection Rule, the Azure Monitor Data Collection Rules page showed no configured DCRs for this lab.

The baseline state confirmed:

```text
Data Collection Rules: 0
Destinations: 0
Resources Collecting: 0
```

This provided clear before-and-after evidence for the monitoring configuration.

---

## Data Collection Rule Creation

A new Data Collection Rule was created with the following configuration:

```text
Rule Name:
DCR-AZMON-WINDOWS

Resource Group:
RG-AZMON-SUPPORT-LAB

Region:
North Central US

Telemetry Type:
Agent-based - Windows
```

The rule was created specifically for Windows guest telemetry.

A Data Collection Endpoint was not configured because this lab uses standard Azure Monitor connectivity and does not use Azure Monitor Private Link.

A user-assigned managed identity was also not required for this configuration.

---

## Resource Association

The Data Collection Rule was associated with:

```text
AZMON-WIN01
```

The VM was intentionally selected as the only monitored resource for this DCR.

This creates a direct relationship:

```text
DCR-AZMON-WINDOWS
      ↓
AZMON-WIN01
```

The association allows Azure Monitor Agent to receive the configuration defined by the DCR.

---

## Performance Counter Collection

Performance counters were configured to provide guest operating system performance visibility.

The collection was intentionally limited to counters useful for troubleshooting.

Sampling interval:

```text
60 seconds
```

---

## CPU Counter

Selected CPU counter:

```text
\Processor Information(_Total)\% Processor Time
```

Purpose:

- Measure processor utilization
- Compare guest CPU telemetry with Azure platform CPU metrics
- Investigate sustained CPU utilization
- Support high CPU incident analysis

---

## Memory Counter

Selected memory counter:

```text
\Memory\Available Bytes
```

Purpose:

- Monitor available system memory
- Detect memory pressure
- Support performance troubleshooting
- Identify low-memory conditions

---

## Disk Counters

Selected disk counters:

```text
\LogicalDisk(_Total)\Avg. Disk Queue Length
```

and:

```text
\LogicalDisk(_Total)\Free Megabytes
```

Purpose:

- Monitor disk queue pressure
- Review available disk capacity
- Support storage-related performance investigations
- Identify abnormal disk activity

---

## Network Counter

Selected network counter:

```text
\Network Interface(*)\Bytes Total/sec
```

Purpose:

- Monitor guest network throughput
- Identify unusual network traffic
- Compare network telemetry with Azure host metrics
- Support connectivity and performance troubleshooting

---

## Performance Counter Summary

Configured counters:

```text
CPU:
1 selected

Memory:
1 selected

Disk:
2 selected

Network:
1 selected
```

All counters were configured with a:

```text
60-second sampling interval
```

This provides useful monitoring coverage while avoiding unnecessary telemetry collection.

---

## Performance Destination

The performance counter destination was configured as:

```text
Destination Type:
Log Analytics Workspace
```

Workspace:

```text
LAW-AZMON-SUPPORT-LAB
```

Destination table:

```text
Perf
```

No transformation was applied.

Monitoring path:

```text
Windows Performance Counters
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Perf
```

---

## Windows Event Log Collection

A second data source was created for Windows Event Logs.

The event collection was intentionally limited to troubleshooting-relevant severity levels.

---

## Application Log

Selected event levels:

```text
Critical
Error
Warning
```

Not selected:

```text
Information
Verbose
```

---

## System Log

Selected event levels:

```text
Critical
Error
Warning
```

Not selected:

```text
Information
Verbose
```

---

## Security Log

Security audit events were not enabled for this phase.

Not selected:

```text
Audit Success
Audit Failure
```

The goal of this phase was operational troubleshooting rather than security event monitoring.

---

## Windows Event Destination

The Windows Event Log destination was configured as:

```text
Destination Type:
Log Analytics Workspace
```

Workspace:

```text
LAW-AZMON-SUPPORT-LAB
```

Destination table:

```text
Event
```

No transformation was applied.

Monitoring path:

```text
Windows Event Logs
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Event
```

---

## Data Collection Summary

After configuration, the DCR contained two data sources:

```text
Performance Counters
Windows Event Logs
```

Both were routed to:

```text
LAW-AZMON-SUPPORT-LAB
```

The completed collection model became:

```text
AZMON-WIN01
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ├── Performance Counters
      │       ↓
      │      Perf
      │
      └── Windows Event Logs
              ↓
             Event
```

---

## Data Collection Rule Deployment

The completed DCR configuration was reviewed before deployment.

Final configuration included:

```text
Rule:
DCR-AZMON-WINDOWS

Resource:
AZMON-WIN01

Data Source 1:
Performance Counters

Data Source 2:
Windows Event Logs

Destination:
LAW-AZMON-SUPPORT-LAB
```

The rule was successfully created.

After deployment, the DCR Overview showed:

```text
Data Sources:
2

Connected Resources:
1

Platform Type:
Windows
```

This confirmed that the monitoring configuration was active.

---

## Azure Monitor Agent Deployment

After the Data Collection Rule was associated with AZMON-WIN01, Azure Monitor Agent was installed on the VM.

Extension:

```text
AzureMonitorWindowsAgent
```

Observed version:

```text
1.45.0.0
```

Provisioning status:

```text
Provisioning succeeded
```

Handler status:

```text
Ready
```

Status message indicated successful extension enablement.

This confirmed that Azure Monitor Agent was installed and operational.

---

## Agent Deployment Workflow

```text
DCR Created
      ↓
AZMON-WIN01 Associated
      ↓
Azure Monitor Agent Installed
      ↓
Agent Provisioning Succeeded
      ↓
DCR Configuration Applied
      ↓
Guest Telemetry Collected
```

---

## Log Analytics Validation

After Azure Monitor Agent deployment, telemetry was validated in:

```text
LAW-AZMON-SUPPORT-LAB
```

Three telemetry types were tested:

```text
Heartbeat
Perf
Event
```

---

## Heartbeat Validation

The following KQL query was used:

```kusto
Heartbeat
| where TimeGenerated > ago(30m)
| project TimeGenerated, Computer, OSType
| order by TimeGenerated desc
```

Results returned records for:

```text
AZMON-WIN01
```

This confirmed:

- Azure Monitor Agent was communicating
- The VM was successfully connected to Azure Monitor
- Log Analytics was receiving agent heartbeat data

Heartbeat status:

```text
VERIFIED
```

---

## Performance Data Validation

The following query was used:

```kusto
Perf
| where TimeGenerated > ago(30m)
| project TimeGenerated, Computer, ObjectName, CounterName, InstanceName, CounterValue
| order by TimeGenerated desc
```

The query returned guest performance telemetry from:

```text
AZMON-WIN01
```

Observed data included configured categories for:

- CPU
- Memory
- Disk
- Network

This confirmed that the performance counter path was operational.

Performance telemetry status:

```text
VERIFIED
```

---

## Initial Event Collection Baseline

The Windows Event query was initially executed with:

```kusto
Event
| where TimeGenerated > ago(30m)
| project TimeGenerated, Computer, EventLog, EventLevelName, Source, EventID
| order by TimeGenerated desc
```

The initial query returned no matching records.

This was not treated as a monitoring failure.

Because the DCR only collects:

```text
Critical
Error
Warning
```

the VM may not have generated a qualifying Windows event during the initial collection window.

A controlled warning event was therefore generated for validation.

---

## Controlled Windows Event Test

A controlled Windows warning was generated on AZMON-WIN01 through Azure VM Run Command.

The event used:

```text
Event Log:
Application

Event Type:
Warning

Event ID:
100

Source:
AZMON-Lab
```

Description:

```text
Controlled warning generated for Azure Monitor Log Analytics validation.
```

---

## Controlled Event Generation Script

The Windows VM executed:

```powershell
Write-Output "=== AZURE MONITOR EVENT TEST ==="

eventcreate.exe /T WARNING /ID 100 /L APPLICATION /SO "AZMON-Lab" /D "Controlled warning generated for Azure Monitor Log Analytics validation."

Write-Output ""
Write-Output "=== WINDOWS EVENT VALIDATION ==="

Get-WinEvent -FilterHashtable @{
    LogName   = 'Application'
    Id        = 100
    StartTime = (Get-Date).AddMinutes(-15)
} |
Select-Object -First 1 TimeCreated, Id, LevelDisplayName, ProviderName |
Format-List
```

Local Windows validation returned:

```text
ID:
100

Level:
Warning

Provider:
AZMON-Lab
```

This confirmed that the controlled event was successfully created inside the Windows guest operating system.

---

## Windows Event Ingestion Validation

After allowing time for telemetry ingestion, the following query was executed:

```kusto
Event
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where EventID == 100
| project TimeGenerated, Computer, EventLog, EventLevelName, Source, EventID
| order by TimeGenerated desc
```

The query returned the controlled event.

Observed values included:

```text
Computer:
AZMON-WIN01

EventLog:
Application

EventLevelName:
Warning

Source:
AZMON-Lab

EventID:
100
```

This proved the complete telemetry path:

```text
Windows Server
      ↓
Application Event Log
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Event Table
      ↓
KQL Query
```

Windows Event collection status:

```text
VERIFIED
```

---

## End-to-End Monitoring Validation

At the conclusion of this phase, the monitoring pipeline status was:

```text
Azure Monitor Agent ........ VERIFIED
DCR Resource Association ... VERIFIED
Heartbeat .................. VERIFIED
Performance Counters ....... VERIFIED
Windows Event Collection ... VERIFIED
Controlled Event ID 100 .... VERIFIED
Log Analytics Ingestion .... VERIFIED
```

This established a known-good guest telemetry baseline for future troubleshooting scenarios.

---

## Comparison of Telemetry Types

### Heartbeat

Purpose:

```text
Agent connectivity and monitoring availability
```

Table:

```text
Heartbeat
```

---

### Performance Counters

Purpose:

```text
Guest operating system performance monitoring
```

Table:

```text
Perf
```

---

### Windows Event Logs

Purpose:

```text
Guest operating system event investigation
```

Table:

```text
Event
```

---

## Troubleshooting Model

If guest telemetry stops appearing, the following sequence can be investigated:

```text
AZMON-WIN01 Running?
      ↓
Azure Monitor Agent Installed?
      ↓
Extension Healthy?
      ↓
DCR Association Present?
      ↓
Correct Data Sources Configured?
      ↓
Correct Workspace Destination?
      ↓
Expected Table Exists?
      ↓
Query Time Range Correct?
```

This model will later be used during missing-telemetry incidents.

---

## Agent Troubleshooting

Potential Azure Monitor Agent issues include:

- Extension provisioning failure
- Agent service failure
- DCR association removed
- Network connectivity issue
- Invalid DCR configuration
- Incorrect workspace destination
- Resource region mismatch
- Data source configuration issue

A healthy extension should show:

```text
Provisioning succeeded
```

and:

```text
Handler Status:
Ready
```

---

## Heartbeat Troubleshooting

If heartbeat records are missing:

```kusto
Heartbeat
| where TimeGenerated > ago(1h)
| summarize LastHeartbeat=max(TimeGenerated) by Computer
| order by LastHeartbeat desc
```

This helps identify the last time each monitored system communicated with Azure Monitor.

---

## Performance Troubleshooting

Example CPU investigation:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where CounterName contains "% Processor Time"
| project TimeGenerated, Computer, ObjectName, CounterName, CounterValue
| order by TimeGenerated desc
```

This query is used during CPU-related incidents.

---

## Windows Event Troubleshooting

Example event investigation:

```kusto
Event
| where TimeGenerated > ago(1h)
| where Computer =~ "AZMON-WIN01"
| project TimeGenerated, EventLog, EventLevelName, Source, EventID
| order by TimeGenerated desc
```

A more targeted query can filter by:

- Event ID
- Severity
- Source
- Event Log
- Time range

---

## Repository KQL Files

Queries are stored in:

```text
KQL/Heartbeat-Queries.kql
KQL/Performance-Queries.kql
KQL/Windows-Event-Queries.kql
```

These files provide reusable troubleshooting queries outside the Azure portal.

---

## Screenshot Evidence

Evidence for this phase is stored in:

```text
Screenshots/03-Agent-DCR/
```

Screenshot sequence:

```text
01-DCR-Baseline-No-Collection.png
02-DCR-AZMON-WINDOWS-Basics.png
03-DCR-AZMON-WINDOWS-Resource-Association.png
04-DCR-Performance-Counters.png
04A-DCR-Performance-CPU-Memory-Counters.png
04B-DCR-Performance-Disk-Network-Counters.png
05-DCR-Performance-Log-Analytics-Destination.png
05A-DCR-Performance-Perf-Table.png
06-DCR-Windows-Event-Logs.png
07-DCR-Windows-Event-Log-Destination.png
07A-DCR-Windows-Event-Event-Table.png
08-DCR-Collect-And-Deliver-Summary.png
09-DCR-AZMON-WINDOWS-Review-Create.png
10-DCR-AZMON-WINDOWS-Deployment-Complete.png
11-DCR-AZMON-WINDOWS-Overview.png
12-AZMON-WIN01-Azure-Monitor-Agent-Installed.png
12A-AZMON-WIN01-Azure-Monitor-Agent-Status.png
13-AZMON-WIN01-Heartbeat-Verified.png
14-AZMON-WIN01-Performance-Data-Verified.png
15-AZMON-WIN01-Event-Collection-Baseline.png
15A-AZMON-WIN01-Controlled-Warning-Generated.png
15B-AZMON-WIN01-Windows-Event-Ingestion-Verified.png
```

This sequence documents the complete process from an empty DCR environment through successful telemetry ingestion.

---

## Security and Privacy Considerations

Screenshots are reviewed before publication.

Unnecessary information is cropped or redacted where appropriate, including:

- Subscription IDs
- Public IP addresses
- User account identities
- Browser URLs containing Azure resource identifiers
- Workspace identifiers where unnecessary
- VM resource IDs where unnecessary

Lab resource names remain visible because they provide useful architectural evidence.

---

## Cost Considerations

Monitoring collection was intentionally limited to useful troubleshooting telemetry.

Performance counters were limited to five selected counters at 60-second intervals.

Windows Event Logs were limited to:

```text
Critical
Error
Warning
```

for:

```text
Application
System
```

This reduces unnecessary log ingestion compared with collecting all Windows events and every available performance counter.

---

## Lessons Learned

This phase demonstrated that creating a Log Analytics workspace alone does not provide guest operating system telemetry.

A complete guest monitoring pipeline requires:

```text
Monitored Resource
+
Azure Monitor Agent
+
Data Collection Rule
+
Data Source Configuration
+
Destination
+
Log Analytics Validation
```

The controlled Windows warning also demonstrated an important troubleshooting principle.

An empty query does not automatically mean the monitoring pipeline is broken.

The initial Event query returned zero results because no qualifying Warning, Error, or Critical event had been generated during the collection period.

By creating a known event and validating it locally before searching Log Analytics, the data collection pipeline could be tested methodically.

---

## Skills Demonstrated

This phase demonstrates:

- Azure Monitor
- Azure Monitor Agent
- Azure Virtual Machines
- Data Collection Rules
- Log Analytics
- Kusto Query Language
- Windows Server
- Performance counters
- Windows Event Logs
- Azure VM Run Command
- Telemetry validation
- Agent troubleshooting
- Monitoring architecture
- Controlled test generation
- Cloud troubleshooting
- Technical documentation
- GitHub portfolio evidence management

---

## Phase Status

```text
Data Collection Rule: COMPLETE
VM Resource Association: COMPLETE
Azure Monitor Agent: VERIFIED
Performance Counter Collection: VERIFIED
Windows Event Collection: VERIFIED
Heartbeat Validation: VERIFIED
Perf Table Validation: VERIFIED
Event Table Validation: VERIFIED
Controlled Event Test: COMPLETE
KQL Repository Queries: COMPLETE
```

This phase is complete and provides a validated guest operating system monitoring foundation for the remaining Azure Monitor troubleshooting scenarios.
