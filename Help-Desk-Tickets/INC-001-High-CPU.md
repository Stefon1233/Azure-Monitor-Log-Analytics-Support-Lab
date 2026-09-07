# INC-001 — High CPU Utilization on Azure Virtual Machine

## Incident Summary

**Incident ID:** INC-001  
**Title:** High CPU Utilization on AZMON-WIN01  
**Category:** Azure Monitor / Virtual Machine Performance  
**Severity:** Warning  
**Affected Resource:** AZMON-WIN01  
**Resource Group:** RG-AZMON-SUPPORT-LAB  
**Region:** North Central US  
**Detection Method:** Azure Monitor Metric Alert  
**Alert Rule:** ALRT-AZMON-HIGH-CPU  
**Status:** Recovery In Progress  

---

## Environment

The affected system is a Windows Server virtual machine deployed as part of the Azure Monitor + Log Analytics Support Lab.

### Virtual Machine

- Name: AZMON-WIN01
- Operating System: Windows Server 2022 Datacenter: Azure Edition
- VM Size: Standard_D2als_v6
- vCPUs: 2
- Memory: 4 GiB
- Virtual Network: VNET-AZMON-SUPPORT
- Subnet: SNET-AZMON-WORKLOAD
- Monitoring Agent: Azure Monitor Agent

### Monitoring Configuration

The VM is monitored through:

- Azure Monitor
- Azure Monitor Agent
- Data Collection Rule
- Log Analytics Workspace
- Azure platform metrics
- Azure Monitor alerts
- KQL queries

Monitoring pipeline:

```text
AZMON-WIN01
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Metrics / Logs / Alerts
```

---

## Alert Configuration

The following Azure Monitor metric alert was configured before the incident:

### Alert Rule

```text
ALRT-AZMON-HIGH-CPU
```

### Condition

```text
Signal:
Percentage CPU

Threshold Type:
Static

Aggregation:
Average

Operator:
Greater Than

Threshold:
80%

Evaluation Frequency:
1 minute

Lookback Window:
5 minutes
```

### Severity

```text
Severity 2 — Warning
```

### Action Group

```text
AG-AZMON-SUPPORT
```

### Automatic Resolution

Enabled.

The alert was configured to automatically resolve after CPU utilization returned below the configured threshold.

---

## Incident Objective

The purpose of this controlled incident was to validate the complete Azure Monitor alert lifecycle:

```text
Normal CPU
      ↓
Controlled CPU Workload
      ↓
CPU Utilization Increases
      ↓
Azure Monitor Metric Evaluation
      ↓
ALRT-AZMON-HIGH-CPU Fires
      ↓
Alert Investigation
      ↓
Workload Terminates
      ↓
CPU Returns to Baseline
      ↓
Alert Automatically Resolves
```

---

## Baseline Condition

Before generating the incident, Azure Monitor showed normal CPU utilization for AZMON-WIN01.

Baseline monitoring included:

- Percentage CPU
- VM Availability
- Network traffic
- Disk I/O
- Azure Monitor Agent heartbeat
- Guest performance counters

The VM was operational and Azure Monitor Agent communication had already been validated through Log Analytics.

---

## Pre-Incident Monitoring Validation

Before the CPU test, the following monitoring components were confirmed operational:

```text
Azure Monitor Agent: VERIFIED
DCR Association: VERIFIED
Heartbeat: VERIFIED
Performance Counters: VERIFIED
Windows Event Collection: VERIFIED
Log Analytics Ingestion: VERIFIED
```

This established a known-good monitoring baseline before intentionally generating the high CPU condition.

---

## Incident Generation

A controlled CPU workload was started remotely through Azure VM Run Command.

The workload was designed to:

- Generate sustained CPU utilization
- Run multiple PowerShell worker processes
- Automatically terminate after a maximum duration
- Avoid permanent operating system changes
- Provide enough sustained utilization for Azure Monitor alert evaluation

Two PowerShell worker processes were launched.

The controlled workload was configured with a maximum runtime of approximately 10 minutes.

---

## Controlled Workload Script

The incident used the following PowerShell-based workload:

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

The workload was intentionally temporary and designed only for controlled monitoring validation.

