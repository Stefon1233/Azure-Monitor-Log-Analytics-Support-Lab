# Azure Monitor + Log Analytics Support Lab

## 06 — Azure Monitor Alerts

## Overview

This phase of the Azure Monitor + Log Analytics Support Lab focuses on proactive monitoring and alert-based incident detection.

Azure Monitor alerts allow operational conditions to be detected automatically instead of requiring an administrator to continuously watch metric charts.

The primary alert implemented and validated during this phase was:

```text
ALRT-AZMON-HIGH-CPU
```

The alert monitors:

```text
AZMON-WIN01
```

for sustained high CPU utilization.

A controlled PowerShell workload was intentionally generated on the Windows Server VM to increase processor utilization, trigger the alert, validate the Fired state, monitor recovery, and verify 
automatic alert resolution.

This phase demonstrates the complete alert lifecycle:

```text
Normal Resource State
      ↓
Metric Collected
      ↓
Threshold Exceeded
      ↓
Azure Monitor Evaluates Condition
      ↓
Alert Fires
      ↓
Action Group Processes Notification
      ↓
Incident Investigation
      ↓
Root Cause Identified
      ↓
Condition Clears
      ↓
Alert Automatically Resolves
```

---

## Objectives

The objectives of this phase were to:

- Create an Azure Monitor metric alert
- Monitor Percentage CPU for AZMON-WIN01
- Configure a static CPU threshold
- Create an Azure Monitor Action Group
- Configure email notification
- Assign alert severity
- Enable automatic alert resolution
- Validate the alert rule after deployment
- Generate a controlled high CPU condition
- Observe the CPU increase in Azure Monitor
- Confirm the alert enters the Fired state
- Investigate the affected resource
- Validate the controlled workload root cause
- Confirm the workload terminates
- Verify CPU recovery
- Confirm the alert automatically resolves
- Document the complete alert lifecycle

---

# Environment

## Virtual Machine

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

# Monitoring Baseline

Before configuring the high CPU alert, the monitoring environment was validated.

The following components were already operational:

```text
Azure Monitor Agent: VERIFIED
DCR Association: VERIFIED
Heartbeat: VERIFIED
Performance Counters: VERIFIED
Windows Event Collection: VERIFIED
Log Analytics Ingestion: VERIFIED
Percentage CPU Metric: VERIFIED
VM Availability Metric: VERIFIED
```

Establishing this baseline was important because it separated monitoring configuration problems from the actual simulated performance incident.

---

# Why Alerting Was Added

Azure Monitor metrics provide visibility into resource conditions, but an administrator would otherwise need to manually inspect charts to identify abnormal behavior.

An alert rule automates this process.

The monitoring model becomes:

```text
Metric
      ↓
Condition
      ↓
Alert Rule
      ↓
Action Group
      ↓
Operational Response
```

For this lab:

```text
Percentage CPU
      ↓
Average CPU > 80%
      ↓
ALRT-AZMON-HIGH-CPU
      ↓
AG-AZMON-SUPPORT
      ↓
Incident Investigation
```

---

# High CPU Alert Rule

The alert rule created during this phase is:

```text
ALRT-AZMON-HIGH-CPU
```

Scope:

```text
AZMON-WIN01
```

Signal:

```text
Percentage CPU
```

---

# Alert Condition

The CPU alert was configured using a static metric threshold.

Configuration:

```text
Signal:
Percentage CPU

Threshold Type:
Static

Aggregation Type:
Average

Operator:
Greater Than

Threshold:
80%

Evaluation Frequency:
1 minute

Lookback Period:
5 minutes
```

---

## Threshold Explanation

The 80-percent threshold was selected for the controlled lab because it represents sustained CPU pressure while remaining practical to trigger using a temporary test workload.

The rule does not fire simply because one instantaneous CPU sample exceeds 80 percent.

Azure evaluates the configured metric using:

```text
Average CPU
over
5-minute lookback window
```

