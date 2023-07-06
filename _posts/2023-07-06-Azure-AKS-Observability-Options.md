---
layout: post
title: Azure AKS Observability Options
tags: 
- Azure Monitor
- Container Insights
- Prometheus
- Grafana
- Azure Monitor Workspace
- Log Analytics Workspace
---

Azure recently GA'd [Managed Prometheus](https://learn.microsoft.com/en-Us/azure/azure-monitor/essentials/prometheus-metrics-overview) and [Managed Grafana](https://learn.microsoft.com/en-us/azure/managed-grafana/overview) to support AKS observability.These new services are coupled with the [Azure Monitor Workspace](https://learn.microsoft.com/en-Us/azure/azure-monitor/essentials/azure-monitor-workspace-overview). Azure already has [Container Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-overview), [Log Analytics Workspaces](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview), and [Metrics Explorer](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/metrics-getting-started) for AKS observability. This begs the question. What's the difference between these options and when should you choose one over other?

## Overview

This post is focused on AKS monitoring. While Prometheus metrics can be gathered from other systems, Azure Monitor Workspace only supports various Kubernetes deployment types at the time of this document's writing.

Below is a comparison of the three options for monitoring AKS.

1. No observability
2. Container Insights only
3. Managed Prometheus only

You can deploy Container Insights and Prometheus at the same time. However, this article won't delve into that space. It's safe to assume deploying Container Insights and Prometheus side-by-side will give you the combined functionality of each solution listed below.

