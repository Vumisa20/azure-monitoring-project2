# Project 2 – Implementation Guide

## Purpose

This document records the main build and validation sequence used for the NationWall Azure monitoring project.

## Phase 1 – Monitoring Foundation

- Log Analytics workspace: `law-nationwall-project2`
- Region: `South Africa North`
- Monitored VM: `vm-nationwall-private`
- Data Collection Rules: performance counters and Syslog

## Phase 2 – Validate Data

### CPU trend
```kusto
Perf
| where TimeGenerated > ago(30m)
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 5m)
| order by TimeGenerated asc
```

### Disk trend
```kusto
Perf
| where TimeGenerated > ago(30m)
| where ObjectName == "Logical Disk"
| where CounterName == "% Used Space"
| summarize AvgDiskUsed = avg(CounterValue) by bin(TimeGenerated, 5m)
| order by TimeGenerated asc
```

### Failed-login event
```kusto
Syslog
| where Facility == "authpriv"
| where SyslogMessage has "Failed password"
| project TimeGenerated, Computer, ProcessName, SeverityLevel, SyslogMessage
| order by TimeGenerated desc
```

## Phase 3 – Alert Rules

- `NationWall-High-CPU` → `AvgCPU > 80`
- `NationWall-High-Disk` → `AvgDiskUsed > 80`
- `NationWall-Failed-Login` → `FailedLogins > 0`

## Phase 4 – Action Group

`AG-NationWall-Monitoring`

Configured with a verified email receiver and the Logic App action using Common Alert Schema.

## Phase 5 – Logic App

`logicapp-nationwall-project2`

```text
When an HTTP request is received
                |
                v
        Send an email (V2)
```

## Phase 6 – Validation

The Logic App Run history showed successful runs for both the HTTP trigger and email action. A controlled Linux authentication failure was also found in Syslog through KQL.

## Troubleshooting

The first single-event failed-login alert attempt failed because Azure reported that a query was missing. It was recreated as an aggregated log-search alert using `FailedLogins > 0`.

A captured Action Group sample test returned a failed/unknown result, while the Logic App subsequently showed successful workflow runs. This is retained as a troubleshooting note.

## Evidence Checklist

Capture:
1. Log Analytics workspace
2. CPU query results
3. Disk query results
4. Syslog results
5. CPU alert
6. Disk alert
7. Failed-login alert
8. Action Group
9. Logic App workflow
10. Successful Logic App Run history

## Handover

After Project 2 documentation is committed, start Project 3 with a clean Terraform repository and environment structure.