---

## Incident Symptoms

The controlled workload produced the expected monitoring symptoms.

Observed evidence included:

- CPU utilization increased significantly above baseline
- CPU utilization exceeded the configured 80% threshold
- Azure Monitor detected the sustained CPU condition
- ALRT-AZMON-HIGH-CPU entered the Fired state
- AZMON-WIN01 was identified as the affected resource
- The alert rule remained enabled and operational throughout the test

This confirmed that the Azure platform metric and metric-alert configuration were functioning as designed.

---

## Azure Platform Metric Investigation

Azure Monitor Percentage CPU was reviewed during the incident.

Metric:

```text
Percentage CPU
```

The metric chart showed a clear increase in processor utilization after the controlled PowerShell workload was launched.

This provided direct evidence that:

- The VM experienced a real CPU increase
- The condition was sustained long enough for alert evaluation
- The metric exceeded the configured alert threshold

---

## Log Analytics Performance Investigation

Guest performance telemetry was also available through:

```text
LAW-AZMON-SUPPORT-LAB
```

The following KQL query can be used to investigate guest CPU telemetry:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where CounterName contains "% Processor Time"
| project TimeGenerated, Computer, ObjectName, CounterName, CounterValue
| order by TimeGenerated desc
```

This provides guest operating system performance evidence independently of the Azure host metric chart.

---

## Azure Monitor Alert Investigation

The configured metric alert successfully entered the Fired state.

Alert:

```text
ALRT-AZMON-HIGH-CPU
```

Monitor condition:

```text
Fired
```

The fired alert confirmed that:

- Percentage CPU crossed the configured threshold
- Azure Monitor evaluated the metric successfully
- The alert rule was active
- AZMON-WIN01 was correctly scoped
- The alert lifecycle was functioning
- The action group was associated with the rule

This completed the detection phase of the incident.

---

## Root Cause

The high CPU condition was caused by an intentionally generated PowerShell workload created specifically for Azure Monitor alert validation.

The controlled workload launched two PowerShell worker processes running:

```text
C:\Temp\AZMON-HighCPU.ps1
```

Evidence collected during the incident confirmed:

- The controlled workload was successfully started
- Azure Monitor recorded a significant CPU increase
- CPU utilization exceeded the configured 80% threshold
- ALRT-AZMON-HIGH-CPU entered the Fired state
- AZMON-WIN01 was identified as the affected resource

The workload was configured with a maximum runtime of approximately 10 minutes to prevent an uncontrolled or persistent resource-consumption condition.

By the time post-incident process validation was performed, no AZMON-HighCPU.ps1 worker processes remained.

This indicates that the controlled workload had already reached its configured timeout and terminated automatically.

The root cause was therefore confirmed as:

```text
Controlled PowerShell CPU workload
C:\Temp\AZMON-HighCPU.ps1
```

---

## Post-Test Process Validation

After the alert fired, the VM was checked for remaining controlled CPU worker processes.

The validation returned:

```text
Workers detected: 0
```

This result does not indicate that the incident failed.

The workload had already:

1. Started successfully
2. Increased VM CPU utilization
3. Exceeded the alert threshold
4. Triggered ALRT-AZMON-HIGH-CPU
5. Reached its configured maximum runtime
6. Terminated automatically

The post-test process check therefore confirmed that no controlled CPU workload remained active on the VM.

---

## Remediation

No active CPU worker processes remained when remediation validation was performed.

The controlled CPU workload was configured with a 10-minute maximum runtime. By the time post-incident process validation was performed, no AZMON-HighCPU.ps1 worker processes remained.

The workload had therefore already terminated automatically, and Azure Monitor metrics were used to validate recovery.

Post-workload validation included:

1. Checked for AZMON-HighCPU.ps1 worker processes
2. Confirmed zero controlled worker processes remained
3. Reviewed Percentage CPU in Azure Monitor
4. Monitored CPU utilization for return toward baseline
5. Monitored ALRT-AZMON-HIGH-CPU for automatic resolution
6. Verified the virtual machine remained operational

The remediation method for this controlled incident was therefore:

```text
Automatic workload timeout
+
Post-incident process validation
+
Azure Monitor recovery validation
```

No permanent VM configuration changes were required.

---

## Recovery Validation

The recovery phase validates that the monitoring condition clears after the controlled workload terminates.

Validation includes:

- Controlled PowerShell workers no longer running
- Percentage CPU decreases from the incident peak
- CPU utilization returns below the alert threshold
- VM remains available
- Azure Monitor Agent continues reporting
- Performance telemetry continues arriving
- Alert condition automatically resolves

The CPU recovery metric provides evidence of the transition:

```text
High CPU
      ↓
