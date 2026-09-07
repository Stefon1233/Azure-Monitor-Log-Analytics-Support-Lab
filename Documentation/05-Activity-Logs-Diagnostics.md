# Azure Monitor + Log Analytics Support Lab

## 05 — Activity Logs and Diagnostic Settings

## Overview

This phase of the Azure Monitor + Log Analytics Support Lab focuses on Azure control-plane monitoring through the Azure Activity Log, subscription-level diagnostic settings, Log Analytics, and 
Kusto Query Language.

The objective was to collect Azure administrative activity from the subscription and route that telemetry into:

```text
LAW-AZMON-SUPPORT-LAB
```

where the data could be queried through:

```text
AzureActivity
```

This phase also produced a useful troubleshooting scenario.

After the diagnostic setting was created, Azure Activity Log events were visible in the Azure portal but initially did not appear in Log Analytics.

Instead of recreating the configuration immediately, the monitoring path was validated layer by layer.

The investigation ultimately confirmed:

```text
Azure Activity Log Source Events: VERIFIED
Diagnostic Setting: VERIFIED
Log Analytics Destination: VERIFIED
AzureActivity Ingestion: VERIFIED
Controlled Administrative Event: VERIFIED
KQL Investigation: VERIFIED
```

This demonstrated both Azure monitoring configuration and a realistic telemetry-ingestion troubleshooting workflow.

---

# Objectives

The objectives of this phase were to:

- Review Azure Activity Log events
- Establish an Activity Log baseline
- Configure subscription-level diagnostic settings
- Export Activity Log data to Log Analytics
- Validate diagnostic setting categories
- Validate the Log Analytics destination
- Generate a controlled administrative event
- Confirm source Activity Log events exist
- Troubleshoot delayed AzureActivity ingestion
- Validate the AzureActivity table
- Query administrative operations with KQL
- Validate Start and Success operation lifecycle records
- Store reusable KQL queries in the repository
- Correlate Azure administrative activity with operational incidents
- Document the complete troubleshooting workflow

---

# Environment

## Azure Subscription

```text
Azure subscription 1
```

---

## Resource Group

```text
RG-AZMON-SUPPORT-LAB
```

---

## Virtual Machine

```text
AZMON-WIN01
```

Operating system:

```text
Windows Server 2022 Datacenter: Azure Edition
```

Region:

```text
North Central US
```

---

## Log Analytics Workspace

```text
LAW-AZMON-SUPPORT-LAB
```

Region:

```text
North Central US
```

---

## Data Collection Rule

```text
DCR-AZMON-WINDOWS
```

---

## Diagnostic Setting

```text
DIAG-AZMON-ACTIVITY-LOG
```

---

# Azure Monitoring Architecture

This lab now contains two major telemetry paths.

## Guest Operating System Telemetry

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

This path provides guest operating system monitoring.

---

## Azure Control-Plane Telemetry

```text
Azure Subscription
      ↓
Azure Activity Log
      ↓
DIAG-AZMON-ACTIVITY-LOG
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
AzureActivity
      ↓
KQL
```

This path provides Azure administrative and control-plane monitoring.

---

# Why Activity Log Monitoring Matters

Guest telemetry answers questions about what is happening inside the operating system.

Examples:

```text
Is the Azure Monitor Agent communicating?
Is CPU utilization high?
Are Windows warnings or errors occurring?
Is memory available?
Is disk activity abnormal?
```

The Azure Activity Log answers a different set of questions:

```text
What Azure administrative operation occurred?
When did the operation occur?
Was the operation successful?
Which resource group was affected?
Which Azure resource provider processed the request?
Was a resource created, modified, started, stopped, or deleted?
```

Both perspectives are useful during cloud troubleshooting.

---

# Azure Activity Log

Azure Activity Log records Azure Resource Manager control-plane operations.

Examples observed during this lab included:

```text
Run Command on Virtual Machine
Create or update metric alert
Create or update action group
Create or update data collection rule
Create or Update Virtual Machine
Create or Update Virtual Machine Extension
Validate Deployment
Add or modify schedules
Health Event Updated
Health Event Resolved
```

