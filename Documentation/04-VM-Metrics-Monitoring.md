# Azure Monitor + Log Analytics Support Lab

## 04 — VM Metrics Monitoring

## Overview

This phase of the Azure Monitor + Log Analytics Support Lab focuses on Azure virtual machine platform metrics.

Azure Monitor provides platform-level metrics for Azure virtual machines without requiring guest operating system instrumentation for basic host monitoring.

The monitored virtual machine in this lab is:

```text
AZMON-WIN01
```

The primary objective of this phase was to establish a performance baseline and validate visibility into:

```text
CPU
Network
Disk I/O
VM Availability
```

These metrics provide the first layer of operational monitoring before deeper guest telemetry, KQL investigation, and alert-based incident response are used.

---

## Objectives

The objectives of this phase were to:

- Review the Azure Monitor experience for AZMON-WIN01
- Establish a VM monitoring baseline
- Review Percentage CPU
- Review network traffic
- Review disk I/O
- Review VM availability
- Compare host metrics with guest telemetry
- Use metrics during a controlled high CPU incident
- Confirm CPU recovery after the workload ended
- Document the role of platform metrics during troubleshooting

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

### Virtual Network

```text
VNET-AZMON-SUPPORT
```

### Subnet

```text
SNET-AZMON-WORKLOAD
10.20.1.0/24
```

### Log Analytics Workspace

```text
LAW-AZMON-SUPPORT-LAB
```

### Data Collection Rule

```text
DCR-AZMON-WINDOWS
```

---

## Azure Monitor VM Baseline

Before incident generation, the AZMON-WIN01 monitoring experience was reviewed to establish a known-good baseline.

The Azure Monitor page showed host monitoring data and resource health information for the VM.

The initial monitoring baseline was captured before generating troubleshooting conditions.

Baseline evidence included:

- VM operational state
- Availability
- CPU activity
- Azure Monitor integration
- Monitoring options
- Guest telemetry availability

This baseline provided a reference point for later incident analysis.

---

## Platform Metrics vs Guest Telemetry

This lab intentionally uses both Azure platform metrics and guest operating system telemetry.

### Azure Platform Metrics

Platform metrics are available directly from the Azure resource.

Examples used in this phase include:

```text
Percentage CPU
Network In Total
Network Out Total
Disk Read Bytes
Disk Write Bytes
VM Availability Metric
```

These metrics provide resource-level visibility.

---

## Guest Operating System Telemetry

Guest telemetry is collected through:

```text
AZMON-WIN01
      ↓
Azure Monitor Agent
      ↓
DCR-AZMON-WINDOWS
      ↓
LAW-AZMON-SUPPORT-LAB
```

Guest performance data is stored in:

```text
Perf
```

This allows the lab to compare:

```text
Azure platform metric
vs
Windows guest performance counter
```

Both sources are useful during troubleshooting.

---

## Metrics Investigation Workflow

The general VM metrics troubleshooting workflow used in this lab is:

```text
Monitoring Alert or User Report
      ↓
Open AZMON-WIN01
      ↓
Azure Monitor Metrics
      ↓
Select Relevant Metric
      ↓
Set Investigation Time Range
      ↓
Identify Abnormal Trend
      ↓
Correlate with Guest Telemetry
      ↓
Investigate Root Cause
      ↓
Remediate
      ↓
Verify Metric Recovery
```

---

# Percentage CPU

## Purpose

Percentage CPU is one of the primary performance metrics used to identify processor pressure.

The metric can help identify:

- Sustained processor utilization
- Sudden CPU spikes
- Performance degradation
- Resource-intensive workloads
- Potential VM sizing issues
- Background processes consuming excessive CPU

---

## CPU Baseline

The Percentage CPU metric was reviewed before the controlled high CPU incident.

Configuration:

```text
Scope:
AZMON-WIN01

Metric Namespace:
Virtual Machine Host

Metric:
Percentage CPU

Aggregation:
Average
```

The time range was adjusted to provide a focused operational view.

The baseline showed normal processor utilization before the controlled workload was launched.

---

## Percentage CPU Troubleshooting Interpretation

CPU monitoring should not rely only on a single instantaneous value.

Important considerations include:

- Duration of the CPU increase
- Frequency of CPU spikes
- Average CPU over the evaluation window
- Whether CPU returns to normal
- Whether users or services are impacted
- Whether other metrics also change

A short spike may be normal.

Sustained high CPU may indicate a problem requiring investigation.

---