and reevaluates the rule:

```text
Every 1 minute
```

This reduces the likelihood that a very brief CPU spike alone will trigger the incident.

---

# Alert Severity

The alert was configured as:

```text
Severity 2 — Warning
```

This severity was appropriate for the controlled high CPU scenario because:

- The VM remained operational
- The condition represented degraded performance
- Immediate awareness was useful
- The incident did not represent a complete outage

---

# Action Group

An Azure Monitor Action Group was created:

```text
AG-AZMON-SUPPORT
```

Display name:

```text
AZMONSUPPORT
```

The display name was configured within Azure's character limit.

---

## Action Group Purpose

The Action Group provides the notification path for the alert.

Alert flow:

```text
ALRT-AZMON-HIGH-CPU
      ↓
AG-AZMON-SUPPORT
      ↓
Email Notification
```

An email notification was configured for the lab.

The email address is intentionally excluded or redacted from public GitHub screenshots.

---

# Custom Properties

Custom alert properties were not required for this initial alert.

Configuration:

```text
Custom Properties:
None
```

This kept the alert configuration focused on the actual monitoring condition.

Custom properties could be added later in a production environment to provide values such as:

```text
Environment
Application
SupportTeam
Runbook
EscalationGroup
```

---

# Automatic Resolution

Automatic alert resolution was enabled.

Configuration:

```text
Automatically resolve alerts:
Enabled
```

This allowed Azure Monitor to transition the incident from:

```text
Fired
```

to:

```text
Resolved
```

after the CPU condition returned below the configured threshold.

This was important because the lab was designed to demonstrate the full alert lifecycle rather than only alert creation.

---

# Alert Rule Description

The alert rule was documented with a description similar to:

```text
Alerts when average CPU utilization on AZMON-WIN01 exceeds 80 percent during a 5-minute evaluation window.
```

The description makes the operational purpose of the alert clear to another administrator reviewing the environment.

---

# Alert Rule Deployment

After configuring:

```text
Condition
Action Group
Severity
Rule Name
Description
Automatic Resolution
```

the alert was reviewed and created.

The alert rule was enabled immediately after deployment.

---

# Alert Rule Validation

After creation, the Azure Monitor Alert Rules page confirmed:

```text
ALRT-AZMON-HIGH-CPU
```

was:

```text
Enabled
```

This validated that the rule had been successfully deployed before the controlled CPU incident began.

---

# Alert Overview

The alert rule Overview page was captured as post-deployment evidence.

The Overview provided evidence of:

- Alert rule name
- Enabled state
- Resource scope
- Severity
- Signal condition
- Action group
- Evaluation configuration

This established a known-good alert baseline before generating the incident.

---

# Controlled High CPU Incident

The alert rule was tested using:

```text
INC-001 — High CPU Utilization on Azure Virtual Machine
```

The goal was to intentionally create a processor-intensive but temporary condition on AZMON-WIN01.

---

# Controlled Workload Design

The workload was designed with several safeguards.

Requirements:

- Generate sustained processor activity
- Use a known script
- Run multiple worker processes
- Automatically stop after a limited period
- Avoid permanent system changes
- Avoid rebooting or destabilizing the VM
- Run long enough for the 5-minute alert window

Maximum duration:

```text
10 minutes
```

Workers:

```text
2 PowerShell processes
```

---

# Controlled CPU Script

The workload was generated using Azure VM Run Command.

Script:

```powershell
New-Item -Path "C:\Temp" -ItemType Directory -Force | Out-Null

@'
$EndTime = (Get-Date).AddMinutes(10)

while ((Get-Date) -lt $EndTime) {
    for ($i = 0; $i -lt 1000000; $i++) {
        [math]::Sqrt($i) | Out-Null
    }
}
'@ | Set-Content "C:\Temp\AZMON-HighCPU.ps1"

1..2 | ForEach-Object {
    Start-Process powershell.exe `
        -ArgumentList "-NoProfile -ExecutionPolicy Bypass -File C:\Temp\AZMON-HighCPU.ps1" `
        -WindowStyle Hidden
}
```