These records provide an administrative history of changes made to the Azure environment.

---

# Activity Log Baseline

Before configuring Activity Log export, the Azure Monitor Activity Log was reviewed directly in the portal.

The baseline confirmed that Azure was already recording control-plane activity generated throughout the lab.

The Activity Log contained events related to:

```text
AZMON-WIN01
DCR-AZMON-WINDOWS
ALRT-AZMON-HIGH-CPU
AG-AZMON-SUPPORT
RG-AZMON-SUPPORT-LAB
Azure VM Run Command
Deployment operations
Monitoring configuration
```

This was important because it established that source events existed before troubleshooting Log Analytics ingestion.

---

# Baseline Investigation

The Activity Log was filtered using:

```text
Subscription:
Azure subscription 1

Timespan:
Last 24 hours

Resource Group:
RG-AZMON-SUPPORT-LAB
```

The filtered result showed dozens of administrative operations.

This confirmed:

```text
Azure Activity Log Source:
HEALTHY
```

---

# Diagnostic Setting Configuration

A subscription-level diagnostic setting was created.

Diagnostic setting name:

```text
DIAG-AZMON-ACTIVITY-LOG
```

Destination:

```text
LAW-AZMON-SUPPORT-LAB
```

Destination type:

```text
Log Analytics Workspace
```

---

# Activity Log Categories

The diagnostic setting was configured with the Activity Log categories available in the Azure portal.

Selected categories:

```text
Administrative
Security
ServiceHealth
Alert
Recommendation
Policy
Autoscale
ResourceHealth
```

These categories provide broad Azure control-plane visibility.

---

# Administrative Category

The Administrative category was particularly important for this lab.

Administrative operations include actions such as:

```text
Resource creation
Resource modification
Virtual machine operations
Alert rule changes
Action group changes
Data Collection Rule changes
Tag changes
Deployment actions
```

The controlled event generated later in the lab was recorded in this category.

---

# Security Category

The Security category was enabled to capture subscription Activity Log security-related events where available.

---

# ServiceHealth Category

The ServiceHealth category was enabled to provide visibility into Azure service-health related control-plane activity.

---

# Alert Category

The Alert category was enabled to collect Activity Log alert-related records.

---

# Recommendation Category

The Recommendation category was enabled to capture applicable Azure recommendations.

---

# Policy Category

The Policy category was enabled to provide visibility into policy-related subscription operations.

---

# Autoscale Category

The Autoscale category was enabled to capture applicable autoscale-related events.

---

# ResourceHealth Category

The ResourceHealth category was enabled to capture health-related resource events.

---

# Destination Configuration

The diagnostic setting was configured with:

```text
Send to Log Analytics workspace:
Enabled
```

Selected subscription:

```text
Azure subscription 1
```

Selected workspace:

```text
LAW-AZMON-SUPPORT-LAB
```

The following destinations were not enabled:

```text
Archive to a storage account
Stream to an event hub
Send to partner solution
```

The purpose of this lab was Log Analytics investigation.

---

# Diagnostic Setting Validation

After the diagnostic setting was created, the configuration was reopened and validated.

The portal showed:

```text
Diagnostic Setting:
DIAG-AZMON-ACTIVITY-LOG

Administrative:
Enabled

Security:
Enabled

ServiceHealth:
Enabled

Alert:
Enabled

Recommendation:
Enabled

Policy:
Enabled

Autoscale:
Enabled

ResourceHealth:
Enabled

Send to Log Analytics workspace:
Enabled

Workspace:
LAW-AZMON-SUPPORT-LAB
```

No unsaved configuration changes were present.

This confirmed that the diagnostic setting itself was correctly configured.

---

# Expected Destination Table

Activity Log records exported to Log Analytics are available through:

```text
AzureActivity
```

This table becomes the control-plane counterpart to the existing guest monitoring tables:

```text
Heartbeat
Perf
Event
AzureActivity
```

---

