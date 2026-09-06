# Azure Monitor + Log Analytics Support Lab

## 01 — Environment Setup

## Overview

This project is a hands-on Azure monitoring and cloud support lab designed to simulate real-world monitoring, alerting, log analysis, and incident troubleshooting tasks performed by Cloud Support, 
Help Desk, Systems Administration, and Azure Operations teams.

The environment focuses on monitoring a Windows virtual machine and its supporting Azure resources using Azure Monitor, Log Analytics, Azure Monitor Agent, Data Collection Rules, Activity Logs, 
diagnostic settings, alerts, and Kusto Query Language (KQL).

Unlike a general Azure administration lab, this project focuses specifically on the support workflow:

```text
Resource
   ↓
Telemetry
   ↓
Azure Monitor
   ↓
Log Analytics
   ↓
Alert / User Report
   ↓
Investigation
   ↓
KQL Analysis
   ↓
Root Cause
   ↓
Remediation
   ↓
Validation
```

The lab intentionally generates controlled monitoring and availability incidents so that each issue can be investigated and documented from detection through resolution.

---

# Lab Objectives

The primary objectives of this project are to:

* Deploy a Windows Azure virtual machine for monitoring
* Configure a dedicated Log Analytics workspace
* Deploy Azure Monitor Agent
* Configure Data Collection Rules
* Collect Windows Event Logs
* Collect guest operating system performance data
* Review Azure platform metrics
* Analyze Azure Activity Logs
* Configure diagnostic settings
* Route operational data to Log Analytics
* Create Azure Monitor alert rules
* Trigger controlled monitoring alerts
* Query telemetry using KQL
* Investigate missing telemetry
* Troubleshoot VM availability problems
* Investigate high CPU conditions
* Correlate monitoring events across multiple Azure data sources
* Perform root-cause analysis
* Document incident remediation
* Validate service recovery after remediation

---

# Existing Azure Environment

This project is being built alongside an existing Azure Cloud Support Administration lab.

Existing Azure lab resources include:

```text
Resource Group:
RG-IT-Support-Lab

Region:
North Central US

Virtual Network:
VNET-IT-Support

Subnet:
SNET-Workstations
10.10.1.0/24

Network Security Group:
NSG-IT-Support
```

The existing environment remains separate from this monitoring lab so that monitoring configurations, alerts, telemetry, and troubleshooting scenarios can be modified without affecting previous 
portfolio evidence.

---

# New Monitoring Lab Environment

A dedicated resource group will be created for this project.

```text
Resource Group:
RG-AZMON-SUPPORT-LAB

Region:
North Central US
```

Keeping the monitoring resources in a dedicated resource group makes it easier to:

* Identify project resources
* Review Activity Log events
* Apply monitoring settings
* Track costs
* Remove lab resources after testing
* Separate this project from other Azure labs

---

# Resource Naming Standard

The lab uses descriptive Azure resource prefixes.

| Resource Type           | Planned Name          |
| ----------------------- | --------------------- |
| Resource Group          | RG-AZMON-SUPPORT-LAB  |
| Virtual Machine         | AZMON-WIN01           |
| Virtual Network         | VNET-AZMON-SUPPORT    |
| Subnet                  | SNET-AZMON-WORKLOAD   |
| Network Security Group  | NSG-AZMON-WIN01       |
| Log Analytics Workspace | LAW-AZMON-SUPPORT-LAB |
| Data Collection Rule    | DCR-AZMON-WINDOWS     |
| Action Group            | AG-AZMON-SUPPORT      |
| Public IP               | PIP-AZMON-WIN01       |
| Network Interface       | NIC-AZMON-WIN01       |

Alert rules will use the following naming convention:

```text
ALRT-AZMON-[CONDITION]
```

Examples:

```text
ALRT-AZMON-HIGH-CPU
ALRT-AZMON-VM-AVAILABILITY
ALRT-AZMON-ACTIVITY-LOG
ALRT-AZMON-MISSING-HEARTBEAT
ALRT-AZMON-WINDOWS-ERRORS
```

---

# Planned Architecture

