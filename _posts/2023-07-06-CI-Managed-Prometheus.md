---
layout: post
title: Container Insights vs. Managed Prometheus
tags: 
- Azure Monitor
- Container Insights
- Prometheus
- Grafana
- Azure Monitor Workspace
- Log Analytics Workspace
---

Azure recently GA'd [Managed Prometheus](https://learn.microsoft.com/en-Us/azure/azure-monitor/essentials/prometheus-metrics-overview) and [Managed Grafana](https://learn.microsoft.com/en-us/azure/managed-grafana/overview). The monitoring landscape becomes confusing when additional services, such as [Azure Monitor Workspace](https://learn.microsoft.com/en-Us/azure/azure-monitor/essentials/azure-monitor-workspace-overview), come into play. Azure already has [Container Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-overview), [Log Analytics Workspaces](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview), and [Metrics Explorer](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/metrics-getting-started). 

What's the difference between all these and when should you choose one over the other? This document hopes to shed light on that.

## 