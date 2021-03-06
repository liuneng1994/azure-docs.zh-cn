---
title: 使用 Blitzz 将数据从 Cassandra 迁移到 Azure Cosmos DB Cassandra API
description: 了解如何使用 Blitzz 将数据从 Apache Cassandra 数据库迁移到 Azure Cosmos DB Cassandra API。
author: SnehaGunda
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 08/21/2019
ms.author: sngun
ms.reviewer: sngun
ms.openlocfilehash: 746232a85e326f08d44cf2dbe0a4ae718c7cc312
ms.sourcegitcommit: 6d2a147a7e729f05d65ea4735b880c005f62530f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/22/2019
ms.locfileid: "69984363"
---
# <a name="migrate-data-from-cassandra-to-azure-cosmos-db-cassandra-api-account-using-blitzz"></a>使用 Blitzz 将数据从 Cassandra 迁移到 Azure Cosmos DB Cassandra API 帐户

Azure Cosmos DB 中的 Cassandra API 已成为 Apache Cassandra 上运行的企业工作负荷的理想选择, 原因如下: 

* **无管理和监视开销:** 它消除了跨 OS、JVM 和 yaml 文件及其交互管理和监视大量设置的开销。

* **显著节省成本:** 可以通过 Azure Cosmos DB 节省成本, 包括 VM 的成本、带宽和任何适用的许可证。 此外, 无需管理数据中心、服务器、SSD 存储、网络和电力成本。 

* **能够使用现有的代码和工具:** Azure Cosmos DB 提供的线路协议级别与现有 Cassandra SDK 和工具兼容。 此兼容性确保只需经过细微的更改，就可以将现有代码库用于 Azure Cosmos DB Cassandra API。