```text
Microsoft Azure
│
└── RG-AZMON-SUPPORT-LAB
    │
    ├── AZMON-WIN01
    │   │
    │   ├── Windows Operating System
    │   │
    │   ├── Azure Monitor Agent
    │   │
    │   ├── Windows Event Logs
    │   │
    │   └── Guest Performance Data
    │   │
    │   ├── NIC-AZMON-WIN01
    │   └── NSG-AZMON-WIN01
    │
    ├── VNET-AZMON-SUPPORT
    │   │
    │   └── SNET-AZMON-WORKLOAD
    │
    ├── LAW-AZMON-SUPPORT-LAB
    │   │
    │   ├── Heartbeat
    │   ├── Event
    │   ├── AzureActivity
    │   └── Performance Data
    │
    ├── DCR-AZMON-WINDOWS
    │   │
    │   ├── Windows Event Logs
    │   └── Performance Counters
    │
    ├── Azure Monitor
    │   │
    │   ├── Metrics
    │   ├── Logs
    │   ├── Activity Log
    │   ├── Alerts
    │   └── VM Monitoring
    │
    └── AG-AZMON-SUPPORT
        │
        └── Alert Notifications
```

---

# Monitoring Data Sources

The project will investigate several different categories of telemetry.

## Azure Platform Metrics

Azure VM platform metrics will be reviewed for conditions such as:

* CPU utilization
* Network activity
* Disk operations
* VM availability
* VM performance trends

These metrics will provide the first layer of monitoring during several incident scenarios.

---

# Guest Operating System Telemetry

The Windows virtual machine will use Azure Monitor Agent to send guest operating system telemetry to Log Analytics.

Planned guest telemetry includes:

* Windows Application events
* Windows System events
* Performance data
* Agent heartbeat information

This allows monitoring beyond the Azure resource itself and provides visibility into events occurring inside the operating system.

---

# Azure Monitor Agent

Azure Monitor Agent will be used as the monitoring agent for:

```text
AZMON-WIN01
```

The agent will provide the connection between the Windows guest operating system and Azure Monitor data collection.

Agent communication will later be validated using the Log Analytics `Heartbeat` table.

Expected troubleshooting path:

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
```

If heartbeat records stop appearing, the monitoring pipeline can be investigated at each layer.

---

# Data Collection Rule

The planned Data Collection Rule is:

```text
DCR-AZMON-WINDOWS
```

The DCR will define which guest telemetry should be collected from the Windows VM.

Planned data sources include:

```text
Windows Event Logs
Performance Counters
```

The DCR will be associated with:

```text
AZMON-WIN01
```

The destination will be:

```text
LAW-AZMON-SUPPORT-LAB
```

---

# Log Analytics Workspace

The central log repository for this project will be:

```text
LAW-AZMON-SUPPORT-LAB
```

The workspace will be used for:

* Centralized log collection
* KQL queries
* Incident investigation
* Historical telemetry analysis
* Alert rule queries
* Windows Event analysis
* Azure Activity analysis
* Agent communication validation

The lab intentionally uses one primary workspace so that telemetry can be analyzed from a central location.

---

# Planned Log Analytics Tables

The exact tables available will depend on the data sources successfully configured during the lab.

Primary tables expected during the project include:

```text
Heartbeat
Event
AzureActivity
```

Additional performance-related tables will be documented after the final collection configuration is confirmed.

The table names and available schemas will be validated in the live workspace rather than assumed.

---

# Activity Log Monitoring

Azure Activity Log will be used to investigate control-plane operations affecting Azure resources.

Example events may include:

* Virtual machine start
* Virtual machine stop
* Virtual machine restart
* Resource configuration changes
* Resource creation
* Resource deletion
* Failed Azure operations
* Administrative changes

Activity Log events will be important during troubleshooting because they provide evidence of what occurred at the Azure resource-management layer.

---

# Diagnostic Settings

Diagnostic settings will be configured where appropriate to route supported Azure telemetry into the Log Analytics workspace.

The project will document:

1. Source resource
2. Available diagnostic categories
3. Selected categories
4. Destination workspace
5. Data validation
6. Troubleshooting if telemetry does not appear

Diagnostic settings will not be treated as complete until the expected data is verified in Log Analytics.

---

# Azure Monitor Alerts

Several alert types will be configured during this project.

Planned alerts include:

```text
High CPU Alert
VM Availability Alert
Activity Log Alert
Missing Heartbeat / Log Alert
Windows Error Event Alert
```

The alert workflow will be documented as:

```text
Condition Occurs
      ↓
Azure Monitor Detects Condition
      ↓
