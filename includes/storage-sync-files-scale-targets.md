---
title: include 文件
description: include 文件
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 05/05/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 6a053b94813145f9ccd69158d18edb728d5dad61
ms.sourcegitcommit: 6bb98654e97d213c549b23ebb161bda4468a1997
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/03/2019
ms.locfileid: "74795740"
---
| 资源 | 确定目标 | 硬限制 |
|----------|--------------|------------|
| 每个区域的存储同步服务数 | 20个存储同步服务 | 是 |
| 每个存储同步服务的同步组数 | 100 个同步组 | 是 |
| 每个存储同步服务的已注册服务器 | 99 台服务器 | 是 |
| 每个同步组的云终结点 | 1 个云终结点 | 是 |
| 每个同步组的服务器终结点 | 50 个服务器终结点 | No |
| 每个服务器的服务器终结点数 | 30 个服务器终结点 | 是 |
| 每个同步组的文件系统对象数（目录和文件） | 100000000对象 | No |
| 目录中的最大文件系统对象（目录和文件）数 | 5000000对象 | 是 |
| 最大对象（目录和文件）安全描述符大小 | 64 KiB | 是 |
| 文件大小 | 100 GiB | No |
| 要进行分层的文件的最小文件大小 | V9.x：基于文件系统群集大小（双文件系统群集大小）。 例如，如果文件系统群集大小为4kb，则文件的最小大小为8kb。<br> V8 及更早版本： 64 KiB  | 是 |

> [!Note]  
> Azure 文件同步终结点可以扩展到 Azure 文件共享的大小。 如果达到 Azure 文件共享大小限制，同步将无法运行。