---

# Controlled Workload Confirmation

The Run Command output confirmed that the high CPU workload had successfully started.

Expected validation included:

```text
Status:
CPU workload started

Workers:
2

Maximum duration:
10 minutes
```

This provided direct evidence that the test condition had been intentionally generated.

---

# Metric Response

After workload generation, Azure Monitor Percentage CPU was reviewed.

The metric chart showed a clear increase from the earlier baseline.

Monitoring flow:

```text
Normal CPU
      ↓
PowerShell Workload Starts
      ↓
Percentage CPU Increases
      ↓
CPU Remains Elevated
```

This demonstrated that the workload generated a real performance condition rather than only creating an alert configuration artifact.

---

# Alert Evaluation

The alert rule evaluated:

```text
Average Percentage CPU
```

against:

```text
80%
```

using:

```text
5-minute lookback
```

with:

```text
1-minute evaluation frequency
```

After sustained CPU utilization exceeded the configured condition, the alert fired.

---

# Fired Alert

Azure Monitor showed:

```text
ALRT-AZMON-HIGH-CPU
```

with the monitor condition:

```text
Fired
```

This proved that:

- The metric was being collected
- The rule was enabled
- The scope was correct
- The condition was evaluated
- CPU exceeded the threshold
- Azure Monitor generated an alert instance

---

# Fired Alert Investigation

The fired alert was opened for investigation.

Evidence included:

- Alert rule name
- Fired state
- Severity
- Affected resource
- Timestamp
- Metric condition
- Resource scope

Affected resource:

```text
AZMON-WIN01
```

Severity:

```text
2 — Warning
```

---

# Incident Detection Model

The successful test demonstrated:

```text
AZMON-WIN01
      ↓
Percentage CPU
      ↓
Average CPU > 80%
      ↓
Azure Monitor
      ↓
ALRT-AZMON-HIGH-CPU
      ↓
Fired Alert
```

This completed the detection stage of INC-001.

---

# Guest Performance Correlation

Azure platform metrics were not the only source of performance data available.

Guest performance telemetry was also collected through:

```text
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Perf
```

Example CPU query:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where CounterName contains "% Processor Time"
| project TimeGenerated, Computer, ObjectName, CounterName, CounterValue
| order by TimeGenerated desc
```

This allows Azure host metrics to be compared against Windows guest performance data.

---

# Root Cause

The root cause was known because the condition was intentionally generated.

Root cause:

```text
Controlled PowerShell CPU workload
```

Script:

```text
C:\Temp\AZMON-HighCPU.ps1
```

The workload launched two PowerShell workers specifically to create processor pressure.

---

# Process Validation

After the alert fired, the VM was checked for the controlled worker processes.

The workload had been configured with a maximum runtime of approximately 10 minutes.

By the time the post-test process check was performed:

```text
Workers detected:
0
```

No controlled CPU worker processes remained.

---

# Why Zero Workers Was Expected

The zero-worker result did not indicate that the alert test failed.

Evidence already confirmed:

1. The controlled workload started
2. CPU utilization increased
3. CPU exceeded the configured threshold
4. Azure Monitor detected the condition
5. ALRT-AZMON-HIGH-CPU fired
6. The workload reached its maximum runtime
7. The controlled worker processes exited

Therefore:

```text
Workers detected: 0
```

represented the post-workload state.

---

# Workload Termination

The test workload was intentionally designed with:

```text
10-minute automatic timeout
```

This safeguard prevented the lab from accidentally leaving a processor-intensive workload running indefinitely.

The remediation path became:

```text
Controlled Workload
      ↓
10-Minute Runtime
      ↓
Automatic Termination
      ↓
No Workers Remaining
      ↓
