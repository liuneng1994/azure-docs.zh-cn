---
title: 添加列：模块引用
titleSuffix: Azure Machine Learning
description: 了解如何使用 Azure 机器学习中的 "添加列" 模块连接两个数据集。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 10/22/2019
ms.openlocfilehash: 55981279cb1902424d1a0f77af097dc379d7222f
ms.sourcegitcommit: c22327552d62f88aeaa321189f9b9a631525027c
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2019
ms.locfileid: "73493965"
---
# <a name="add-columns-module"></a>添加列模块

本文介绍 Azure 机器学习设计器（预览版）中的模块。

使用此模块连接两个数据集。 你将指定为输入的两个数据集中的所有列合并为一个数据集。 如果需要连接两个以上的数据集，请使用多个**添加列**的实例。



## <a name="how-to-configure-add-columns"></a>如何配置添加列
1. 将 "**添加列**" 模块添加到管道。

2. 连接要连接的两个数据集。 如果要合并两个以上的数据集，可以将多个 "**添加列**" 的组合组合在一起。

    - 可以合并具有不同行数的两个列。 对于较小的源列中的每一行，输出数据集的缺失值均为。

    - 不能选择要添加的单个列。 使用 "**添加列**" 时，将连接每个数据集中的所有列。 因此，如果您只想要添加列的一个子集，请使用 "选择数据集中的列" 创建包含所需列的数据集。

3. 运行管道。

### <a name="results"></a>结果
管道运行后：

- 若要查看新数据集的前行，请右键单击 "**添加列**" 的输出，然后选择 "可视化"。

新数据集中的列数等于两个输入数据集的列数之和。

如果输入数据集中存在两个具有相同名称的列，则将向该列的名称添加一个数字后缀。 例如，如果有两个名为 TargetOutcome 的列的实例，则左列将被重命名为 TargetOutcome_1，正确的列将被重命名为 TargetOutcome_2。

## <a name="next-steps"></a>后续步骤

查看可用于 Azure 机器学习[的模块集](module-reference.md)。 