# Initial AzureActivity Validation

The first validation query was:

```kusto
AzureActivity
| where TimeGenerated > ago(2h)
| project TimeGenerated, OperationNameValue, ActivityStatusValue,
          ResourceGroup, ResourceProviderValue
| order by TimeGenerated desc
```

Initially, the query returned:

```text
No results
```

---

# Initial Troubleshooting Decision

The empty result was not immediately treated as a configuration failure.

The following possibilities were considered:

```text
Diagnostic setting propagation delay
Log Analytics ingestion delay
Query time range too narrow
No qualifying event after export configuration
Incorrect diagnostic setting scope
Incorrect workspace destination
```

Rather than deleting and recreating the setting, each layer was verified independently.

---

# Troubleshooting Principle

A key troubleshooting principle used during this phase was:

```text
Do not change a configuration until evidence identifies the configuration as the problem.
```

Repeatedly rebuilding monitoring settings can make troubleshooting more difficult because it changes the environment while the investigation is still underway.

---

# Source Activity Log Verification

The Azure Monitor Activity Log was checked directly.

Filter:

```text
Timespan:
Last 24 hours

Resource Group:
RG-AZMON-SUPPORT-LAB
```

The result showed:

```text
35 Activity Log events
```

Recent operations included:

```text
Run Command on Virtual Machine
Create or update metric alert
Create or update action group
Create or update data collection rule
Create or Update Virtual Machine
Create or Update Virtual Machine Extension
Validate Deployment
```

This proved:

```text
Source Activity Log Events:
PRESENT
```

---

# Troubleshooting Isolation

At this point the investigation established:

```text
Azure Activity Log source events:
VERIFIED

Diagnostic setting:
VERIFIED

Workspace destination:
VERIFIED

AzureActivity ingestion:
NOT YET VERIFIED
```

This isolated the issue to the export or ingestion portion of the monitoring path.

---

# Controlled Administrative Event

To create a known event that could be searched later, a temporary tag was added to:

```text
RG-AZMON-SUPPORT-LAB
```

Example controlled tag:

```text
ActivityTest = Export-Validation-2
```

This was selected because a tag modification:

- Creates a real Azure Resource Manager operation
- Does not interrupt the virtual machine
- Does not alter networking
- Does not restart services
- Can be easily removed
- Produces useful control-plane evidence

---

# Controlled Event Workflow

```text
RG-AZMON-SUPPORT-LAB
      ↓
Temporary Tag Added
      ↓
Azure Resource Manager Processes Change
      ↓
Microsoft.Resources/tags/write
      ↓
Azure Activity Log
      ↓
DIAG-AZMON-ACTIVITY-LOG
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
AzureActivity
```

---

# Initial Controlled Event Query

The following lab-focused query was used:

```kusto
AzureActivity
| where TimeGenerated > ago(2h)
| where ResourceGroup =~ "RG-AZMON-SUPPORT-LAB"
| project TimeGenerated, OperationNameValue, ActivityStatusValue,
          ResourceGroup, ResourceProviderValue
| order by TimeGenerated desc
```

Initially, this query also returned no records.

---

# Delayed Ingestion

The diagnostic setting configuration was left unchanged while Azure completed the export and ingestion process.

This was important because all source-side evidence already indicated that the monitoring configuration was valid.

The environment remained:

```text
Source events:
AVAILABLE

Diagnostic setting:
CONFIGURED

Workspace:
CORRECT

AzureActivity:
WAITING FOR INGESTION
```

---

# Successful AzureActivity Ingestion

After allowing additional processing time, the same AzureActivity query was executed again.

This time Log Analytics returned a record.

Observed values included:

```text
OperationNameValue:
MICROSOFT.RESOURCES/TAGS/WRITE

ActivityStatusValue:
Start

ResourceGroup:
RG-AZMON-SUPPORT-LAB

ResourceProviderValue:
MICROSOFT.RESOURCES
```

This was the first successful confirmation that Activity Log records had reached the workspace.

---

# End-to-End Ingestion Validation