Controlled workload terminates
      ↓
CPU utilization decreases
      ↓
Threshold condition clears
      ↓
Azure Monitor resolves alert
```

---

## Monitoring Pipeline Validation

This incident validated multiple layers of the Azure monitoring environment.

### Azure Platform Monitoring

```text
AZMON-WIN01
      ↓
Percentage CPU
      ↓
Azure Monitor
      ↓
ALRT-AZMON-HIGH-CPU
```

### Guest Telemetry

```text
AZMON-WIN01
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
      ↓
Perf
```

Together these provide both Azure platform-level and guest operating system performance visibility.

---

## Evidence

Screenshot evidence collected during the incident includes:

```text
Screenshots/06-Alerts/

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

---

## Incident Timeline

```text
Monitoring baseline established
      ↓
High CPU alert rule configured
      ↓
Controlled CPU workload launched
      ↓
CPU utilization increased
      ↓
Percentage CPU exceeded 80%
      ↓
ALRT-AZMON-HIGH-CPU fired
      ↓
Alert evidence captured
      ↓
10-minute controlled workload timeout reached
      ↓
Worker validation returned zero active workers
      ↓
CPU recovery monitoring initiated
      ↓
Awaiting automatic alert resolution
```

---

## Preventive and Operational Considerations

In a production environment, sustained high CPU could indicate:

- Runaway application processes
- Resource-intensive scheduled tasks
- Malware or unwanted processes
- Poorly optimized applications
- Insufficient VM sizing
- Increased user or application load
- Background maintenance activity

A production response could include:

- Identifying top CPU-consuming processes
- Reviewing recent application changes
- Reviewing scheduled tasks
- Reviewing Windows Event Logs
- Examining historical CPU trends
- Scaling the VM if sustained utilization is legitimate
- Creating additional alert thresholds
- Investigating workload optimization opportunities

In this lab, however, the CPU increase was intentionally generated and the exact source was known.

---

## Lessons Learned

This incident demonstrated the importance of establishing a monitoring baseline before generating a test condition.

Because Azure Monitor Agent, DCR association, Log Analytics ingestion, and platform metrics had already been validated, the high CPU condition could be investigated without uncertainty about 
whether the monitoring platform itself was functioning.

The incident also demonstrated the difference between:

```text
Metric collection
Alert detection
Root-cause identification
Remediation
Recovery validation
Alert resolution
```

Each stage requires separate evidence.

The automatic timeout built into the workload also prevented the controlled test from becoming a persistent resource-consumption problem.

---

## Skills Demonstrated

This incident demonstrates:

- Microsoft Azure
- Azure Monitor
- Azure Virtual Machines
- Azure platform metrics
- Percentage CPU analysis
- Azure Monitor alert rules
- Static threshold alerts
- Action groups
- Azure Monitor Agent
- Data Collection Rules
- Log Analytics
- KQL
- Performance troubleshooting
- PowerShell
- Azure Run Command
- Controlled fault simulation
- Incident detection
- Incident investigation
- Root-cause analysis
- Post-incident validation
- Alert lifecycle management
- Technical documentation

---

## Current Incident Status: Resolved

```text
Controlled workload: COMPLETED
CPU metric increase: VERIFIED
Alert state: RESOLVED
Root cause: CONTROLLED CPU WORKLOAD
Remediation: WORKLOAD TERMINATED AUTOMATICALLY
Alert resolution: VERIFIED
Incident Status: CLOSED

```

The incident remains open until Azure Monitor confirms CPU recovery and ALRT-AZMON-HIGH-CPU automatically transitions from Fired to Resolved.
