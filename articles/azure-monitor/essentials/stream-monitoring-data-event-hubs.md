---
title: Stream Azure monitoring data to an event hub and external partners
description: Learn how to stream your Azure monitoring data to an event hub to get the data into a partner SIEM or analytics tool.
services: azure-monitor
author: bwren
ms.author: bwren
ms.topic: conceptual
ms.date: 07/15/2020
ms.reviewer: lualderm
---
# Stream Azure monitoring data to an event hub or external partner

Azure Monitor provides full stack monitoring for applications and services in Azure, in other clouds, and on-premises. In most cases, the most effective method to stream monitoring data to external tools is by using [Azure Event Hubs](../../event-hubs/index.yml). This article provides a brief description on how to stream data and then lists some of the partners where you can send it. Some partners have special integration with Azure Monitor and might be hosted on Azure.

## Create an Event Hubs namespace

Before you configure streaming for any data source, you need to [create an Event Hubs namespace and event hub](../../event-hubs/event-hubs-create.md). This namespace and event hub is the destination for all of your monitoring data. An Event Hubs namespace is a logical grouping of event hubs that share the same access policy, much like a storage account has individual blobs within that storage account. Consider the following details about the Event Hubs namespace and event hubs that you use for streaming monitoring data:

* The number of throughput units allows you to increase throughput scale for your event hubs. Only one throughput unit is typically necessary. If you need to scale up as your log usage increases, you can manually increase the number of throughput units for the namespace or enable auto inflation.
* The number of partitions allows you to parallelize consumption across many consumers. A single partition can support up to 20 MBps or approximately 20,000 messages per second. Depending on the tool consuming the data, it might or might not support consuming from multiple partitions. Four partitions are reasonable to start with if you're not sure about the number of partitions to set.
* You set message retention on your event hub to at least seven days. If your consuming tool goes down for more than a day, this retention ensures that the tool can pick up where it left off for events up to seven days old.
* You should use the default consumer group for your event hub. There's no need to create other consumer groups or use a separate consumer group unless you plan to have two different tools consume the same data from the same event hub.
* For the Azure activity log, you pick an Event Hubs namespace, and Azure Monitor creates an event hub within that namespace called _insights-logs-operational-logs_. For other log types, you can either choose an existing event hub or have Azure Monitor create an event hub per log category.
* Outbound port 5671 and 5672 must typically be opened on the computer or virtual network consuming data from the event hub.

## Monitoring data available
[Sources of monitoring data for Azure Monitor](../data-sources.md) describes the data tiers for Azure applications and the kinds of data available for each. The following table lists each of these tiers and a description of how that data can be streamed to an event hub. Follow the links provided for further detail.

| Tier | Data | Method |
|:---|:---|:---|
| [Azure tenant](../data-sources.md#azure-tenant) | Azure Active Directory audit logs | Configure a tenant diagnostic setting on your Azure Active Directory tenant. For more information, see [Tutorial: Stream Azure Active Directory logs to an Azure event hub](../../active-directory/reports-monitoring/tutorial-azure-monitor-stream-logs-to-event-hub.md). |
| [Azure subscription](../data-sources.md#azure-subscription) | Azure activity log | Create a log profile to export activity log events to event hubs. For more information, see [Stream Azure platform logs to Azure event hubs](../essentials/resource-logs.md#send-to-azure-event-hubs). |
| [Azure resources](../data-sources.md#azure-resources) | Platform metrics<br> Resource logs |Both types of data are sent to an event hub by using a resource diagnostic setting. For more information, see [Stream Azure resource logs to an event hub](../essentials/resource-logs.md#send-to-azure-event-hubs). |
| [Operating system (guest)](../data-sources.md#operating-system-guest) | Azure virtual machines | Install the [Azure Diagnostics extension](../agents/diagnostics-extension-overview.md) on Windows and Linux virtual machines in Azure. For more information, see [Streaming Azure Diagnostics data in the hot path by using event hubs](../agents/diagnostics-extension-stream-event-hubs.md) for details on Windows VMs. See [Use Linux Diagnostic extension to monitor metrics and logs](../../virtual-machines/extensions/diagnostics-linux.md#protected-settings) for details on Linux VMs. |
| [Application code](../data-sources.md#application-code) | Application Insights | Use diagnostic settings to stream to event hubs. This tier is only available with workspace-based Application Insights resources. For help with setting up workspace-based Application Insights resources, see [Workspace-based Application Insights resources](../app/create-workspace-resource.md#workspace-based-application-insights-resources) and [Migrate to workspace-based Application Insights resources](../app/convert-classic-resource.md#migrate-to-workspace-based-application-insights-resources).|

## Manual streaming with a logic app
For data that you can't directly stream to an event hub, you can write to Azure Storage Then you can use a time-triggered logic app that [pulls data from Azure Blob Storage](../../connectors/connectors-create-api-azureblobstorage.md#add-action) and [pushes it as a message to the event hub](../../connectors/connectors-create-api-azure-event-hubs.md#add-action).

## Partner tools with Azure Monitor integration

Routing your monitoring data to an event hub with Azure Monitor enables you to easily integrate with external SIEM and monitoring tools. The following table lists examples of tools with Azure Monitor integration.

| Tool | Hosted in Azure | Description |
|:---|:---| :---|
|  IBM QRadar | No | The Microsoft Azure DSM and Microsoft Azure Event Hubs Protocol are available for download from [the IBM support website](https://www.ibm.com/support). |
| Splunk | No | [Splunk Add-on for Microsoft Cloud Services](https://splunkbase.splunk.com/app/3110/) is an open-source project available in Splunkbase. <br><br> If you can't install an add-on in your Splunk instance and, for example, you're using a proxy or running on Splunk Cloud, you can forward these events to the Splunk HTTP Event Collector by using [Azure Function for Splunk](https://github.com/Microsoft/AzureFunctionforSplunkVS). This tool is triggered by new messages in the event hub. |
| SumoLogic | No | Instructions for setting up SumoLogic to consume data from an event hub are available at [Collect Logs for the Azure Audit App from Event Hubs](https://help.sumologic.com/docs/integrations/microsoft-azure/audit/#collecting-logs-for-the-azure-audit-app-from-event-hub). |
| ArcSight | No | The ArcSight Azure Event Hubs smart connector is available as part of [the ArcSight smart connector collection](https://community.microfocus.com/cyberres/arcsight/f/arcsight-product-announcements/163662/announcing-general-availability-of-arcsight-smart-connectors-7-10-0-8114-0). |
| Syslog server | No | If you want to stream Azure Monitor data directly to a Syslog server, you can use a [solution based on an Azure function](https://github.com/miguelangelopereira/azuremonitor2syslog/).
| LogRhythm | No| Instructions to set up LogRhythm to collect logs from an event hub are available at [this LogRhythm website](https://logrhythm.com/six-tips-for-securing-your-azure-cloud-environment/).
|Logz.io | Yes | For more information, see [Get started with monitoring and logging by using Logz.io for Java apps running on Azure](/azure/developer/java/fundamentals/java-get-started-with-logzio).

Other partners might also be available. For a more complete list of all Azure Monitor partners and their capabilities, see [Azure Monitor partner integrations](../partners.md).

## Next steps
* [Archive the activity log to a storage account](./activity-log.md#legacy-collection-methods)
* [Read the overview of the Azure activity log](../essentials/platform-logs-overview.md)
* [Set up an alert based on an activity log event](../alerts/alerts-log-webhook.md)