The successful record proved the complete path:

```text
Azure Resource Manager Operation
      ↓
Azure Activity Log
      ↓
Subscription Diagnostic Setting
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
AzureActivity
      ↓
KQL Results
```

Status:

```text
AzureActivity Ingestion:
VERIFIED
```

---

# Controlled Tag Write Investigation

A more targeted KQL query was then executed:

```kusto
AzureActivity
| where TimeGenerated > ago(4h)
| where ResourceGroup =~ "RG-AZMON-SUPPORT-LAB"
| where OperationNameValue =~ "MICROSOFT.RESOURCES/TAGS/WRITE"
| project TimeGenerated, OperationNameValue, ActivityStatusValue,
          ResourceGroup, ResourceProviderValue
| order by TimeGenerated desc
```

The query returned two related operation records.

---

# Start Record

The first record represented the beginning of the controlled Azure operation.

Observed state:

```text
ActivityStatusValue:
Start
```

Operation:

```text
MICROSOFT.RESOURCES/TAGS/WRITE
```

Resource group:

```text
RG-AZMON-SUPPORT-LAB
```

Resource provider:

```text
MICROSOFT.RESOURCES
```

---

# Success Record

The second record represented successful completion of the same administrative operation.

Observed state:

```text
ActivityStatusValue:
Success
```

Operation:

```text
MICROSOFT.RESOURCES/TAGS/WRITE
```

Resource group:

```text
RG-AZMON-SUPPORT-LAB
```

---

# Administrative Operation Lifecycle

The controlled event demonstrated that one Azure administrative action can generate multiple Activity Log records.

Observed lifecycle:

```text
Tag Write Requested
      ↓
ActivityStatusValue = Start
      ↓
Azure Resource Manager Processes Operation
      ↓
ActivityStatusValue = Success
```

This provides additional context when investigating Azure control-plane operations.

---

# Successful Validation Result

Final controlled-event validation:

```text
Operation:
MICROSOFT.RESOURCES/TAGS/WRITE

Resource Group:
RG-AZMON-SUPPORT-LAB

Start Record:
VERIFIED

Success Record:
VERIFIED

AzureActivity:
VERIFIED
```

---

# AzureActivity KQL Queries

Reusable Activity Log queries are stored in:

```text
KQL/Activity-Log-Queries.kql
```

---

# Query 01 — Recent Activity

```kusto
AzureActivity
| where TimeGenerated > ago(2h)
| project TimeGenerated, OperationNameValue, ActivityStatusValue,
          ResourceGroup, ResourceProviderValue
| order by TimeGenerated desc
```

Purpose:

```text
Verify Activity Log ingestion and review recent administrative operations.
```

---

# Query 02 — Recent Subscription Operations

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| order by TimeGenerated desc
| take 50
```

Purpose:

```text
Review a broader sample of Azure control-plane operations.
```

---

# Query 03 — Failed Azure Operations

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where ActivityStatusValue =~ "Failed"
| project TimeGenerated, OperationNameValue, ActivityStatusValue,
          ResourceGroup, ResourceProviderValue
| order by TimeGenerated desc
```

Purpose:

```text
Identify failed Azure administrative operations.
```

---

# Query 04 — Lab Resource Group Operations

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where ResourceGroup =~ "RG-AZMON-SUPPORT-LAB"
| project TimeGenerated, OperationNameValue, ActivityStatusValue,
          ResourceGroup, ResourceProviderValue
| order by TimeGenerated desc
```

Purpose:

```text
Limit investigation to this monitoring lab.
```

---

# Query 05 — Controlled Tag Write

```kusto
AzureActivity
| where TimeGenerated > ago(4h)
| where ResourceGroup =~ "RG-AZMON-SUPPORT-LAB"
| where OperationNameValue =~ "MICROSOFT.RESOURCES/TAGS/WRITE"
| project TimeGenerated, OperationNameValue, ActivityStatusValue,
          ResourceGroup, ResourceProviderValue