CPU Recovery Validation
```

---

# CPU Recovery

After the controlled workload terminated, Percentage CPU was reviewed again.

The chart showed:

```text
Normal Baseline
      ↓
CPU Spike
      ↓
High CPU Period
      ↓
CPU Decline
      ↓
Return Toward Baseline
```

This confirmed the performance condition had cleared.

---

# Recovery Validation

Recovery validation included:

- CPU utilization decreased
- CPU returned below the configured threshold
- No controlled PowerShell workers remained
- VM remained operational
- Monitoring continued
- Azure Monitor Agent remained active
- Guest telemetry continued arriving

This demonstrated that the VM recovered without requiring a reboot or permanent configuration change.

---

# Automatic Alert Resolution

Because automatic resolution was enabled, Azure Monitor continued evaluating the condition after CPU utilization decreased.

Once the metric no longer met:

```text
Average CPU > 80%
```

the alert transitioned from:

```text
Fired
```

to:

```text
Resolved
```

---

# Resolved Alert

The Azure Alerts page confirmed:

```text
ALRT-AZMON-HIGH-CPU
```

had entered the:

```text
Resolved
```

state.

This completed the operational alert lifecycle.

---

# Resolved Alert Overview

The resolved alert instance was opened to verify the final state.

The resolved alert evidence confirmed:

- Same alert rule
- Same affected VM
- Alert previously fired
- Condition later cleared
- Azure Monitor automatically resolved the alert

---

# Complete Alert Lifecycle

The final lifecycle demonstrated during this phase was:

```text
Alert Rule Created
      ↓
Alert Rule Enabled
      ↓
Normal CPU Baseline
      ↓
Controlled Workload Started
      ↓
CPU Increased
      ↓
Threshold Exceeded
      ↓
Alert Fired
      ↓
Incident Investigated
      ↓
Controlled Workload Timed Out
      ↓
CPU Recovered
      ↓
Condition Cleared
      ↓
Alert Automatically Resolved
```

---

# Alert State Comparison

## Normal

```text
CPU < 80%
Alert Condition = Not Met
```

---

## Fired

```text
Average CPU > 80%
Alert Condition = Met
Monitor Condition = Fired
```

---

## Recovered

```text
CPU Drops Below Threshold
Alert Condition No Longer Met
```

---

## Resolved

```text
Monitor Condition = Resolved
Incident Recovery Verified
```

---

# Alert Investigation Workflow

The troubleshooting workflow demonstrated by INC-001 was:

```text
Alert Notification
      ↓
Open Fired Alert
      ↓
Identify Resource
      ↓
Review Metric
      ↓
Confirm Threshold Violation
      ↓
Review Guest Performance Data
      ↓
Identify Workload
      ↓
Validate Workload Termination
      ↓
Review Recovery Metric
      ↓
Confirm Alert Resolution
      ↓
Close Incident
```

---

# Role of Action Groups

Action Groups separate:

```text
Alert Detection
```

from:

```text
Alert Notification
```

This allows multiple alert rules to reuse a centralized notification configuration.

In this lab:

```text
AG-AZMON-SUPPORT
```

can later be reused for:

```text
VM Availability Alerts
Activity Log Alerts
Log Search Alerts
Missing Heartbeat Alerts
Windows Event Alerts
```

This is more scalable than creating separate notification configurations for every rule.

---

# Alert Naming Standard

The lab uses:

```text
ALRT-AZMON-[CONDITION]
```

Examples include:

```text
ALRT-AZMON-HIGH-CPU
ALRT-AZMON-VM-AVAILABILITY
ALRT-AZMON-ACTIVITY-LOG
ALRT-AZMON-MISSING-HEARTBEAT
ALRT-AZMON-WINDOWS-ERRORS
```

This naming standard makes alert purpose immediately recognizable.

---

# Alert Severity Model

A simple severity model can be used throughout the lab.

Example:

```text
Severity 0:
Critical emergency

