---
title: 配置 Prometheus 集成的容器 Azure Monitor |Microsoft Docs
description: 本文介绍如何配置容器代理的 Azure Monitor，以擦除 Prometheus 与 Azure Kubernetes Service 群集的指标。
ms.topic: conceptual
ms.date: 10/15/2019
ms.openlocfilehash: f1da2142f287bde83be7cede282bd854ce822d23
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/25/2019
ms.locfileid: "75403521"
---
# <a name="configure-scraping-of-prometheus-metrics-with-azure-monitor-for-containers"></a>将抓取的 Prometheus 度量值配置为用于容器的 Azure Monitor

[Prometheus](https://prometheus.io/)是一种常用的开源指标监视解决方案，属于[云本机计算基础](https://www.cncf.io/)。 容器 Azure Monitor 提供无缝载入体验来收集 Prometheus 指标。 通常，若要使用 Prometheus，需要使用存储设置和管理 Prometheus 服务器。 与 Azure Monitor 集成后，不需要 Prometheus 服务器。 只需通过导出程序或 pod （应用程序）公开 Prometheus 指标终结点，容器 Azure Monitor 容器的容器即可为你擦除指标。 

![Prometheus 的容器监视体系结构](./media/container-insights-prometheus-integration/monitoring-kubernetes-architecture.png)

>[!NOTE]
>抓取 Prometheus 指标支持的最低代理版本为 ciprod07092019 或更高版本，并且 `KubeMonAgentEvents` 表中用于写入配置和代理错误的代理版本为 ciprod10112019。 有关代理版本和每个版本中包含的内容的其他信息，请参阅[代理发行说明](https://github.com/microsoft/Docker-Provider/tree/ci_feature_prod)。 若要验证您的代理版本，请从 "**节点**" 选项卡中选择一个节点，然后在 "**代理图像标记**" 属性的 "属性" 窗格中说明值。

### <a name="prometheus-scraping-settings"></a>Prometheus 抓取设置

从 Prometheus 中的抓取度量值是从以下两个角度之一执行的：

* 群集范围内的 HTTP URL，并从所列出的服务终结点中发现目标。 例如，k8s services （如 kube）和 kube，以及特定于应用程序的 pod 注释。 将在 "ConfigMap" 部分 *[Prometheus data_collection_settings]* 中定义在此上下文中收集的指标。
* 整个节点-HTTP URL，并从所列出的服务终结点中发现目标。 将在 ConfigMap 部分 *[Prometheus_data_collection_settings]* 中定义在此上下文中收集的指标。

| 终结点 | 范围 | 示例 |
|----------|-------|---------|
| Pod 批注 | 群集范围 | 批注 <br>`prometheus.io/scrape: "true"` <br>`prometheus.io/path: "/mymetrics"` <br>`prometheus.io/port: "8000"` <br>`prometheus.io/scheme: "http"` |
| Kubernetes 服务 | 群集范围 | `http://my-service-dns.my-namespace:9100/metrics` <br>`https://metrics-server.kube-system.svc.cluster.local/metrics` |
| url/端点 | 每节点和/或群集范围 | `http://myurl:9101/metrics` |

如果指定了 URL，则容器 Azure Monitor 仅擦除终结点。 指定 Kubernetes 服务时，服务名称将与群集 DNS 服务器一起解析，以获取 IP 地址，然后将解析的服务擦除。

|范围 | 密钥 | 数据类型 | 值 | Description |
|------|-----|-----------|-------|-------------|
| 群集范围 | | | | 指定以下三种方法中的任意一种，以擦除指标的终结点。 |
| | `urls` | String | 逗号分隔的数组 | HTTP 终结点（指定了 IP 地址或有效的 URL 路径）。 例如：`urls=[$NODE_IP/metrics]`。 （$NODE _IP 是容器参数的特定 Azure Monitor，可以使用而不是节点 IP 地址。 必须全部大写。） |
| | `kubernetes_services` | String | 逗号分隔的数组 | Kubernetes 服务的数组，用于擦除 kube 指标的指标。 例如：`kubernetes_services = ["https://metrics-server.kube-system.svc.cluster.local/metrics", http://my-service-dns.my-namespace:9100/metrics]`。|
| | `monitor_kubernetes_pods` | Boolean | True 或 False | 如果设置为 "群集范围内的设置 `true`"，则容器代理的 Azure Monitor 将在整个群集中擦除 Kubernetes pod，以获得以下 Prometheus 批注：<br> `prometheus.io/scrape:`<br> `prometheus.io/scheme:`<br> `prometheus.io/path:`<br> `prometheus.io/port:` |
| | `prometheus.io/scrape` | Boolean | True 或 False | 启用抓取的。 `monitor_kubernetes_pods` 必须设置为 `true`。 |
| | `prometheus.io/scheme` | String | http 或 https | 默认为废弃 over HTTP。 如有必要，将设置为 `https`。 | 
| | `prometheus.io/path` | String | 逗号分隔的数组 | 要从中提取指标的 HTTP 资源路径。 如果未 `/metrics`度量值路径，请在此批注中定义它。 |
| | `prometheus.io/port` | String | 9102 | 指定要从其擦除的端口。 如果未设置端口，则它将默认为9102。 |
| | `monitor_kubernetes_pods_namespaces` | String | 逗号分隔的数组 | 允许从 Kubernetes pod 擦除指标的命名空间列表。<br> 例如： `monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]` |
| 节点范围 | `urls` | String | 逗号分隔的数组 | HTTP 终结点（指定了 IP 地址或有效的 URL 路径）。 例如：`urls=[$NODE_IP/metrics]`。 （$NODE _IP 是容器参数的特定 Azure Monitor，可以使用而不是节点 IP 地址。 必须全部大写。） |
| 节点范围或群集范围内的 | `interval` | String | 60s | 收集间隔默认值为一分钟（60秒）。 可以将 *[prometheus_data_collection_settings]* 和/或 *[prometheus_data_collection_settings]* 的集合修改为时间单位，如 s、m、h。 |
| 节点范围或群集范围内的 | `fieldpass`<br> `fielddrop`| String | 逗号分隔的数组 | 可以通过设置 "允许（`fieldpass`）" 和 "禁止（`fielddrop`）" 列表来指定要收集或不从终结点收集的某些指标。 必须先设置允许列表。 |

ConfigMaps 是一个全局列表，只能有一个 ConfigMap 应用于代理。 不能将其他 ConfigMaps overruling 到集合中。

## <a name="configure-and-deploy-configmaps"></a>配置和部署 ConfigMaps

执行以下步骤，配置 ConfigMap 配置文件并将其部署到群集。

1. [下载](https://github.com/microsoft/OMS-docker/blob/ci_feature_prod/Kubernetes/container-azm-ms-agentconfig.yaml)模板 ConfigMap yaml 文件并将其另存为容器 azm-agentconfig. yaml。

2. 编辑 ConfigMap yaml 文件，并将自定义设置为擦除 Prometheus 指标。

    - 若要在群集范围内收集 Kubernetes 服务，请使用以下示例配置 ConfigMap 文件。

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        kubernetes_services = ["http://my-service-dns.my-namespace:9102/metrics"]
        ```

    - 若要从群集中的特定 URL 配置抓取 Prometheus 指标，请使用以下示例配置 ConfigMap 文件。

        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings
        [prometheus_data_collection_settings.cluster] 
        interval = "1m"  ## Valid time units are s, m, h.
        fieldpass = ["metric_to_pass1", "metric_to_pass12"] ## specify metrics to pass through 
        fielddrop = ["metric_to_drop"] ## specify metrics to drop from collecting
        urls = ["http://myurl:9101/metrics"] ## An array of urls to scrape metrics from
        ```

    - 若要为群集中的每个节点从代理的 DaemonSet 配置抓取 Prometheus 指标，请在 ConfigMap 中配置以下内容：
    
        ```
        prometheus-data-collection-settings: |- 
        # Custom Prometheus metrics data collection settings 
        [prometheus_data_collection_settings.node] 
        interval = "1m"  ## Valid time units are s, m, h. 
        urls = ["http://$NODE_IP:9103/metrics"] 
        fieldpass = ["metric_to_pass1", "metric_to_pass2"] 
        fielddrop = ["metric_to_drop"] 
        ```

        >[!NOTE]
        >$NODE _IP 是容器参数的特定 Azure Monitor，可以使用而不是节点 IP 地址。 它必须全部大写。 

    - 若要通过指定 pod 批注配置 Prometheus 指标的抓取，请执行以下步骤：

       1. 在 ConfigMap 中，指定以下内容：

            ```
            prometheus-data-collection-settings: |- 
            # Custom Prometheus metrics data collection settings
            [prometheus_data_collection_settings.cluster] 
            interval = "1m"  ## Valid time units are s, m, h
            monitor_kubernetes_pods = true 
            ```

       2. 为 pod 批注指定以下配置：

           ```
           - prometheus.io/scrape:"true" #Enable scraping for this pod 
           - prometheus.io/scheme:"http:" #If the metrics endpoint is secured then you will need to set this to `https`, if not default ‘http’
           - prometheus.io/path:"/mymetrics" #If the metrics path is not /metrics, define it with this annotation. 
           - prometheus.io/port:"8000" #If port is not 9102 use this annotation
           ```
    
          如果要将监视限制为具有批注的 pod 的特定命名空间，例如仅包含专用于生产工作负荷的 pod，请将 `monitor_kubernetes_pod` 设置为在 ConfigMap 中 `true`，并添加命名空间筛选器 `monitor_kubernetes_pods_namespaces` 指定要从其擦除的命名空间。 例如： `monitor_kubernetes_pods_namespaces = ["default1", "default2", "default3"]`

3. 通过运行以下 kubectl 命令创建 ConfigMap： `kubectl apply -f <configmap_yaml_file.yaml>`。
    
    示例：`kubectl apply -f container-azm-ms-agentconfig.yaml`。 
    
    在生效之前，配置更改可能需要几分钟才能完成，并且群集中的所有 omsagent 箱都将重新启动。 重新启动是所有 omsagent pod 的滚动重启，而不是同时重新启动。 重新启动完成后，会显示一条类似于以下内容的消息，其中包括 result： `configmap "container-azm-ms-agentconfig" created`。

## <a name="applying-updated-configmap"></a>应用更新的 ConfigMap

如果已将 ConfigMap 部署到群集，并且想要使用较新的配置对其进行更新，则可以编辑以前使用的 ConfigMap 文件，然后使用与之前相同的命令进行应用，`kubectl apply -f <configmap_yaml_file.yaml`。

在生效之前，配置更改可能需要几分钟才能完成，并且群集中的所有 omsagent 箱都将重新启动。 重新启动是所有 omsagent pod 的滚动重启，而不是同时重新启动。 重新启动完成后，会显示一条类似于以下内容的消息，其中包括 result： `configmap "container-azm-ms-agentconfig" updated`。

## <a name="verify-configuration"></a>验证配置 

若要验证是否已成功应用配置，请使用以下命令从代理 pod 查看日志： `kubectl logs omsagent-fdf58 -n=kube-system`。 如果 omsagent pod 出现配置错误，输出将显示类似于以下内容的错误：

``` 
***************Start Config Processing******************** 
config::unsupported/missing config schema version - 'v21' , using defaults
```

还可以查看与应用配置更改相关的错误。 以下选项可用于对配置更改和抓取 Prometheus 指标执行额外的故障排除：

- 使用同一个 `kubectl logs` 命令从代理 pod 日志。 

- 从实时日志。 实时日志显示类似于以下内容的错误：

    ```
    2019-07-08T18:55:00Z E! [inputs.prometheus]: Error in plugin: error making HTTP request to http://invalidurl:1010/metrics: Get http://invalidurl:1010/metrics: dial tcp: lookup invalidurl on 10.0.0.10:53: no such host
    ```

- Log Analytics 工作区中的**KubeMonAgentEvents**表。 数据每小时发送一次，*并在出现配置错误的情况*中擦除错误和*错误*严重性。 如果没有错误，则表中的条目将包含带有严重性*信息*的数据，而不报告任何错误。 **Tags**属性包含有关在其上发生错误的 pod 和容器 ID 的详细信息，以及最后一个小时内的第一个匹配项和最后一个匹配项。

错误会阻止 omsagent 分析文件，从而导致重新启动并使用默认配置。 更正 ConfigMap 中的错误后，请通过运行以下命令，保存 yaml 文件并应用更新的 ConfigMaps： `kubectl apply -f <configmap_yaml_file.yaml`。

## <a name="query-prometheus-metrics-data"></a>查询 Prometheus 指标数据

若要查看 prometheus 指标擦除 by Azure Monitor 和代理报告的任何配置/抓取错误，请查看[Query prometheus 度量值数据](container-insights-log-search.md#query-prometheus-metrics-data)和[查询配置或抓取错误](container-insights-log-search.md#query-config-or-scraping-errors)。

## <a name="view-prometheus-metrics-in-grafana"></a>在 Grafana 中查看 Prometheus 指标

容器 Azure Monitor 支持在 Grafana 仪表板中查看 Log Analytics 工作区中存储的度量值。 我们提供了一个模板，你可以从 Grafana 的 "[仪表板" 存储库](https://grafana.com/grafana/dashboards?dataSource=grafana-azure-monitor-datasource&category=docker)下载该模板，以帮助你了解如何从受监视的群集查询其他数据以在自定义 Grafana 仪表板中进行可视化。 

## <a name="review-prometheus-data-usage"></a>查看 Prometheus 数据使用情况

若要确定每个指标大小的引入量（以 GB 为单位），以了解它是否很高，请提供以下查询。

```
InsightsMetrics 
| where Namespace == "prometheus"
| where TimeGenerated > ago(24h)
| summarize VolumeInGB = (sum(_BilledSize) / (1024 * 1024 * 1024)) by Name
| order by VolumeInGB desc
| render barchart
```
输出将显示类似于以下内容的结果：

![数据引入卷的日志查询结果](./media/container-insights-prometheus-integration/log-query-example-usage-03.png)

若要估计每个指标的大小（GB）是一个月来了解工作区中收到的数据引入量是否很高，请提供以下查询。

```
InsightsMetrics 
| where Namespace contains "prometheus"
| where TimeGenerated > ago(24h)
| summarize EstimatedGBPer30dayMonth = (sum(_BilledSize) / (1024 * 1024 * 1024)) * 30 by Name
| order by EstimatedGBPer30dayMonth desc
| render barchart
```

输出将显示类似于以下内容的结果：

![数据引入卷的日志查询结果](./media/container-insights-prometheus-integration/log-query-example-usage-02.png)

使用[Azure Monitor 日志管理使用情况和成本](../platform/manage-cost-storage.md)中提供了有关如何监视数据使用情况和分析成本的详细信息。

## <a name="next-steps"></a>后续步骤

若要详细了解如何在[此处](container-insights-agent-config.md)为容器工作负载的 stdout、stderr 和环境变量配置代理收集设置。 