| order by TimeGenerated desc
```

Purpose:

```text
Verify the controlled administrative event and its Start/Success lifecycle.
```

---

# Query 06 — Activity Volume

```kusto
AzureActivity
| summarize Records=count(),
            FirstEvent=min(TimeGenerated),
            LastEvent=max(TimeGenerated)
```

Purpose:

```text
Quickly verify whether AzureActivity contains any records.
```

---

# Query 07 — Daily Activity Count

```kusto
AzureActivity
| summarize count() by bin(TimeGenerated, 1d)
| order by TimeGenerated desc
```

Purpose:

```text
Review Activity Log ingestion volume by day.
```

---

# Query 08 — Operation Status Summary

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| summarize Count=count() by ActivityStatusValue
| order by Count desc
```

Purpose:

```text
Compare successful, started, failed, and other operation states.
```

---

# Query 09 — Most Common Operations

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| summarize Count=count() by OperationNameValue
| order by Count desc
```

Purpose:

```text
Identify the most common Azure administrative operations.
```

---

# Query 10 — Resource Provider Activity

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| summarize Count=count() by ResourceProviderValue
| order by Count desc
```

Purpose:

```text
Identify which Azure resource providers generated the most activity.
```

---

# Activity Log and High CPU Incident Correlation

Activity Log telemetry complements:

```text
INC-001 — High CPU Utilization
```

The high CPU condition itself was generated inside Windows and detected through Azure platform metrics.

However, Azure control-plane operations occurring near the incident could be reviewed through AzureActivity.

Examples include:

```text
Run Command on Virtual Machine
Create or update metric alert
Create or update action group
```

This provides administrative context surrounding the incident.

---

# Run Command Correlation

The controlled CPU workload and Windows event tests were executed using:

```text
Azure VM Run Command
```

Activity Log records showed operations such as:

```text
Run Command on Virtual Machine
```

This provides evidence that administrative actions against the VM can be correlated with later guest operating system behavior.

---

# Future Availability Incident Correlation

Activity Log will be especially valuable for a future controlled VM availability incident.

Example:

```text
VM Stop or Deallocate Operation
      ↓
AzureActivity Record
      ↓
VM Availability Metric Drops
      ↓
Heartbeat Changes
      ↓
Azure Monitor Alert
      ↓
Incident Investigation
```

This provides multiple independent evidence sources for root-cause analysis.

---

# Control Plane vs Data Plane

An important concept demonstrated in this lab is the distinction between control-plane and guest/data-plane telemetry.

## Control Plane

AzureActivity provides information about Azure management operations.

Examples:

```text
Create VM
Update VM
Run Command
Modify Alert Rule
Change Tags
Create DCR
```

---

## Guest Operating System

The following provide information from inside Windows:

```text
Heartbeat
Perf
Event
```

---

## Platform Metrics

Azure Monitor metrics provide resource-level numeric telemetry such as:

```text
Percentage CPU
Network In
Network Out
Disk Read
Disk Write
VM Availability
```

---

# Layered Troubleshooting Model

The monitoring environment can now answer questions across multiple layers.

```text
Azure Control Plane
      ↓
AzureActivity

Azure Platform
      ↓
Metrics

Guest Operating System
      ↓
Heartbeat / Perf / Event

Detection
      ↓
Azure Monitor Alerts
```

This layered approach improves troubleshooting accuracy.

---

# Activity Log Troubleshooting Methodology

The delayed-ingestion scenario produced the following troubleshooting methodology.

## Step 1 — Verify Source Events

```text
Monitor
→ Activity Log
→ Confirm events exist
```

Result:

```text
VERIFIED
```

---

## Step 2 — Verify Diagnostic Setting

```text
DIAG-AZMON-ACTIVITY-LOG
```

Confirm:

```text
Categories enabled
Log Analytics enabled
Correct workspace selected
```

Result:

```text
VERIFIED
```

---

## Step 3 — Verify Destination

Workspace:

```text
LAW-AZMON-SUPPORT-LAB
```

Result:

```text
VERIFIED
```

---