Severity 1:
Major outage or severe service impact

Severity 2:
Warning / degraded service

Severity 3:
Informational operational issue

Severity 4:
Low-priority informational event
```

INC-001 used:

```text
Severity 2
```

because the VM remained available despite elevated CPU.

---

# Metrics vs Alerts

Metrics answer:

```text
What is happening?
```

Alerts answer:

```text
When should someone respond?
```

For example:

```text
Percentage CPU = 95%
```

is metric data.

But:

```text
Average Percentage CPU > 80% for configured evaluation window
```

can become an actionable alert condition.

---

# Alerts vs Log Analytics

Metric alerts are useful for fast numeric conditions.

Log Analytics is useful for deeper investigation.

Example:

```text
Azure Metric:
Percentage CPU > 80%
```

can trigger the alert.

Then:

```text
Perf
```

can provide guest CPU telemetry for additional analysis.

---

# Alerts vs Windows Event Logs

Some issues are better detected from event data than from resource metrics.

For example:

```text
Windows Application Error
Windows System Error
Service Failure
Application Crash
```

may be better suited for a log-search alert.

The DCR already collects:

```text
Application:
Critical
Error
Warning

System:
Critical
Error
Warning
```

This provides the foundation for later log-based alerts.

---

# Alerts vs Activity Log

Azure Activity Log alerts can detect control-plane events such as:

```text
VM Start
VM Stop
VM Deallocate
Resource Delete
Administrative Change
Policy Event
Service Health Event
```

This complements metric alerts.

The lab therefore supports multiple alert sources:

```text
Metrics
Logs
Activity Log
```

---

# Future Alert Scenarios

The environment can support additional alert rules including:

```text
ALRT-AZMON-VM-AVAILABILITY
```

Purpose:

```text
Detect VM availability changes
```

---

```text
ALRT-AZMON-MISSING-HEARTBEAT
```

Purpose:

```text
Detect missing Azure Monitor Agent communication
```

---

```text
ALRT-AZMON-WINDOWS-ERRORS
```

Purpose:

```text
Detect selected Windows error events
```

---

```text
ALRT-AZMON-ACTIVITY-LOG
```

Purpose:

```text
Detect selected Azure control-plane operations
```

---

# Production High CPU Investigation

In a production environment, sustained high CPU could be caused by:

- Runaway application
- Increased legitimate workload
- Scheduled task
- Antivirus scan
- Windows Update activity
- Memory pressure causing additional CPU overhead
- Malware
- Inefficient application code
- Insufficient VM sizing
- Application loop
- Background processing

A production investigation would not assume the root cause in advance.

---

# Production Investigation Questions

Useful questions include:

```text
When did CPU begin increasing?
Was the increase sudden or gradual?
Is the CPU condition sustained?
Which process is responsible?
Did a deployment occur?
Did application traffic increase?
Did a scheduled task run?
Did Azure configuration change?
Are there relevant Windows events?
Has this happened before?
Does the VM need additional capacity?
```

---

# Potential Production Remediation

Depending on root cause, remediation could include:

- Stop or restart a runaway process
- Restart an affected service
- Roll back a deployment
- Optimize an application
- Modify a scheduled task
- Investigate malware
- Increase VM size
- Scale out workload
- Modify alert thresholds
- Add additional monitoring
- Escalate to application owners

---

# False Positive Considerations

Alert thresholds should be based on operational baselines.

An alert set too low may create:

```text
Alert fatigue
```

An alert set too high may fail to detect meaningful degradation.

Production threshold tuning should consider:

- Typical CPU baseline
- Expected workload peaks
- Duration of high utilization
- Business impact
- Application characteristics
- VM size
- Historical trends

---

# Alert Evaluation Considerations

The combination of:

```text
Threshold
Aggregation
Lookback
Evaluation Frequency
```

determines how sensitive an alert is.

For this lab:

```text
Threshold:
80%