## High CPU Incident Integration

Percentage CPU later became the primary signal for:

```text
INC-001 — High CPU Utilization
```

The alert rule used:

```text
ALRT-AZMON-HIGH-CPU
```

Condition:

```text
Percentage CPU
Average
Greater than 80%
5-minute lookback
1-minute evaluation frequency
```

The controlled PowerShell workload generated enough processor utilization to cause the alert to fire.

This demonstrated the relationship between:

```text
Metric Collection
      ↓
Threshold Evaluation
      ↓
Azure Monitor Alert
      ↓
Incident Investigation
```

---

# Network Metrics

## Purpose

Azure network metrics provide visibility into traffic entering and leaving the virtual machine.

The metrics used were:

```text
Network In Total
Network Out Total
```

Aggregation:

```text
Sum
```

---

## Network In Total

Network In Total measures network traffic received by the VM.

This can help investigate:

- Increased inbound traffic
- Application communication
- File transfer activity
- Unexpected traffic patterns
- Network-intensive workloads

---

## Network Out Total

Network Out Total measures network traffic transmitted by the VM.

This can help investigate:

- Outbound application traffic
- File transfers
- Monitoring traffic
- Update activity
- Unexpected egress behavior

---

## Combined Network View

Both network metrics were displayed on the same chart.

This makes it easier to compare:

```text
Inbound Traffic
vs
Outbound Traffic
```

The combined chart provides a visual baseline for future network troubleshooting scenarios.

---

## Network Troubleshooting Model

```text
Connectivity or Performance Issue
      ↓
Review Network In / Network Out
      ↓
Identify Traffic Trend
      ↓
Compare with Incident Timestamp
      ↓
Review NSG / NIC / Application State
      ↓
Correlate with Guest Telemetry
```

Azure platform network metrics do not replace packet-level troubleshooting, but they provide useful context.

---

# Disk I/O Metrics

## Purpose

Disk metrics were reviewed to establish visibility into storage activity for AZMON-WIN01.

The metrics used were:

```text
Disk Read Bytes
Disk Write Bytes
```

Aggregation:

```text
Sum
```

---

## Disk Read Bytes

Disk Read Bytes provides visibility into data read from VM disks.

A significant increase may indicate:

- Application workload
- File scanning
- Software installation
- Updates
- Data processing
- Heavy read activity

---

## Disk Write Bytes

Disk Write Bytes provides visibility into data written to VM disks.

A significant increase may indicate:

- Application logging
- Software installation
- Update activity
- File creation
- Data processing
- Temporary workload activity

---

## Combined Disk View

Disk Read Bytes and Disk Write Bytes were displayed together.

This provides a simple comparison between:

```text
Read Activity
vs
Write Activity
```

The disk metrics establish a baseline that can later be compared against abnormal storage conditions.

---

## Guest Disk Telemetry

Azure host disk metrics can also be compared with guest counters collected through the Data Collection Rule.

Guest counters include:

```text
\LogicalDisk(_Total)\Avg. Disk Queue Length
```

and:

```text
\LogicalDisk(_Total)\Free Megabytes
```

This creates two complementary views:

```text
Azure Platform Disk I/O
+
Windows Guest Disk Performance
```

---

# VM Availability Metric

## Purpose

The VM Availability Metric provides a direct signal indicating whether the VM is considered available.

This metric is particularly useful for:

- Availability monitoring
- Outage investigation
- Planned stop/start testing
- Correlating outages with Activity Log events
- Availability alerting

---

## Availability Baseline

The VM Availability Metric was reviewed while AZMON-WIN01 was operational.

The metric provided a baseline indicating that the VM was available.

This baseline will later support:

```text
INC-003 — VM Availability
```

A future controlled availability incident can compare:

```text
Availability Metric
Heartbeat
AzureActivity
Azure Monitor Alert
```

---

## Availability Investigation Model

```text
VM Becomes Unavailable
      ↓
VM Availability Metric Changes
      ↓
Heartbeat Stops or Changes
      ↓
AzureActivity Reviewed
      ↓
Administrative Operation Identified
      ↓
Root Cause Confirmed
```

This correlation is stronger than relying on one telemetry source alone.

---

# Metric Time Ranges

The lab uses focused time ranges during troubleshooting.

Common examples include:

```text
Last 30 minutes
Last 1 hour
Last 24 hours
```

A shorter time range is useful during controlled incidents because it makes the event easier to identify visually.

A wider range is useful for:

- Historical comparison
- Trend identification
- Recurring issues
- Baseline analysis

---

# Aggregation

Different metrics require different aggregation types.

Examples used in this lab include:

```text
Percentage CPU:
Average
```

```text
Network In Total:
Sum
```

```text
Network Out Total:
Sum
```

```text
Disk Read Bytes:
Sum
```

```text
Disk Write Bytes:
Sum
```

```text
VM Availability Metric:
Average
```

Choosing the correct aggregation is important because it affects how the metric is interpreted.

---

# Metrics and Alerting

Metrics become operationally useful when combined with Azure Monitor alerts.

In this lab:

```text
Percentage CPU
```

was connected to:

```text
ALRT-AZMON-HIGH-CPU
```

Alert lifecycle:

```text
CPU Baseline
      ↓
CPU Exceeds 80%
      ↓
Azure Monitor Evaluates Condition
      ↓
Alert Fires
      ↓
Incident Investigation
      ↓
Workload Ends
      ↓
CPU Recovers
      ↓
Alert Resolves
```

This demonstrated that metrics can be used not only for visual analysis but also for automated incident detection.

---

# CPU Recovery Validation

After the controlled high CPU workload ended, Percentage CPU was reviewed again.

The chart showed:

```text
Normal Baseline
      ↓
CPU Increase
      ↓
High CPU Period
      ↓
CPU Drop
      ↓
Return Toward Baseline
```

This provided evidence that:

- The abnormal workload had ended
- Processor pressure had decreased
- The VM returned toward its normal performance state
- The alert condition was no longer present

The CPU recovery chart was important evidence for closing INC-001.

---

# Metric Correlation with KQL

Azure platform metrics can be compared with guest performance telemetry collected in Log Analytics.

Example KQL query:

```kusto
Perf
| where TimeGenerated > ago(30m)
| where Computer =~ "AZMON-WIN01"
| where CounterName contains "% Processor Time"
| project TimeGenerated, Computer, ObjectName, CounterName, CounterValue
| order by TimeGenerated desc
```

This provides Windows guest CPU telemetry.

The investigation can therefore compare:

```text
Azure Percentage CPU Metric
+
Perf % Processor Time
```

This improves confidence during root-cause analysis.

---

# Metric Correlation with Activity Log

Metrics can also be correlated with Azure administrative operations.

Example scenario:

```text
VM Availability Drops
      ↓
Check VM Availability Metric
      ↓
Review AzureActivity
      ↓
Find VM Stop / Deallocate Operation
      ↓
Confirm Control-Plane Root Cause
```

This type of correlation will be used later in the lab.

---

# Metric Correlation with Alerts

Metric alerting creates another layer of evidence.

Example:

```text
Percentage CPU
      ↓
ALRT-AZMON-HIGH-CPU
      ↓
Fired
      ↓
Incident Investigation
      ↓
Resolved
```

The alert provides:

- Detection timestamp
- Severity
- Affected resource
- Condition
- Monitor state

The metric chart provides the performance trend surrounding that event.

---

# Troubleshooting CPU Issues

A high CPU investigation can include:

1. Confirm the metric increase
2. Identify the incident time window
3. Determine whether CPU is sustained or temporary
4. Review the alert condition
5. Review guest performance data
6. Identify CPU-intensive processes
7. Review Windows events if relevant
8. Remediate the workload
9. Confirm CPU recovery
10. Confirm alert resolution

---

# Troubleshooting Network Issues

A network investigation can include:

1. Review Network In Total
2. Review Network Out Total
3. Compare traffic before and during the incident
4. Check VM availability
5. Check NSG configuration
6. Check NIC configuration
7. Check guest networking
8. Review application connectivity
9. Compare with Azure Activity Log changes

---

# Troubleshooting Disk Issues

A disk investigation can include:

1. Review Disk Read Bytes
2. Review Disk Write Bytes
3. Review guest disk queue length
4. Review free disk space
5. Check Windows Event Logs
6. Identify storage-intensive processes
7. Compare with workload activity
8. Validate recovery after remediation

---

# Troubleshooting Availability Issues

A VM availability investigation can include:

1. Review VM Availability Metric
2. Review VM power state
3. Query Heartbeat
4. Review AzureActivity
5. Check Azure Monitor alerts
6. Identify recent administrative operations
7. Confirm recovery
8. Validate agent communication

---

# Platform Metric Advantages

Azure platform metrics provide several benefits:

- Available directly from the Azure resource
- Useful before guest monitoring is configured
- Low-latency operational visibility
- Useful for dashboards and alerts
- Easy historical visualization
- Useful for incident correlation

They provide a fast first step during troubleshooting.

---

# Platform Metric Limitations

Platform metrics do not provide every detail required for root-cause analysis.

For example, Percentage CPU can show:

```text
CPU is high
```

but it does not directly identify:

```text
Which Windows process caused the high CPU?
```

For deeper investigation, additional telemetry may be required:

```text
Azure Monitor Agent
Log Analytics
Perf
Event
PowerShell
Windows tools
```

This is why the lab combines platform and guest monitoring.

---

# Monitoring Layers

The lab now has multiple monitoring layers:

```text
Layer 1:
Azure Platform Metrics

Layer 2:
Azure Monitor Agent

Layer 3:
Data Collection Rule

Layer 4:
Log Analytics

Layer 5:
Azure Monitor Alerts

Layer 6:
Activity Log

Layer 7:
KQL Investigation
```

These layers provide complementary visibility.

---

# Screenshot Evidence

Evidence for this phase is stored in:

```text
Screenshots/04-VM-Metrics/
```

Screenshot sequence:

```text
01-AZMON-WIN01-Monitor-Baseline.png
02-AZMON-WIN01-CPU-Percentage-Metric.png
03-AZMON-WIN01-Network-Traffic-Metrics.png
04-AZMON-WIN01-Disk-IO-Metrics.png
05-AZMON-WIN01-Availability-Metric.png
```

Additional CPU incident evidence is stored in:

```text
Screenshots/06-Alerts/
```

Relevant recovery evidence includes:

```text
06-AZMON-WIN01-Controlled-High-CPU-Started.png
07-AZMON-WIN01-High-CPU-Metric.png
12-AZMON-WIN01-CPU-Recovery.png
```

---

# Incident Integration

The VM metrics phase directly supports:

```text
INC-001 — High CPU Utilization
```

and will later support:

```text
INC-003 — VM Availability
```

Potential future incidents can also use these metrics for:

- Network performance issues
- Disk performance issues
- Resource sizing analysis
- Availability troubleshooting

---

# Security Considerations

Metric screenshots are reviewed before GitHub publication.

The portfolio copy should avoid unnecessary exposure of:

- Subscription IDs
- User account identities
- Public IP addresses
- Browser URLs containing Azure resource identifiers

Lab resource names remain visible because they demonstrate the architecture.

---

# Cost Considerations

Azure platform metrics provide monitoring visibility without requiring broad guest telemetry collection.

This lab also limits guest performance collection to a small number of counters.

This approach reduces unnecessary ingestion while preserving useful troubleshooting data.

The VM itself is stopped and deallocated when not required for active lab work to reduce compute cost.

---

# Lessons Learned

This phase demonstrated that Azure platform metrics provide immediate operational visibility into the virtual machine.

Metrics such as:

```text
Percentage CPU
Network In Total
Network Out Total
Disk Read Bytes
Disk Write Bytes
VM Availability Metric
```

can quickly identify abnormal resource behavior.

However, metrics alone do not always identify the root cause.

The strongest troubleshooting workflow combines:

```text
Platform Metrics
+
Guest Performance Counters
+
Windows Event Logs
+
Azure Activity Log
+
Azure Monitor Alerts
+
KQL
```

The controlled high CPU incident demonstrated this layered monitoring approach in practice.

---

# Skills Demonstrated

This phase demonstrates:

- Microsoft Azure
- Azure Monitor
- Azure Virtual Machines
- VM metrics
- Percentage CPU monitoring
- Network monitoring
- Disk I/O monitoring
- VM availability monitoring
- Metric aggregation
- Monitoring baselines
- Performance troubleshooting
- Azure Monitor Agent
- Log Analytics
- KQL correlation
- Alert correlation
- Incident investigation
- Recovery validation
- Root-cause analysis
- Technical documentation

---

# Phase Status

```text
VM Monitoring Baseline: COMPLETE
Percentage CPU Metric: VERIFIED
Network Metrics: VERIFIED
Disk I/O Metrics: VERIFIED
VM Availability Metric: VERIFIED
High CPU Incident Integration: COMPLETE
CPU Recovery Validation: COMPLETE
Metric / KQL Correlation: VERIFIED
Metric / Alert Correlation: VERIFIED
```

This phase is complete and provides a validated Azure VM platform-metrics baseline for the remaining alerting, Activity Log, and root-cause investigation scenarios.