## Step 4 — Generate Known Activity

Controlled event:

```text
Microsoft.Resources/tags/write
```

Result:

```text
GENERATED
```

---

## Step 5 — Query AzureActivity

Initial result:

```text
No results
```

---

## Step 6 — Avoid Unnecessary Reconfiguration

Because source events and diagnostic configuration were correct, the setting was left unchanged.

---

## Step 7 — Allow Ingestion Time

Azure completed the monitoring export path.

---

## Step 8 — Requery AzureActivity

Result:

```text
Records returned
```

---

## Step 9 — Validate Controlled Event

Observed:

```text
Start
Success
```

Result:

```text
VERIFIED
```

---

# Troubleshooting Outcome

The issue was not caused by:

```text
Missing source events
Incorrect diagnostic category selection
Incorrect Log Analytics workspace
Missing Administrative category
Incorrect query syntax
```

The monitoring pipeline required additional time before AzureActivity records became visible.

No destructive corrective action was required.

---

# Root Cause of Initial No-Results Condition

The initial no-results condition was consistent with delayed diagnostic export / ingestion after the subscription diagnostic setting was created.

The configuration itself was valid.

Evidence supporting this conclusion:

```text
Activity Log source events existed
Diagnostic setting remained unchanged
Workspace destination remained unchanged
AzureActivity later populated successfully
Controlled event appeared without rebuilding the setting
```

---

# Resolution

Resolution was achieved without deleting or recreating:

```text
DIAG-AZMON-ACTIVITY-LOG
```

The diagnostic configuration remained intact.

After Azure completed ingestion processing, the `AzureActivity` table began returning data.

Resolution:

```text
Allow monitoring pipeline to complete ingestion
+
Revalidate AzureActivity
+
Confirm controlled event lifecycle
```

---

# Why This Is Useful Portfolio Evidence

This phase demonstrates more than simply creating a diagnostic setting.

It demonstrates:

```text
Monitoring configuration
+
Failure observation
+
Evidence collection
+
Scope isolation
+
Controlled test generation
+
KQL validation
+
Successful resolution
```

This is closer to a real cloud support workflow than a simple configuration walkthrough.

---

# Screenshot Evidence

Evidence for this phase is stored in:

```text
Screenshots/05-Activity-Logs/
```

The evidence sequence includes:

```text
01-Azure-Activity-Log-Baseline.png
02-Azure-Activity-Log-Diagnostic-Setting.png
03-Azure-Activity-Log-Diagnostic-Setting-Saved.png
04-AzureActivity-KQL-No-Results.png
05-Azure-Activity-Log-Diagnostic-Setting-Validation.png
06-Azure-Activity-Log-Source-Events-Verified.png
07-Controlled-Activity-Log-Event-Generated.png
08-AzureActivity-KQL-Ingestion-Verified.png
09-Controlled-Activity-Log-Write-Verified.png
```

The exact screenshot count is validated separately through the repository.

---

# Evidence Progression

The screenshot sequence documents:

```text
Activity Log Baseline
      ↓
Diagnostic Setting Configuration
      ↓
Diagnostic Setting Saved
      ↓
Controlled Activity Generated
      ↓
Initial AzureActivity No Results
      ↓
Diagnostic Setting Revalidated
      ↓
Source Events Confirmed
      ↓
Fresh Controlled Event Generated
      ↓
AzureActivity Ingestion Verified
      ↓
Controlled Start/Success Lifecycle Verified
```

---

# Security and Privacy Considerations

Before GitHub publication, Activity Log screenshots should avoid exposing unnecessary identifiers.

Review or crop:

```text
Azure account identity
Subscription ID
Browser URL
Tenant-specific identifiers
Public IP addresses
Email addresses
```

Lab resource names remain useful technical evidence and can remain visible.

Examples:

```text
RG-AZMON-SUPPORT-LAB
LAW-AZMON-SUPPORT-LAB
DCR-AZMON-WINDOWS
AZMON-WIN01
```

---

# Cost Considerations

