---
title: UserNameTextBox UI 元素
description: 介绍了 Azure 门户的 Microsoft.Compute.UserNameTextBox UI 元素。 允许用户提供 Windows 或 Linux 用户名。
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: c7544ae7d872a64547cb6c57ce8af9a09fc6c3d8
ms.sourcegitcommit: f788bc6bc524516f186386376ca6651ce80f334d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/03/2020
ms.locfileid: "75651899"
---
# <a name="microsoftcomputeusernametextbox-ui-element"></a>Microsoft.Compute.UserNameTextBox UI 元素

一个具有针对 Windows 和 Linux 用户名的内置验证的文本框控件。

## <a name="ui-sample"></a>UI 示例

![Microsoft.Compute.UserNameTextBox](./media/managed-application-elements/microsoft.compute.usernametextbox.png)

## <a name="schema"></a>架构

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.UserNameTextBox",
  "label": "User name",
  "defaultValue": "",
  "toolTip": "",
  "constraints": {
    "required": true,
    "regex": "^[a-z0-9A-Z]{1,30}$",
    "validationMessage": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
  },
  "osPlatform": "Windows",
  "visible": true
}
```

## <a name="sample-output"></a>示例输出

```json
"Example name"
```

## <a name="remarks"></a>备注

- 如果 `constraints.required` 设置为 true，则文本框必须包含值才能成功通过验证。 默认值为 **true**。
- 必须指定 `osPlatform`，它可以是 **Windows** 或 **Linux**。
- `constraints.regex` 是一个 JavaScript 正则表达式模式。 如果指定，则文本框的值必须与模式完全匹配才能成功通过验证。 默认值为 **null**。
- `constraints.validationMessage` 是当文本框的值未通过 `constraints.regex` 指定的验证时会显示的一个字符串。 如果未指定，则会使用文本框的内置验证消息。 默认值为 **null**。
- 此元素具有基于为 `osPlatform` 指定的值的内置验证。 内置验证可以与自定义正则表达式一起使用。 如果指定了 `constraints.regex` 的值，则会同时触发内置验证和自定义验证。

## <a name="next-steps"></a>后续步骤

* 有关创建 UI 定义的简介，请参阅 [CreateUiDefinition 入门](create-uidefinition-overview.md)。
* 有关 UI 元素中的公用属性的说明，请参阅 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