| Topic | No Observability | Container Insights | Managed Prometheus | Notes |
|---|---|---|---|---|
| Information Collected | Metrics | Metrics and Logs | Metrics | |
| Metric Granularity | [Basic](#metrics-collected---no-observability) | [Basic](#metrics-collected---container-insights) | [Extensive](#metrics-collected---managed-prometheus) | |
| Metric Configurability | None | [Basic](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-agent-config) - 1 ConfigMap | [Extensive](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/prometheus-metrics-scrape-configuration) - 4 ConfigMaps | |
| Metric Target | Metrics Explorer | Metrics Explorer | Azure Monitor Workspace | |
| Logs | None | [Container, OS, AKS, and Syslog](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-overview) | None | [Log - Container Insights](#logs---container-insights) |
| Logs Target | Metrics Explorer | Metrics Explorer, Log Analytics Workspace | Azure Monitor Workspace | Container Insights may support other targets in the future. |
| Deployment | Default | [Container Insights](#deployment---container-insights) | [Managed Prometheus](#deployment---managed-prometheus) | |
| Data Retention | Metrics - [93 Days](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-platform-metrics#retention-of-metrics) | Metrics - [93 days](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/data-platform-metrics#retention-of-metrics)</br>Logs - [Configurable up to 2 years interactive. 7 years archive.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-retention-archive) | Metrics - [18 months](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/azure-monitor-workspace-overview#data-considerations) | |
| Alert Support | Yes | Yes | Yes - Alerting done within Managed Prometheus | |
| Private Network Support | Metrics - No. A public endpoint is used. | Metrics - No. A public endpoint is used.</br>Logs - Yes, via [AMPLS](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/private-link-security). | TODO | |
| Visualization | Metrics Explorer, Workbooks, Dashboards | Metrics Explorer, Workbooks, Dashboards | Workbooks, Dashboards, Managed Grafana | |
| Cost | None | Logs - Ingestion, queries, storage | Metrics - Ingestion, queries, storage | [See the pricing calculator](https://aka.ms/pricing) for details. There is a lot of cost management options available in Container Insights and Managed Prometheus |

## Basic Decision Tree

There are some basic guidelines around when to use Container Insights, when to use Managed Prometheus, and when to use both.

You should use Container Insights if...

- You need logs from your Kubernetes cluster
- You don't need detailed metrics beyond what Container Insights supplies

You should consider Managed Prometheus if...

- You need detailed Kubernetes metrics
- You need metric data retention beyond that offered Azure Metrics Explorer
- You are comfortable paying for Managed Grafana to visualize your data
- You already have a great deal of infrastructure managed by OSS Prometheus and visualized in Grafana

In general, Managed Prometheus is your go-to for complex metrics requirements from Kubernetes. Azure currently supports Managed Prometheus for various incarnations of Kubernetes, only. While OSS Prometheus supports many [third-party exporters](https://prometheus.io/docs/instrumenting/exporters/) to gather metrics from popular products like Cassandra, MySQL, and MongoDB, it is unknown if Managed Prometheus will eventually support these.

Consider the additional cost of Managed Grafana if you need to visualize your Managed Prometheus metrics. Managed Grafana is a separate service. However, Managed Grafana does a wonderful job of not only visualizing metrics, but also logs, including those from a Log Analytics Workspace. There are countless free dashboards available for Grafana.

If you only need to alert on metrics, native Azure Alerting is integrated into Managed Prometheus. 

## Metrics Collected - No Observability

Below is a list of metrics created when no observability solution is deployed to AKS as of the date this article was published. These metrics will within Azure Metrics Explorer. There is a heavy overlap of these metrics compared to the metrics sent to Metrics Explorer by Container Insights. The main difference is that Container Insights metrics appear within their own namespace and have a more obscure naming convention.

- API Server (Preview)
  - Inflight Requests
- Cluster Autoscaler (Preview)
  - Cluster Health
  - Scale Down Cooldown
  - Unneeded Nodes
  - Unscheduled Pods
- Nodes
  - Statuses for various node conditions
  - Total amount of available memory in a managed cluster
  - Total number of available CPU cores in a managed cluster
- Nodes (Preview)
  - CPU Usage Millicores
  - CPU Usage Percentage
  - Disk Used Bytes
  - Disk Used Percentage
  - Memory RSS Bytes
  - Memory RSS Percentage
  - Memory Working Set Bytes
  - Memory Working Set Percentage
  - Network In Bytes
  - Network Out Bytes
- PODs
  - Number of pods by phase
- Number of pods in Ready state


## Metrics Collected - Container Insights

The following metrics are collected by Container Insights and appear within Azure Metrics Explorer. Note that there is overlap with the metrics collected when [no observability](#metrics-collected---no-observability) exists on the AKS cluster. However, the names are different.

Items with "preview" appended are metrics currently in preview at the time of this document's writing.

- insights.container/pods
  - completedJobsCount (Preview)
  - podCount
  - podReadyPercentage (Preview)
  - retartingContainerCount (Preview)
- insights.container/containers
  - cpuExceededpercentage (Preview)
  - cpuThresholdViolated (Preview)
  - memoryRssExceededPercentage (Preview)
  - memoryRssthresholdViolated (Preview)
  - memoryWorkingSetExceededPercentage (Preview)
  - memoryWorkingSetThresholdViolated (Preview)
- insights.container/nodes
  - cpuUsageAllocatablePercentage (Preview)
  - cpuUsageMillicores
  - cpuUsagePercentage
  - diskUsedPercentage (Preview)
  - memoryRssAllocatablePercentage (Preview)
  - memoryRssBytes
  - memoryRssPercentage
  - memoryWorkingSetAllocatablePercentage
  - memoryWorkingSetBytes
  - memoryWorkingSetPercentage
  - nodesCount
- insights.containers/persistentvolumes
  - pvUsageExceededPercentage (Preview)
  - pvUsageThresholdViolated (Preview)

## Metrics Collected - Managed Prometheus

The metrics collected by Prometheus are too extensive to list here. Please, see Microsoft's [default Prometheus metrics configuration](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/prometheus-metrics-scrape-default) documentation to see what is available.

## Logs - Container Insights

The following tables are populated in Log Analytics Workspace when Container Insights is deployed. This list of tables assumes 

- ContainerInventory
- ContainerNodeInventory
- Heartbeat
    - This table is not really Container Insights-centric. It stores heartbeats from any Azure Monitor agent sending data to the Log Analytics Workspace.
- InsightsMetrics
- KubeNodeInventory
- KubePodInventory
- KubeServices
- Perf
- Syslog
    - Populated only when Syslog is populated in the DCR

## Deployment - Container Insights

Container Insights is deployed via a Data Collection Rule.

## Deployment - Managed Prometheus

Managed Prometheus is deployed via a Data Collection Rule using the "Microsoft-PrometheusMetrics" stream, includes a lableIncludeFilter option on the stream, and only supports a destination type of "monitoringAccounts" (a.k.a. Azure Monitor Workspace).

The DCR requires its own Data Collection Endpoint.

The DCR association is done on the Kubernetes service. It is not done on the underlying compute. This is important to understand as AKS is backed by VMSS. The Azure Monitor Agent, which monitors a VM or VMSS, requires a DCR association between the DCR and the VM or VMSS. Managed Prometheus creates an association between the DCR and the Kubernetes service.