Diagnostic settings can increase Log Analytics ingestion depending on event volume.

This lab contains:

```text
1 subscription diagnostic setting
1 Log Analytics workspace
1 primary lab resource group
```

Activity Log event volume for a small lab environment is limited compared with a production subscription.

The broader monitoring architecture is intentionally sized for demonstration and troubleshooting rather than large-scale production ingestion.

---

# Lessons Learned

## Lesson 1 — Source Data Must Be Verified First

An empty Log Analytics query does not prove that Azure has no events.

Always verify the source.

In this phase:

```text
Activity Log source events:
PRESENT
```

while:

```text
AzureActivity:
Initially empty
```

---

## Lesson 2 — Validate the Entire Pipeline

The correct troubleshooting path was:

```text
Source
→ Diagnostic Setting
→ Destination
→ Ingestion
→ Query
```

rather than focusing only on the KQL result.

---

## Lesson 3 — Avoid Unnecessary Changes

Because the diagnostic setting configuration was already correct, recreating it would have introduced additional variables.

Leaving the configuration intact ultimately proved that the original setting worked.

---

## Lesson 4 — Generate Known Test Data

The controlled tag write created a known event that could be tracked through the entire monitoring pipeline.

This is much stronger than waiting for an unknown random Azure operation.

---

## Lesson 5 — One Operation Can Produce Multiple Records

The controlled tag modification generated:

```text
Start
Success
```

records.

Understanding operation lifecycle records is important when interpreting AzureActivity.

---

## Lesson 6 — Correlate Control-Plane and Guest Data

AzureActivity alone does not show everything happening inside a VM.

Guest and platform telemetry provide additional context.

A complete Azure support investigation may combine:

```text
AzureActivity
Heartbeat
Perf
Event
Metrics
Alerts
```

---

# Operational Troubleshooting Checklist

If AzureActivity appears empty in a future environment:

```text
1. Verify Activity Log source events exist
2. Confirm diagnostic setting exists
3. Confirm Administrative category is enabled
4. Confirm Log Analytics destination is enabled
5. Confirm correct workspace is selected
6. Generate a known administrative event
7. Expand KQL time range
8. Verify query syntax
9. Allow ingestion processing time
10. Requery AzureActivity
11. Avoid rebuilding configuration without evidence
```

---

# Skills Demonstrated

This phase demonstrates:

- Microsoft Azure
- Azure Monitor
- Azure Activity Log
- Azure Resource Manager
- Subscription diagnostic settings
- Log Analytics
- AzureActivity
- Kusto Query Language
- Control-plane monitoring
- Administrative event investigation
- Resource group filtering
- Azure tag operations
- Monitoring pipeline validation
- Telemetry troubleshooting
- Controlled test generation
- Evidence-based troubleshooting
- Root-cause isolation
- Incident correlation
- Technical documentation
- GitHub portfolio evidence management

---

# Final Monitoring Pipeline Status

```text
Azure Activity Log Source Events: VERIFIED
Diagnostic Setting: VERIFIED
Administrative Category: VERIFIED
Log Analytics Destination: VERIFIED
Controlled Tag Write: VERIFIED
AzureActivity Table: VERIFIED
Start Operation Record: VERIFIED
Success Operation Record: VERIFIED
KQL Validation: VERIFIED
Telemetry Ingestion Troubleshooting: COMPLETE
```

---

# Phase Status

```text
Activity Log Baseline: COMPLETE
Diagnostic Setting: COMPLETE
Diagnostic Setting Validation: VERIFIED
Log Analytics Destination: VERIFIED
Controlled Administrative Event: COMPLETE
Initial No-Results Condition: DOCUMENTED
Source Activity Events: VERIFIED
AzureActivity Ingestion: VERIFIED
Controlled Tag Write Start: VERIFIED
Controlled Tag Write Success: VERIFIED
KQL Repository Queries: COMPLETE
Troubleshooting Workflow: COMPLETE
```

This phase is complete and demonstrates both the configuration and troubleshooting of Azure Activity Log export to Log Analytics.