Aggregation:
Average

Lookback:
5 minutes

Evaluation:
1 minute
```

was intentionally selected to create a sustained-condition alert.

---

# Monitoring Correlation

A strong alert investigation combines multiple sources.

For high CPU:

```text
Azure Monitor Alert
      ↓
Percentage CPU Metric
      ↓
Perf Guest Telemetry
      ↓
Windows Process Investigation
      ↓
Windows Event Logs
      ↓
Activity Log if Relevant
```

This creates a stronger root-cause investigation than relying on a single dashboard.

---

# Alert Evidence

Evidence for this phase is stored in:

```text
Screenshots/06-Alerts/
```

The complete screenshot sequence is:

```text
01-ALRT-AZMON-HIGH-CPU-Condition.png
02-ALRT-AZMON-HIGH-CPU-Action-Group.png
03A-ALRT-AZMON-HIGH-CPU-Review-Create-Top.png
03B-ALRT-AZMON-HIGH-CPU-Review-Create-Bottom.png
04-ALRT-AZMON-HIGH-CPU-Enabled.png
05-ALRT-AZMON-HIGH-CPU-Overview.png
06-AZMON-WIN01-Controlled-High-CPU-Started.png
07-AZMON-WIN01-High-CPU-Metric.png
08-ALRT-AZMON-HIGH-CPU-Fired.png
09-ALRT-AZMON-HIGH-CPU-Fired-Overview.png
10-AZMON-WIN01-High-CPU-Workers-Post-Test.png
11-AZMON-WIN01-High-CPU-Workload-Cleared.png
12-AZMON-WIN01-CPU-Recovery.png
13-ALRT-AZMON-HIGH-CPU-Resolved.png
14-ALRT-AZMON-HIGH-CPU-Resolved-Overview.png
```

There are 15 screenshot files because:

```text
03A
03B
```

are separate pieces of evidence.

---

# Evidence Progression

The screenshot sequence documents:

```text
Alert Design
      ↓
Notification Configuration
      ↓
Final Review
      ↓
Rule Enabled
      ↓
Alert Overview
      ↓
Workload Started
      ↓
CPU Increase
      ↓
Alert Fired
      ↓
Fired Alert Details
      ↓
Post-Test Worker Validation
      ↓
Workload Cleared
      ↓
CPU Recovery
      ↓
Alert Resolved
      ↓