Alert Rule Evaluates
      ↓
Alert Fires
      ↓
Investigation Begins
      ↓
Root Cause Identified
      ↓
Remediation
      ↓
Condition Returns to Normal
      ↓
Recovery Validated
```

---

# Action Group

The planned action group is:

```text
AG-AZMON-SUPPORT
```

The action group will provide a centralized notification target for Azure Monitor alerts used in the lab.

Configuration details will be documented when the action group is created.

---

# KQL

Kusto Query Language will be used throughout the project instead of being treated as a separate demonstration.

Queries will support actual troubleshooting scenarios.

Planned query categories include:

```text
Heartbeat Queries
Windows Event Queries
Activity Log Queries
Performance Queries
Incident Investigation Queries
```

The repository contains dedicated KQL files:

```text
KQL/Heartbeat-Queries.kql
KQL/Performance-Queries.kql
KQL/Windows-Event-Queries.kql
KQL/Activity-Log-Queries.kql
KQL/Incident-Investigation-Queries.kql
```

Each query will include context explaining what problem it helps investigate.

---

# KQL Skills to Practice

The project will practice operators and functions including:

```text
where
project
summarize
count
distinct
sort
order by
top
ago()
bin()
extend
join
```

Queries will progress from basic data validation to incident correlation.

---

# Planned Incident Scenarios

Eight troubleshooting incidents will be documented.

## INC-001 — High CPU

A controlled CPU workload will be generated.

Investigation will include:

* CPU metrics
* alert status
* incident time window
* VM investigation
* remediation
* post-remediation metric validation

---

## INC-002 — Missing Heartbeat

Monitoring telemetry will intentionally be interrupted or simulated as missing.

Investigation will focus on:

```text
VM State
      ↓
Azure Monitor Agent
      ↓
DCR Association
      ↓
Workspace Destination
      ↓
