# Azure Monitor + Log Analytics Support Lab

## 02 — Log Analytics Workspace

## Overview

This phase of the project establishes the centralized Log Analytics workspace that will be used throughout the Azure monitoring and troubleshooting lab.

The workspace provides a central location for collecting, storing, querying, and correlating operational telemetry from Azure resources and monitored operating systems.

The workspace created for this project is:

```text
LAW-AZMON-SUPPORT-LAB
```

It resides inside:

```text
RG-AZMON-SUPPORT-LAB
```

Region:

```text
North Central US
```

---

# Objective

The objectives of this phase were to:

* Create a dedicated Log Analytics workspace
* Place the workspace inside the monitoring lab resource group
* Verify workspace deployment
* Verify workspace operational health
* Confirm access to the Logs interface
* Execute an initial KQL validation query
* Establish the workspace as the central telemetry destination for later monitoring phases

---

# Resource Group

The monitoring lab uses the following dedicated resource group:

```text
RG-AZMON-SUPPORT-LAB
```

The resource group was created in:

```text
North Central US
```

Using a dedicated resource group separates this monitoring environment from other Azure portfolio projects.

This provides clearer:

* Resource organization
* Activity Log investigation
* Cost tracking
* Resource cleanup
* Troubleshooting scope
* Portfolio documentation

---

# Log Analytics Workspace

The Log Analytics workspace created for this project is:

```text
LAW-AZMON-SUPPORT-LAB
```

Configuration:

```text
Resource Group:
RG-AZMON-SUPPORT-LAB

Region:
North Central US

Workspace:
LAW-AZMON-SUPPORT-LAB

Status:
Active

Pricing Tier:
Pay-as-you-go
```

The Azure portal reported no operational issues after deployment.

---

# Role of the Workspace

The workspace will function as the central log repository for this project.

Planned telemetry includes:

```text
Azure Monitor Agent
        ↓
Data Collection Rule
        ↓
Windows Event Logs
        ↓
LAW-AZMON-SUPPORT-LAB
```

and:

```text
Azure Resources
        ↓
Diagnostic Settings
        ↓
LAW-AZMON-SUPPORT-LAB
```

The workspace will later support investigation of:

* Agent heartbeat
* Windows events
* Guest operating system telemetry
* Azure Activity Log events
* Monitoring alerts
* Performance conditions
* Missing telemetry
* Failed administrative operations

---

# Planned Data Sources

The workspace currently contains no production workload telemetry because the monitored VM and Data Collection Rule have not yet been deployed.

Planned data sources include:

## Azure Monitor Agent

Azure Monitor Agent will be deployed to:

```text
AZMON-WIN01
```

The agent will send guest operating system telemetry through a Data Collection Rule.

---

## Windows Event Logs

Selected Windows Event Logs will be collected from the VM.

Planned categories include:

```text
System
Application
```

The exact event filters will be documented when the Data Collection Rule is created.

---

## Performance Data

Guest performance information will be collected to support troubleshooting of:

* CPU conditions
* Memory conditions
* Disk activity
* Performance trends

The final tables and counters will be documented after the Data Collection Rule is deployed and validated.

---

## Azure Activity Data

Azure administrative operations will later be analyzed to identify events such as:

* VM startup
* VM shutdown
* VM restart
* Configuration changes
* Resource creation
* Resource deletion
* Failed Azure operations

---

# Initial Workspace Validation

Before connecting telemetry sources, the Log Analytics query interface was tested.

The following KQL query was executed:

```kusto
print
    Lab = "Azure Monitor + Log Analytics Support Lab",
    Workspace = "LAW-AZMON-SUPPORT-LAB",
    Validation = "KQL query engine operational"
```

The query successfully returned one result row containing:

```text
Lab:
Azure Monitor + Log Analytics Support Lab

Workspace:
LAW-AZMON-SUPPORT-LAB

Validation:
KQL query engine operational
```

This confirmed that:

* The workspace was accessible
* The Logs interface loaded correctly
* KQL mode was available
* Queries could execute successfully
* Results could be returned from the Log Analytics query engine

---

# Why the `print` Query Was Used

At this stage, no monitored VM or telemetry source had been connected.

Queries against tables such as:

```text
Heartbeat
Event
AzureActivity
```

might therefore return no data or indicate that the table has not yet received records.

The `print` operator provided a clean method of validating the query engine independently of telemetry ingestion.

This separated two different troubleshooting questions:

```text
Can the query engine execute KQL?
```

from:

```text
Has monitoring data been ingested?
```

The first question has now been validated successfully.

Telemetry ingestion will be validated later.

---

# Validation Result

Result:

```text
PASS
```

The Log Analytics query engine successfully executed the validation query.

Current state:

```text
Workspace Deployment: PASS
Workspace Status: Active
Operational Issues: None Reported
Logs Interface: PASS
KQL Execution: PASS
Telemetry Ingestion: Not Yet Configured
Azure Monitor Agent: Not Yet Configured
Data Collection Rule: Not Yet Configured
```