Resolved Alert Details
```

---

# Incident Documentation

The complete controlled high CPU incident is documented in:

```text
Help-Desk-Tickets/INC-001-High-CPU.md
```

That incident contains:

- Environment
- Alert configuration
- Incident generation
- Symptoms
- Investigation
- Root cause
- Post-test process validation
- Remediation
- Recovery validation
- Evidence
- Lessons learned
- Final incident status

---

# Alert Security Considerations

Alert screenshots are reviewed before publication.

The GitHub version should hide or crop unnecessary values such as:

- Subscription IDs
- Azure account email
- Notification email address
- Browser address bar
- Azure resource URLs
- Public IP addresses
- Other unnecessary tenant identifiers

Useful technical evidence remains visible.

---

# Action Group Privacy

The Action Group screenshot may contain a notification email address.

The email address should be:

```text
Cropped
or
Redacted
```

before publication.

The Action Group name and notification type can remain visible.

---

# Cost Considerations

Azure Monitor alerts can create monitoring-related costs depending on alert type and usage.

This lab keeps the alert configuration intentionally limited.

Current alert implementation:

```text
1 primary metric alert
1 Action Group
1 monitored VM
```

The VM is deallocated when not actively required for lab work to reduce compute costs.

---

# Operational Benefits

The alert configuration provides several practical support benefits.

## Proactive Detection

Administrators do not need to continuously watch CPU charts.

---

## Consistent Threshold

The rule evaluates the same condition repeatedly.

---

## Faster Incident Awareness

The alert provides immediate operational context when the condition is met.

---

## Central Notification

The Action Group provides a reusable notification mechanism.

---

## Recovery Visibility

Automatic resolution provides evidence that the condition returned to normal.

---

# Troubleshooting Alert Rules

If an expected alert does not fire, investigate:

1. Alert rule enabled?
2. Correct resource scope?
3. Correct signal?
4. Correct threshold?
5. Metric data available?
6. Evaluation window long enough?
7. Aggregation correct?
8. Condition actually met?
9. Rule recently created?
10. Azure Monitor processing delay?

---

# Troubleshooting Notifications

If the alert fires but notification is not received:

1. Confirm Action Group association
2. Confirm notification type
3. Confirm email configuration
4. Confirm email verification if required
5. Check spam or junk folder
6. Review Action Group configuration
7. Test Action Group if available

This separates:

```text
Alert Detection Problem
```

from:

```text
Notification Delivery Problem
```

---

# Troubleshooting Resolution

If an alert remains Fired after the condition appears normal:

1. Review metric time range
2. Confirm latest metric samples
3. Review lookback period
4. Confirm automatic resolution enabled
5. Allow evaluation window to clear
6. Refresh alert state
7. Verify underlying condition is actually below threshold

---

# Lessons Learned

This phase demonstrated that successful monitoring requires more than simply creating an alert rule.

The complete process requires:

```text
Valid Metric
+
Correct Scope
+
Meaningful Threshold
+
Evaluation Window
+
Action Group
+
Incident Investigation
+
Recovery Validation
```

The controlled CPU incident also demonstrated why recovery evidence matters.

An alert firing proves detection.

An alert resolving proves that Azure Monitor also recognized that the abnormal condition ended.

The strongest portfolio evidence therefore includes both:

```text
Fired
```

and:

```text
Resolved
```

states.

---

# Key Technical Takeaways

## Metric Alerts

Metric alerts are well suited for numeric conditions such as CPU utilization.

---

## Action Groups

Action Groups centralize alert notifications.

---

## Threshold Windows

A lookback period helps distinguish sustained issues from instantaneous spikes.

---

## Automatic Resolution

Automatic resolution provides a complete incident lifecycle.

---

## Multiple Telemetry Sources

Metrics provide the detection signal, while guest telemetry supports deeper investigation.

---

## Controlled Fault Testing

A safe temporary workload provides a repeatable way to validate monitoring without creating a permanent system problem.

---

# Skills Demonstrated

This phase demonstrates:

- Microsoft Azure
- Azure Monitor
- Azure Monitor Alerts
- Azure Virtual Machines
- Metric alerts
- Percentage CPU monitoring
- Static thresholds
- Aggregation
- Evaluation frequency
- Lookback windows
- Azure Monitor Action Groups
- Email notification configuration
- Alert severity
- Automatic resolution
- Azure VM Run Command
- PowerShell
- Controlled fault simulation
- Performance troubleshooting
- Azure Monitor Agent
- Data Collection Rules
- Log Analytics
- KQL
- Incident detection
- Alert investigation
- Root-cause analysis
- Recovery validation
- Alert lifecycle management
- Technical documentation

---

# Phase Status

```text
High CPU Alert Rule: COMPLETE
Alert Rule Enabled: VERIFIED
Action Group: COMPLETE
Email Notification: CONFIGURED
CPU Threshold: VERIFIED
Controlled CPU Workload: COMPLETE
CPU Increase: VERIFIED
Alert Fired: VERIFIED
Fired Alert Investigation: COMPLETE
Root Cause: VERIFIED
Controlled Workload Termination: VERIFIED
CPU Recovery: VERIFIED
Automatic Alert Resolution: VERIFIED
Resolved Alert Investigation: COMPLETE
INC-001: CLOSED
```

This phase is complete and demonstrates the full Azure Monitor metric alert lifecycle from proactive detection through incident recovery and automatic resolution.