Heartbeat Query
```

---

## INC-003 — VM Availability

The virtual machine will be intentionally stopped or made unavailable.

The incident will correlate:

* Azure Monitor
* VM availability
* Activity Log
* alerts
* recovery timestamps

---

## INC-004 — Windows Event Failure

A controlled Windows event or error condition will be generated or identified.

The event will then be:

1. Collected
2. Sent through the DCR
3. Stored in Log Analytics
4. Located with KQL
5. Investigated
6. Documented

---

## INC-005 — Failed Azure Operation

A failed or controlled Azure administrative operation will be investigated.

Evidence will include:

* Activity Log
* operation status
* timestamp
* affected resource
* error details
* root cause
* corrective action

---

## INC-006 — Metric Alert

A metric threshold will be exceeded.

The project will demonstrate:

```text
Metric
↓
Threshold
↓
Alert Rule
↓
Triggered Alert
↓
Investigation
↓
Resolution
```

---

## INC-007 — Log Alert

A KQL-based log search alert will be created.

The scenario will demonstrate how collected log data can be transformed into an operational alert.

---

## INC-008 — Diagnostic Data Missing

A telemetry collection problem will be investigated.

Possible troubleshooting areas include:

* Diagnostic settings
* DCR configuration
* DCR association
* Azure Monitor Agent
* workspace destination
* query time range
* missing expected records

This scenario is designed to demonstrate troubleshooting of the monitoring platform itself.

---

# Incident Documentation Standard

Each incident ticket will include:

```text
Incident ID
Title
Severity
Affected Resource
Date / Time
User or Monitoring Report
Symptoms
Initial Assessment
Troubleshooting
Evidence
KQL Queries
Root Cause
Resolution
Validation
Preventive Actions
Skills Demonstrated
```

---

# Root-Cause Investigation Model

All incidents will use the following investigation model where applicable.

## 1. Detect

Determine how the problem was discovered.

Examples:

* Azure Monitor alert
* Metric threshold
* User report
* Missing telemetry
* Activity Log event

## 2. Scope

Identify:

* affected resource
* time window
* severity
* user impact
* monitoring impact

## 3. Collect Evidence

Review:

* metrics
* logs
* Activity Log
* Windows events
* alerts
* resource configuration

## 4. Analyze

Use:

* Azure Monitor
* Log Analytics
* KQL
* VM operating system tools

## 5. Identify Root Cause

Determine the specific cause supported by available evidence.

## 6. Remediate

Correct the condition.

## 7. Validate

Verify:

* service recovery
* normal metrics
* restored telemetry
* successful heartbeat
* cleared/resolved alert condition

## 8. Document

Record the investigation and resolution in the repository.

---

# Screenshot Strategy

Screenshots are organized by technical area.

```text
Screenshots/
├── 01-Environment/
├── 02-Log-Analytics/
├── 03-Agent-DCR/
├── 04-VM-Metrics/
├── 05-Activity-Logs/
├── 06-Alerts/
├── 07-KQL/
├── 08-Incidents/
└── 09-Root-Cause/
```

Screenshots will focus on evidence rather than every portal click.

Priority evidence includes:

* Resource group
* VM overview
* Log Analytics workspace
* Azure Monitor Agent
* DCR configuration
* DCR association
* Heartbeat query
* Windows Event query
* AzureActivity query
* VM metric baseline
* High CPU condition
* Alert rule configuration
* Triggered alert
* Activity Log evidence
* KQL investigation
* Root cause evidence
* Remediation
* Post-remediation validation

---

# Screenshot Naming Standard

Screenshot filenames will use a numbered descriptive format.

Example:

```text
01-RG-AZMON-Support-Lab.png
02-AZMON-WIN01-Overview.png
03-Log-Analytics-Workspace.png
04-Azure-Monitor-Agent.png
05-DCR-Association.png
06-Heartbeat-Query.png
```

Incident screenshots will include the incident identifier where appropriate.

Example:

```text
INC-001-High-CPU-Alert.png
INC-001-CPU-Investigation.png
INC-001-Post-Remediation.png
```

---

# Security Considerations

Screenshots and documentation will be reviewed before being published to GitHub.

The repository will avoid exposing:

* passwords
* authentication secrets
* access tokens
* subscription IDs where unnecessary
* sensitive account information
* public IP information where unnecessary
* private credentials

Administrative access will follow the minimum permissions required to complete the lab.

---

# Cost Management

This project is designed as a temporary support lab.

Cost controls include:

* Using only required Azure resources
* Avoiding unnecessary duplicate resources
* Using one primary Log Analytics workspace
* Keeping telemetry collection focused
* Shutting down the VM when it is not needed
* Removing temporary resources when testing is complete
* Reviewing Azure cost information during the project

Monitoring data collection will be intentionally limited to telemetry required for the support scenarios.

---

# Environment Validation Checklist

The environment will be considered ready for monitoring exercises after the following items are validated:

* [ ] `RG-AZMON-SUPPORT-LAB` created
* [ ] Region set to North Central US
* [ ] `VNET-AZMON-SUPPORT` created
* [ ] `SNET-AZMON-WORKLOAD` created
* [ ] `AZMON-WIN01` deployed
* [ ] VM starts successfully
* [ ] Administrative access confirmed
* [ ] `LAW-AZMON-SUPPORT-LAB` created
* [ ] Azure Monitor Agent deployed
* [ ] `DCR-AZMON-WINDOWS` created
* [ ] DCR associated with AZMON-WIN01
* [ ] Windows Event collection configured
* [ ] Performance data collection configured
* [ ] Heartbeat records visible
* [ ] Event records visible
* [ ] Azure platform metrics visible
* [ ] Activity Log accessible
* [ ] Diagnostic settings reviewed
* [ ] Action group created
* [ ] Initial alert rule created
* [ ] KQL queries successfully executed

---

# Expected Skills Demonstrated

Upon completion, this lab will demonstrate practical experience with:

* Microsoft Azure
* Azure Monitor
* Log Analytics
* Azure Monitor Agent
* Data Collection Rules
* Azure Virtual Machines
* Windows administration
* Azure Activity Log
* Diagnostic settings
* Azure Monitor alerts
* Action groups
* Metrics analysis
* Log analysis
* Kusto Query Language
* Monitoring architecture
* Cloud troubleshooting
* Incident investigation
* Root-cause analysis
* Technical documentation
* Service restoration validation

---

# Environment Status

Current project status:

```text
Repository structure: Complete
Git repository: Initialized
Environment documentation: In Progress
Azure resource deployment: Not Started
Monitoring configuration: Not Started
KQL validation: Not Started
Incident simulations: Not Started
README: Not Started
```

The next phase is creation of the dedicated Azure resource group and Log Analytics workspace.