---

# Workspace Investigation Model

Future troubleshooting will use the following general workflow:

```text
Monitoring Issue
      ↓
Identify Time Window
      ↓
Identify Resource
      ↓
Open Log Analytics
      ↓
Select Relevant Table
      ↓
Filter Data with KQL
      ↓
Correlate Events
      ↓
Identify Root Cause
      ↓
Remediate
      ↓
Run Validation Query
```

---

# Planned Tables

The following tables are expected to become important later in the project.

## Heartbeat

Used to validate communication between monitored systems and Azure Monitor.

Example future investigation:

```kusto
Heartbeat
| where TimeGenerated > ago(1h)
| summarize LastHeartbeat=max(TimeGenerated) by Computer
| order by LastHeartbeat desc
```

This will help investigate scenarios where telemetry stops arriving.

---

# Event

Windows Event data will be queried after Windows Event collection is configured.

Example future query:

```kusto
Event
| where TimeGenerated > ago(1h)
| order by TimeGenerated desc
```

This will support Windows troubleshooting scenarios.

---

# AzureActivity

Azure Activity data will later be used to correlate resource-management operations.

Example future investigations include:

* VM stopped
* VM started
* Resource configuration changed
* Azure operation failed
* Resource deleted

The available table and schema will be validated after Activity Log routing is configured.

---

# KQL Repository Structure

Queries used during this project are stored separately from the documentation.

```text
KQL/
├── Heartbeat-Queries.kql
├── Performance-Queries.kql
├── Windows-Event-Queries.kql
├── Activity-Log-Queries.kql
└── Incident-Investigation-Queries.kql
```

The initial workspace validation query was stored in:

```text
KQL/Incident-Investigation-Queries.kql
```

---

# Query Documentation Standard

Each important query will include:

```text
Query Name
Purpose
Scenario
Table
Time Range
Query
Expected Result
Troubleshooting Interpretation
```

This makes the KQL repository useful as both:

* Lab evidence
* Troubleshooting reference material

---

# Screenshot Evidence

The following screenshots document this phase.

## Workspace Deployment

```text
Screenshots/02-Log-Analytics/
01-LAW-AZMON-SUPPORT-LAB-Deployment.png
```

Demonstrates successful workspace deployment.

---

## Workspace Overview

```text
Screenshots/02-Log-Analytics/
02-LAW-AZMON-SUPPORT-LAB-Overview.png
```

Demonstrates:

* Workspace name
* Active status
* Region
* Pricing tier
* Operational health
* Access to Logs

---

## KQL Workspace Validation

```text
Screenshots/07-KQL/
01-KQL-Workspace-Validation.png
```

Demonstrates successful execution of the first KQL query.

---

# Security and Portfolio Considerations

Azure screenshots may expose identifiers that are not useful to recruiters or reviewers.

Before final GitHub publication, screenshots will be reviewed for unnecessary exposure of:

* Subscription IDs
* Workspace IDs
* Tenant identifiers
* Public IP addresses
* User account information
* Authentication information
* Secrets or credentials

Browser address bars may also contain Azure resource identifiers and should be cropped where appropriate.

---

# Troubleshooting Baseline

The workspace is currently considered healthy.

Baseline:

```text
Workspace:
LAW-AZMON-SUPPORT-LAB

Status:
Active

Operational Issues:
None reported

KQL Query Engine:
Operational

Connected VM:
None

Azure Monitor Agent:
Not deployed

Data Collection Rule:
Not created

Heartbeat Data:
Not expected yet

Windows Event Data:
Not expected yet

Guest Performance Data:
Not expected yet
```

This baseline is important because future telemetry problems can be compared against a known-good workspace state.

---

# Skills Demonstrated

This phase demonstrates:

* Azure Portal administration
* Azure Resource Groups
* Azure Monitor
* Log Analytics
* Workspace deployment
* Centralized monitoring architecture
* KQL
* Query validation
* Monitoring baseline creation
* Cloud troubleshooting methodology
* Technical documentation
* Security-conscious screenshot management

---

# Phase Status

```text
Resource Group Creation: Complete
Log Analytics Workspace Deployment: Complete
Workspace Operational Validation: Complete
Initial KQL Validation: Complete
KQL Repository Entry: Complete
Azure Monitor Agent Deployment: Not Started
Data Collection Rule: Not Started
VM Telemetry Collection: Not Started
```

---

# Next Phase

The next phase of the lab will introduce the monitored Windows virtual machine and the telemetry collection pipeline.

Planned sequence:

```text
AZMON-WIN01
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Heartbeat / Events / Performance Data
      ↓
KQL Investigation
```

The next major objective is to deploy the Windows VM and establish the infrastructure required for Azure Monitor Agent and Data Collection Rule testing.

