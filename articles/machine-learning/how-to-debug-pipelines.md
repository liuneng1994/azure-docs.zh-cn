---
title: 调试机器学习管道并对其进行故障排除
titleSuffix: Azure Machine Learning
description: 调试 Azure 机器学习 SDK for Python 中的机器学习管道并对其进行故障排除。 了解开发管道的常见缺陷，以及帮助你在远程执行前后调试脚本的技巧。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
author: likebupt
ms.author: keli19
ms.date: 12/12/2019
ms.openlocfilehash: e0e4ea4689baeb8e85a765a33c3f8124d6bc4362
ms.sourcegitcommit: aee08b05a4e72b192a6e62a8fb581a7b08b9c02a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/09/2020
ms.locfileid: "75764280"
---
# <a name="debug-and-troubleshoot-machine-learning-pipelines"></a>调试机器学习管道并对其进行故障排除
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

本文介绍如何在[AZURE 机器学习 SDK](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)和[Azure 机器学习设计器（预览版）](https://docs.microsoft.com/azure/machine-learning/concept-designer)中对[计算机学习管道](concept-ml-pipelines.md)进行调试和故障排除。


## <a name="debug-and-troubleshoot-in-the-azure-machine-learning-sdk"></a>Azure 机器学习 SDK 中的调试和故障排除
以下部分概述了生成管道时的常见缺陷，以及用于调试在管道中运行的代码的不同策略。 如果在使管道按预期运行时遇到问题，请使用以下提示。 
### <a name="testing-scripts-locally"></a>本地测试脚本

管道中最常见的失败之一是附加的脚本（数据清除脚本、计分脚本等）未按预期运行，或者包含在 Azure 计算机的工作区中难以调试的远程计算上下文中的运行时错误。学习工作室。 

管道本身不能在本地运行，但在本地计算机上运行隔离的脚本可以更快地进行调试，因为无需等待计算和环境生成过程。 执行此操作需要执行某些开发工作：

* 如果数据位于云数据存储中，则需要下载数据并使其可供脚本使用。 使用较小的数据示例，是一种更好的方法，可减少运行时并快速获取有关脚本行为的反馈
* 如果你正在尝试模拟中间管道步骤，你可能需要手动生成特定脚本从上一步开始的对象类型
* 还需要定义自己的环境，并复制远程计算环境中定义的依赖项

在您的本地环境中运行脚本安装程序后，执行以下调试任务会更容易：

* 附加自定义调试配置
* 暂停执行并检查对象状态
* 捕获在运行时之前不会公开的类型或逻辑错误

> [!TIP] 
> 验证脚本是否按预期运行后，下一步是在单步管道中运行该脚本，然后再尝试在包含多个步骤的管道中运行该脚本。

### <a name="debugging-scripts-from-remote-context"></a>从远程上下文调试脚本

在开始生成管道之前，本地测试脚本是调试主要代码片段和复杂逻辑的好方法，但在某些时候，你可能需要在实际管道运行时调试脚本，尤其是在诊断发生的行为时。在管道步骤之间进行交互的过程中。 建议在步骤脚本中使用 `print()` 语句，以便在远程执行过程中可以看到对象状态和预期值，这与调试 JavaScript 代码的方式类似。

日志文件 `70_driver_log.txt` 包含： 

* 脚本执行过程中的所有打印语句
* 脚本的堆栈跟踪 

若要在门户中查找此日志文件和其他日志文件，请首先单击工作区中的管道运行。

![管道运行列表页](./media/how-to-debug-pipelines/pipelinerun-01.png)

导航到管道运行详细信息页。

![管道运行详细信息页](./media/how-to-debug-pipelines/pipelinerun-02.png)

单击特定步骤的模块。 导航到 "**日志**" 选项卡。其他日志包含有关环境映像生成过程和步骤准备脚本的信息。

![管道运行详细信息页日志选项卡](./media/how-to-debug-pipelines/pipelinerun-03.png)

> [!TIP]
> 可在工作区的 "**终结点**" 选项卡中找到*已发布管道*的运行。 可在**试验**或**管道**中找到*非发布管道*的运行。

### <a name="troubleshooting-tips"></a>故障排除提示

下表包含了管道开发期间的一些常见问题，其中包含可能的解决方案。

| 问题 | 可能的解决方案 |
|--|--|
| 无法将数据传递到 `PipelineData` directory | 请确保已在脚本中创建了一个目录，该目录对应于管道期望步骤输出数据的位置。 在大多数情况下，输入参数将定义输出目录，然后显式创建目录。 使用 `os.makedirs(args.output_dir, exist_ok=True)` 创建输出目录。 有关显示此设计模式的评分脚本示例，请参阅[教程](tutorial-pipeline-batch-scoring-classification.md#write-a-scoring-script)。 |
| 依赖项 bug | 如果你在本地开发并测试了脚本，但在管道中的远程计算上运行时发现依赖项问题，请确保你的计算环境依赖项和版本与你的测试环境匹配。 |
| 计算目标存在不明确错误 | 删除并重新创建计算目标可以解决计算目标的某些问题。 |
| 管道未重复使用步骤 | 默认情况下，将启用步骤重用，但请确保未在管道步骤中禁用它。 如果重新使用处于禁用状态，则步骤中的 `allow_reuse` 参数将设置为 `False`。 |
| 不必要地重新运行管道 | 若要确保步骤仅在基础数据或脚本更改时重新运行，请为每个步骤分离你的目录。 如果对多个步骤使用相同的源目录，则可能会遇到不必要的重新运行。 使用管道步骤对象上的 `source_directory` 参数指向该步骤的隔离目录，并确保没有为多个步骤使用相同的 `source_directory` 路径。 |

## <a name="debug-and-troubleshoot-in-azure-machine-learning-designer-preview"></a>在 Azure 机器学习设计器中进行调试和故障排除（预览）

本部分概述了如何在设计器中对管道进行故障排除。
对于在设计器中创建的管道，可以在 "创作" 页或 "管道运行详细信息" 页中找到**日志文件**。

### <a name="access-logs-from-the-authoring-page"></a>从创作页访问日志

提交管道运行并停留在创作页面时，可以找到为每个模块生成的日志文件。

1. 在创作画布中选择任何模块。
1. 在 "属性" 窗格中，切换到 "**日志**" 选项卡。
1. 选择日志文件 `70_driver_log.txt`

    ![创作页模块日志](./media/how-to-debug-pipelines/pipelinerun-05.png)

### <a name="access-logs-from-pipeline-runs"></a>从管道运行访问日志

您还可以在 "管道 **" 或 "** **试验**" 部分的 "管道运行详细信息" 页中找到特定运行的日志文件。

1. 选择在设计器中创建的管道运行。
    ![管道运行页](./media/how-to-debug-pipelines/pipelinerun-04.png)
1. 选择预览窗格中的任何模块。
1. 在 "属性" 窗格中，切换到 "**日志**" 选项卡。
1. 选择日志文件 `70_driver_log.txt`

## <a name="next-steps"></a>后续步骤

* 有关[azureml 管道核心](https://docs.microsoft.com/python/api/azureml-pipeline-core/?view=azure-ml-py)包和[azureml 管道-步骤](https://docs.microsoft.com/python/api/azureml-pipeline-steps/?view=azure-ml-py)包的帮助，请参阅 SDK 参考。

* 请参阅[设计器异常和错误代码](algorithm-module-reference/designer-error-codes.md)的列表。