可以通过多种方式将数据库工作负荷从一个平台迁移到另一个平台。 [Blitzz](https://www.blitzz.io)是一种工具, 可提供一种安全可靠的方法来执行从各种数据库到 Azure Cosmos DB 的零停机时间的迁移。 本文介绍了使用 Blitzz 将数据从 Apache Cassandra 数据库迁移到 Azure Cosmos DB Cassandra API 所需的步骤。

## <a name="benefits-using-blitzz-for-migration"></a>使用 Blitzz 进行迁移的好处

Blitzz 的迁移解决方案遵循迁移复杂的操作工作负荷的分步方法。 下面是 Blitzz 的零停机迁移计划的一些关键方面:

* 它提供从 Apache Cassandra 数据库到 Azure Cosmos DB 的业务逻辑 (表、索引、视图) 的自动迁移。 无需手动创建架构。

* Blitzz 提供高容量和并行数据库复制。 它通过使用一种称为 "变更数据捕获 (CDC)" 的技术, 使源和目标平台在迁移期间保持同步。 通过使用 CDC, Blitzz 连续从源数据库 (Apache Cassandra) 中提取更改流, 并将其应用于目标数据库 (Azure Cosmos DB)。

* 即使系统中出现硬件或软件故障时, 它仍可容错并且保证只传递一次数据。

* 它使用各种安全方法 (如 SSL、加密) 在传输过程中保护数据。

## <a name="steps-to-migrate-data"></a>迁移数据的步骤

本部分介绍设置 Blitzz 并将数据从 Apache Cassandra 数据库迁移到 Azure Cosmos DB 所需的步骤。

1. 从你计划安装 Blitzz replicant 的计算机上, 添加一个安全证书。 Blitzz replicant 需要此证书才能与指定的 Azure Cosmos DB 帐户建立 SSL 连接。 可以通过以下步骤添加证书:

   ```bash
   wget https://cacert.omniroot.com/bc2025.crt
   mv bc2025.crt bc2025.cer
   keytool -keystore $JAVA_HOME/lib/security/cacerts -importcert -alias bc2025ca -file bc2025.cer
   ```

1. 可以通过在[Blitzz 网站](https://www.blitzz.io)上请求演示来获取 Blitzz 安装和二进制文件。 此外, 还可以向团队发送[电子邮件](mailto:success@blitzz.io)。

   ![Blitzz replicant 工具下载](./media/cassandra-migrate-cosmos-db-blitzz/blitzz-replicant-download.png)

   ![Blitzz replicant 文件](./media/cassandra-migrate-cosmos-db-blitzz/replicant-files.png)

1. 从 CLI 终端设置源数据库配置。 使用 **`vi conf/conn/cassandra.yml`** 命令打开配置文件, 并添加 Cassandra 节点的 IP 地址的逗号分隔列表、端口号、用户名、密码和任何其他必需的详细信息。 下面是配置文件中内容的示例:

   ```bash
   type: CASSANDRA
  
   host: 172.17.0.2
   port: 9042

   username: 'cassandra'
   password: 'cassandra'

   max-connections: 30

   ```

   ![打开 Cassandra 连接编辑器](./media/cassandra-migrate-cosmos-db-blitzz/open-connection-editor-cassandra.png)

   ![Cassandra 连接配置](./media/cassandra-migrate-cosmos-db-blitzz/cassandra-connection-configuration.png)

   填写配置详细信息后, 保存并关闭该文件。

1. 您也可以设置源数据库筛选器文件。 筛选器文件指定要迁移的架构或表。 使用 **`vi filter/cassandra_filter.yml`** 命令打开配置文件并输入以下配置详细信息:

   ```bash

   allow:
   -    schema: “io_blitzz”
   Types: [TABLE]
   ```

   填写数据库筛选器详细信息后, 保存并关闭该文件。

1. 接下来, 将设置目标数据库配置。 在定义配置之前, 请先[创建一个 Azure Cosmos DB Cassandra API 帐户](create-cassandra-dotnet.md#create-a-database-account), 然后创建一个密钥空间和一个表来存储迁移的数据。 因为你要从 Apache Cassandra 迁移到 Azure Cosmos DB 中的 Cassandra API, 你可以使用与 Apache Cassandra 相同的分区键。

1. 在迁移数据之前, 请将容器吞吐量提高到应用程序快速迁移所需的容量。 例如, 可以将吞吐量提高到 100000 ru。 在开始迁移之前缩放吞吐量有助于缩短数据迁移的时间。

   ![在整个过程中缩放 Azure Cosmos 容器](./media/cassandra-migrate-cosmos-db-blitzz/scale-throughput.png)

   迁移完成后, 降低吞吐量。 根据存储的数据量和每个操作所需的 ru 数, 你可以估计数据迁移后所需的吞吐量。 若要详细了解如何估算所需的 ru, 请参阅使用 Azure Cosmos DB 容量规划器文章为[容器和数据库预配吞吐量](set-throughput.md)和[估计 RU/秒](estimate-ru-with-capacity-planner.md)。

1. 从 "**连接字符串**" 窗格中获取 Azure Cosmos 帐户的**联系人点、端口、用户名**和**主要密码**。 你将在配置文件中使用这些值。

1. 从 CLI 终端设置目标数据库配置。 使用 **`vi conf/conn/cosmosdb.yml`** 命令打开配置文件, 并添加一个逗号分隔列表, 其中列出了主机 URI、端口号、用户名、密码和其他必需参数。 下面的示例演示配置文件的内容:

   ```bash
   type: COSMOSDB

   host: '<Azure Cosmos account’s Contact point>'
   port: 10350

   username: 'blitzzdemo'
   password: '<Your Azure Cosmos account’s primary password>'

   max-connections: 30
   ```

1. 接下来, 使用 Blitzz 迁移数据。 可以在**完整**或**快照**模式下运行 Blizz replicant:

   * **完全模式**–在此模式下, replicant 将继续在迁移后运行, 并侦听源 Apache Cassandra 系统中的任何更改。 如果检测到任何更改, 则会在目标 Azure Cosmos 帐户上实时复制这些更改。

   * **快照模式**–在此模式下, 你可以执行架构迁移和一次性数据复制。 此选项不支持实时复制。

   通过使用以上两种模式, 可以在不停机的情况下执行迁移。 

1. 若要迁移数据, 请从 Blitzz replicant CLI 终端运行以下命令:

   ```bash
   ./bin/replicant full conf/conn/cassandra.yaml conf/conn/cosmosdb.yaml --filter filter/cassandra_filter.yaml --replace-existing
   ```

   Replicant UI 显示复制进度。 完成架构迁移和快照操作后, 进度将显示 100%。 迁移完成后, 可以验证目标 Azure Cosmos 数据库上的数据。

   ![Cassandra 数据迁移输出](./media/cassandra-migrate-cosmos-db-blitzz/cassandra-data-migration-output.png)


1. 由于你已使用完整模式进行迁移, 因此你可以在源 Apache Cassandra 数据库上执行插入、更新或删除数据等操作。 稍后验证是否已在目标 Azure Cosmos 数据库上实时复制它们。 迁移之后, 请务必降低为 Azure Cosmos 容器配置的吞吐量。

1. 可以停止 replicant 任何点, 然后用 **--resume**开关重新启动它。 复制从已停止的点继续, 而不会影响数据一致性。 以下命令演示如何使用 resume 开关。

   ```bash
   ./bin/replicant full conf/conn/cassandra.yaml conf/conn/cosmosdb.yaml --filter filter/cassandra_filter.yaml --replace-existing --resume
   ```

若要详细了解如何将数据迁移到目标和实时迁移, 请参阅[Blitzz replicant 演示](https://www.youtube.com/watch?v=fsUhF9LUZmM)。

## <a name="next-steps"></a>后续步骤

* [在容器和数据库上预配吞吐量](set-throughput.md) 
* [分区关键最佳实践](partitioning-overview.md#choose-partitionkey)
* [使用 Azure Cosmos DB 容量规划器文章估算 RU/秒](estimate-ru-with-capacity-planner.md)