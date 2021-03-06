---
title: 有关 Azure 网络观察程序的常见问题解答（FAQ） |Microsoft Docs
description: 本文解答了有关 Azure 网络观察程序服务的常见问题解答。
services: network-watcher
documentationcenter: na
author: damendo
manager: ''
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/10/2019
ms.author: damendo
ms.openlocfilehash: 0eea6700b8b248a87666071ee02572d356110cd0
ms.sourcegitcommit: 8b37091efe8c575467e56ece4d3f805ea2707a64
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/09/2020
ms.locfileid: "75830167"
---
# <a name="frequently-asked-questions-faq-about-azure-network-watcher"></a>有关 Azure 网络观察程序的常见问题解答（FAQ）
[Azure 网络观察](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)程序服务提供了一套工具，用于监视、诊断、查看指标，并为 Azure 虚拟网络中的资源启用或禁用日志。 本文解答了有关该服务的常见问题。

## <a name="general"></a>常规

### <a name="what-is-network-watcher"></a>什么是网络观察程序？
网络观察程序用于监视和修复 IaaS （基础结构即服务）组件的网络运行状况，其中包括虚拟机、虚拟网络、应用程序网关、负载均衡器和 Azure 虚拟网络中的其他资源。 它不是用于监视 PaaS （平台即服务）基础结构或获取 web/移动分析的解决方案。

### <a name="what-tools-does-network-watcher-provide"></a>网络观察程序提供了哪些工具？
网络观察程序提供三个主要功能集
* 监视
  * [拓扑视图](https://docs.microsoft.com/azure/network-watcher/view-network-topology)显示了虚拟网络中的资源以及它们之间的关系。
  * 使用[连接监视器](https://docs.microsoft.com/azure/network-watcher/connection-monitor)可以监视 VM 与其他网络资源之间的连接性和延迟时间。
  * [网络性能监视器](https://docs.microsoft.com/azure/azure-monitor/insights/network-performance-monitor)使你能够跨混合网络体系结构、Expressroute 线路和服务/应用程序终结点监视连接性和延迟时间。  
* 诊断
  * 通过[IP 流验证](https://docs.microsoft.com/azure/network-watcher/network-watcher-ip-flow-verify-overview)，可以在 VM 级别检测流量筛选问题。
  * [下一个跃点](https://docs.microsoft.com/azure/network-watcher/network-watcher-next-hop-overview)有助于验证流量路由并检测路由问题。
  * 通过[连接故障排除](https://docs.microsoft.com/azure/network-watcher/network-watcher-connectivity-portal)，可以在 VM 和其他网络资源之间进行一次性连接和延迟检查。
  * [数据包捕获](https://docs.microsoft.com/azure/network-watcher/network-watcher-packet-capture-overview)使你能够捕获虚拟网络中的 VM 上的所有流量。
  * [Vpn 故障排除](https://docs.microsoft.com/azure/network-watcher/network-watcher-troubleshoot-overview)在 vpn 网关和连接上运行多个诊断检查，以帮助调试问题。
* 日志记录
  * [NSG Flow 日志](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-overview)可用于记录[网络安全组（nsg）](https://docs.microsoft.com/azure/virtual-network/security-overview)中的所有流量
  * [流量分析](https://docs.microsoft.com/azure/network-watcher/traffic-analytics)处理 NSG 流日志数据，使你能够可视化、查询、分析和了解你的网络流量。


有关更多详细信息，请参阅[网络观察程序概述页](https://docs.microsoft.com/azure/network-watcher/network-watcher-monitoring-overview)。


### <a name="how-does-network-watcher-pricing-work"></a>网络观察程序定价如何工作？
有关网络观察程序组件及其定价，请访问[定价页](https://azure.microsoft.com/pricing/details/network-watcher/)。

### <a name="which-regions-is-network-watcher-supportedavailable-in"></a>哪些区域是支持的网络观察程序/哪些区域可用？
可以在[Azure 服务可用性页](https://azure.microsoft.com/global-infrastructure/services/?products=network-watcher)上查看最新的区域可用性

### <a name="what-are-resource-limits-on-network-watcher"></a>什么是网络观察程序的资源限制？
有关所有限制，请参阅[服务限制](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#network-watcher-limits)页。  

### <a name="why-is-only-one-instance-of-network-watcher-allowed-per-region"></a>为什么每个区域只允许一个网络观察程序实例？
仅需为订阅启用一次网络观察程序，才能使用它的功能，这并不是服务限制。

### <a name="why-do-i-need-to-install-the-network-watcher-extension"></a>为什么需要安装网络观察程序扩展？ 
需要生成或拦截来自 VM 的流量的任何功能都需要网络观察程序扩展。 

### <a name="which-features-require-the-network-watcher-extension"></a>哪些功能需要网络观察程序扩展？
只有数据包捕获、连接故障排除和连接监视器需要网络观察程序扩展。

## <a name="nsg-flow-logs"></a>NSG 流日志

### <a name="what-does-nsg-flow-logs-do"></a>NSG 流日志有什么作用？
可以通过[网络安全组（nsg）](https://docs.microsoft.com/azure/virtual-network/security-overview)合并和管理 Azure 网络资源。 NSG Flow 日志使你可以记录有关通过 Nsg 的所有流量的5元组流信息。 将原始流日志写入 Azure 存储帐户，以便可以根据需要进一步处理、分析、查询或导出这些日志。

### <a name="how-do-i-use-nsg-flow-logs-on-a-storage-account-with-a-firewall-or-through-a-service-endpoints"></a>如何实现使用防火墙或服务终结点的存储帐户上的 NSG 流日志？

若要将存储帐户用于防火墙或通过服务终结点，你必须允许受信任的 Microsoft 服务访问你的存储帐户：

* 在 [NSG 流日志概览页](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs)上找到 NSG 即可找到存储帐户的名称
* 在门户的全局搜索框中键入存储帐户的名称，导航到存储帐户
* 在“设置”部分，选择“防火墙和虚拟网络”
* 在 "允许访问" 中，选择 "**所选网络**"。 然后，在 "**例外**" 下，勾选 **"允许受信任的 Microsoft 服务访问此存储帐户"** 旁边的框 
* 如果已选中，则不需进行更改。  

可以在数分钟后检查存储日志，应该会看到时间戳已更新，或者会看到新的 JSON 文件已创建。

### <a name="what-is-the-difference-between-flow-logs-versions-1--2"></a>流日志版本 1 & 2 之间有何区别？
流日志版本2引入了*流状态*& 存储有关传输的字节和数据包的信息。 [了解详细信息](https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-overview#log-file)。

## <a name="next-steps"></a>后续步骤
 - 有关帮助你开始了解网络观察程序的一些教程，请转到我们的[文档概述页](https://docs.microsoft.com/azure/network-watcher/)。
