---
title: 策略定义结构的详细信息
description: 介绍如何使用策略定义为组织中的 Azure 资源建立约定。
ms.date: 11/26/2019
ms.topic: conceptual
ms.openlocfilehash: c067a5a603c1adcafe6827b3118ecff20ae23238
ms.sourcegitcommit: aee08b05a4e72b192a6e62a8fb581a7b08b9c02a
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/09/2020
ms.locfileid: "75770928"
---
# <a name="azure-policy-definition-structure"></a>Azure Policy 定义结构

Azure Policy 使用资源策略定义来建立资源约定。 每个定义描述资源符合性，以及在资源不符合的情况下会产生什么影响。
通过定义约定，可以控制成本并更轻松地管理资源。 例如，可指定仅允许特定类型的虚拟机。 或者，可要求所有资源都拥有特定标记。 策略由所有子资源继承。 如果将策略应用到资源组，则会将其应用到该资源组中的所有资源。

策略定义架构可在此处找到： [https://schema.management.azure.com/schemas/2019-06-01/policyDefinition.json](https://schema.management.azure.com/schemas/2019-06-01/policyDefinition.json)

使用 JSON 创建策略定义。 策略定义包含以下项的元素：

- mode
- parameters
- 显示名称
- description
- 策略规则
  - 逻辑评估
  - 效果

例如，以下 JSON 说明限制资源部署位置的策略：

```json
{
    "properties": {
        "mode": "all",
        "parameters": {
            "allowedLocations": {
                "type": "array",
                "metadata": {
                    "description": "The list of locations that can be specified when deploying resources",
                    "strongType": "location",
                    "displayName": "Allowed locations"
                },
                "defaultValue": [ "westus2" ]
            }
        },
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "policyRule": {
            "if": {
                "not": {
                    "field": "location",
                    "in": "[parameters('allowedLocations')]"
                }
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}
```

所有 Azure 策略示例均在[Azure 策略示例](../samples/index.md)中。

## <a name="mode"></a>“模式”

**模式**的配置取决于策略是以 Azure 资源管理器属性还是资源提供程序属性为目标。

### <a name="resource-manager-modes"></a>资源管理器模式

**模式**确定将对策略评估哪些资源类型。 支持的模式包括：

- `all`：评估资源组和所有资源类型
- `indexed`：仅评估支持标记和位置的资源类型

大多数情况下，建议将“mode”设置为`all`。 通过门户创建的所有策略定义使用 `all` 模式。 如果使用 PowerShell 或 Azure CLI，则可以手动指定 **mode** 参数。 如果策略定义不包含 **mode** 值，为提供后向兼容性，在 Azure PowerShell 中默认为 `all`，在 Azure CLI 中默认为 `null`。 `null` 模式等同于使用 `indexed` 来支持后向兼容性。

在创建强制执行标记或位置的策略时，应该使用 `indexed`。 虽然并不是必需的，但是它会阻止不支持标记和位置的资源，使其不会在符合性结果中显示为不兼容。 资源组是一个例外。 在资源组上强制执行位置或标记的策略应将“mode”设为 `all`，并专门针对 `Microsoft.Resources/subscriptions/resourceGroups` 类型。 请在[强制执行资源组标记](../samples/enforce-tag-rg.md)查看相关示例。 有关支持标记的资源的列表，请参阅[Azure 资源的标记支持](../../../azure-resource-manager/tag-support.md)。

### <a name="a-nameresource-provider-modes-resource-provider-modes-preview"></a><a name="resource-provider-modes" />资源提供程序模式（预览）

预览期间目前支持以下资源提供程序模式：

- 用于管理[Azure Kubernetes 服务](../../../aks/intro-kubernetes.md)上的许可控制器规则 `Microsoft.ContainerService.Data`。 使用此资源提供程序模式的策略**必须**使用[EnforceRegoPolicy](./effects.md#enforceregopolicy)效果。
- 用于管理 Azure 上的 AKS 引擎 Kubernetes 群集的 `Microsoft.Kubernetes.Data`。
  使用此资源提供程序模式的策略**必须**使用[EnforceOPAConstraint](./effects.md#enforceopaconstraint)效果。
- 用于在[Azure Key Vault](../../../key-vault/key-vault-overview.md)中管理保管库和证书的 `Microsoft.KeyVault.Data`。

> [!NOTE]
> 资源提供程序模式仅支持内置策略定义，并在预览时不支持计划。

## <a name="parameters"></a>参数

参数可减少策略定义的数量，有助于简化策略管理。 使用类似窗体中字段的参数 - `name`、`address`、`city`、`state`。 这些参数始终不变，但其值会基于窗体中的各填写内容变化。
构建策略时，参数同样适用。 如果在策略定义中包括参数，就可以通过使用不同的值重新使用策略以执行不同方案。

> [!NOTE]
> 参数可以添加到现有和已分配的定义。 新参数必须包含 defaultValue 属性。 这可以防止策略或计划的现有分配间接被设为无效。

### <a name="parameter-properties"></a>参数属性

参数有下述可以在策略定义中使用的属性：

- **名称**：参数的名称。 由策略规则中的 `parameters` 部署函数使用。 有关详细信息，请参阅[使用参数值](#using-a-parameter-value)。
- `type`：确定参数是否为**字符串**、**数组**、**对象**、**布尔值**、**整数**、**浮点数**或**日期时间**。
- `metadata`：定义主要由 Azure 门户用于显示用户友好信息的子属性：
  - `description`：有关参数用途的说明。 可以用来提供可接受值的示例。
  - `displayName`：门户中为参数显示的友好名称。
  - `strongType`：（可选）通过门户分配策略定义时使用。 提供上下文感知列表。 有关详细信息，请参阅 [strongType](#strongtype)。
  - `assignPermissions`：（可选）将设置为_true_ ，以便 Azure 门户在策略分配过程中创建角色分配。 如果希望在分配范围之外分配权限，此属性很有用。 策略中的每个角色定义有一个角色分配（或计划中的所有策略中的每个角色定义）。 参数值必须是有效的资源或范围。
- `defaultValue`：（可选）如果未指定任何值，则设置赋值中的参数值。
  在更新已分配的现有策略定义时必须使用此项。
- `allowedValues`：（可选）提供参数数组，该参数在赋值期间接受。

例如，可以定义策略定义来限制资源的部署位置。 **allowedLocations** 可以是该策略定义的一个参数。 每次分配策略定义来限制接受的值时，会使用此参数。 使用 **strongType** 可以在通过门户完成分配时提供增强的体验：

```json
"parameters": {
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": [ "westus2" ],
        "allowedValues": [
            "eastus2",
            "westus2",
            "westus"
        ]
    }
}
```

### <a name="using-a-parameter-value"></a>使用参数值

在策略规则中，用以下 `parameters` 函数语法引用参数：

```json
{
    "field": "location",
    "in": "[parameters('allowedLocations')]"
}
```

此示例引用 **allowedLocations** 参数，该参数已在[参数属性](#parameter-properties)中演示过。

### <a name="strongtype"></a>strongType

在 `metadata` 属性中，可以使用 **strongType** 提供 Azure 门户中的选项多选列表。 **strongType** 的允许值目前包括：

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`
- `omsWorkspace`
- `Microsoft.EventHub/Namespaces/EventHubs`
- `Microsoft.EventHub/Namespaces/EventHubs/AuthorizationRules`
- `Microsoft.EventHub/Namespaces/AuthorizationRules`
- `Microsoft.RecoveryServices/vaults`
- `Microsoft.RecoveryServices/vaults/backupPolicies`

## <a name="definition-location"></a>定义位置

创建计划或策略时，需要指定定义位置。 定义位置必须是管理组或订阅。 此位置决定了计划或策略的分配范围。 资源必须是用于分配的目标定义位置的层次结构中的直系成员或子代。

如果定义位置是：

- **订阅** - 只能将该订阅中的资源分配给策略。
- **管理组** - 只能将子管理组和子订阅中的资源分配给策略。 如果计划将策略定义应用于多个订阅，则位置必须是包含那些订阅的管理组。

## <a name="display-name-and-description"></a>显示名称和说明

请使用“displayName”和“description”来标识策略定义，并提供其使用上下文。 **displayName** 的最大长度为 128个字符，**description** 的最大长度为 512个字符。

## <a name="policy-rule"></a>策略规则

策略规则包括 **If** 和 **Then** 块。 在 **If** 块中，定义强制执行策略时指定的一个或多个条件。 可以对这些条件应用逻辑运算符，以精确定义策略的方案。

在 **Then** 块中，定义满足 **If** 条件时产生的效果。

```json
{
    "if": {
        <condition> | <logical operator>
    },
    "then": {
        "effect": "deny | audit | append | auditIfNotExists | deployIfNotExists | disabled"
    }
}
```

### <a name="logical-operators"></a>逻辑运算符

支持的逻辑运算符为：

- `"not": {condition  or operator}`
- `"allOf": [{condition or operator},{condition or operator}]`
- `"anyOf": [{condition or operator},{condition or operator}]`

**not** 语法反转条件的结果。 **allOf** 语法（与逻辑 **And** 操作相似）要求所有条件为 true。 **anyOf** 语法（与逻辑 **Or** 操作相似）要求一个或多个条件为 true。

可以嵌套逻辑运算符。 以下示例显示了嵌套在 allOf 操作中的 not 操作。

```json
"if": {
    "allOf": [{
            "not": {
                "field": "tags",
                "containsKey": "application"
            }
        },
        {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
        }
    ]
},
```

### <a name="conditions"></a>条件

条件用于评估 **field** 或 **value** 访问器是否符合特定标准。 支持的条件有：

- `"equals": "stringValue"`
- `"notEquals": "stringValue"`
- `"like": "stringValue"`
- `"notLike": "stringValue"`
- `"match": "stringValue"`
- `"matchInsensitively": "stringValue"`
- `"notMatch": "stringValue"`
- `"notMatchInsensitively": "stringValue"`
- `"contains": "stringValue"`
- `"notContains": "stringValue"`
- `"in": ["stringValue1","stringValue2"]`
- `"notIn": ["stringValue1","stringValue2"]`
- `"containsKey": "keyName"`
- `"notContainsKey": "keyName"`
- `"less": "value"`
- `"lessOrEquals": "value"`
- `"greater": "value"`
- `"greaterOrEquals": "value"`
- `"exists": "bool"`

使用 like 和 notLike 条件时，请在值中指定通配符 `*`。
值不应包含多个通配符 `*`。

使用**match**和**notMatch**条件时，请提供 `#` 来匹配数字，`?` 对于字母，`.` 与任何字符匹配，并使用任何其他字符匹配该实际字符。
“match”和“notMatch”区分大小写。 “matchInsensitively”和“notMatchInsensitively”中提供了不区分大小写的替代方案。 例如，请参阅[允许多个名称模式](../samples/allow-multiple-name-patterns.md)。

### <a name="fields"></a>字段

使用字段构成条件。 字段匹配资源请求有效负载中的属性，并说明资源的状态。

支持以下字段：

- `name`
- `fullName`
  - 返回资源全名。 资源全名是最前面为任意父资源名称的资源名称（例如“myServer/myDatabase”）。
- `kind`
- `type`
- `location`
  - 对于不限位置的资源，请使用 **global**。 如需示例，请参阅[示例 - 允许的位置](../samples/allowed-locations.md)。
- `identity.type`
  - 返回在资源上启用的[托管标识](../../../active-directory/managed-identities-azure-resources/overview.md)类型。
- `tags`
- `tags['<tagName>']`
  - 此括号语法支持具有标点符号的标记名称，例如连字符、句点或空格。
  - 其中 **\<tagName\>** 是要验证其条件的标记的名称。
  - 示例：`tags['Acct.CostCenter']`，其中 Acct.CostCenter 是标记的名称。
- `tags['''<tagName>''']`
  - 此括号语法通过双撇号进行转义，可支持在其中包含撇号的标记名称。
  - 其中“\<tagName\>”是要验证其条件的标记的名称。
  - 示例： `tags['''My.Apostrophe.Tag''']`，其中 **"My. tag"** 是标记的名称。
- 属性别名 - 有关列表，请参阅[别名](#aliases)。

> [!NOTE]
> `tags.<tagName>``tags[tagName]` 和 `tags[tag.with.dots]` 仍然是可接受的用于声明标记字段的方式。 但是，首选表达式是上面列出的那些。

#### <a name="use-tags-with-parameters"></a>使用带参数的标记

参数值可以传递给标记字段。 将参数传递给标记字段可在策略分配期间提高策略定义的灵活性。

在以下示例中，`concat` 用于为名为 tagName 参数值的标记创建标记字段查找。 如果该标记不存在，则使用 "`resourcegroup()` 查找" 功能，将 "**修改**" 效果用于使用为审核资源父资源组上的相同命名标记集的值添加标记。

```json
{
    "if": {
        "field": "[concat('tags[', parameters('tagName'), ']')]",
        "exists": "false"
    },
    "then": {
        "effect": "modify",
        "details": {
            "operations": [{
                "operation": "add",
                "field": "[concat('tags[', parameters('tagName'), ']')]",
                "value": "[resourcegroup().tags[parameters('tagName')]]"
            }],
            "roleDefinitionIds": [
                "/providers/microsoft.authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c"
            ]
        }
    }
}
```

### <a name="value"></a>值

也可使用 **value** 来形成条件。 **value** 会针对[参数](#parameters)、[支持的模板函数](#policy-functions)或文本来检查条件。
**value** 可与任何支持的[条件](#conditions)配对。

> [!WARNING]
> 如果_模板函数_的结果为错误，则策略评估将失败。 失败评估是隐式**拒绝**。 有关详细信息，请参阅[避免模板失败](#avoiding-template-failures)。

#### <a name="value-examples"></a>Value 示例

此策略规则示例使用 **value** 将 `resourceGroup()` 函数和返回的 **name** 属性的结果与 **like** 条件 `*netrg` 进行对比。 规则在名称以 `*netrg`结尾的任何资源组中拒绝 `Microsoft.Network/*`**类型**的任何资源。

```json
{
    "if": {
        "allOf": [{
                "value": "[resourceGroup().name]",
                "like": "*netrg"
            },
            {
                "field": "type",
                "notLike": "Microsoft.Network/*"
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

此策略规则示例使用**value**来检查多个嵌套函数的结果是否**等于**`true`。 此规则拒绝并没有至少三个标记的资源。

```json
{
    "mode": "indexed",
    "policyRule": {
        "if": {
            "value": "[less(length(field('tags')), 3)]",
            "equals": true
        },
        "then": {
            "effect": "deny"
        }
    }
}
```

#### <a name="avoiding-template-failures"></a>避免模板失败

使用中的_模板函数_可以**实现许多**复杂的嵌套函数。 如果_模板函数_的结果为错误，则策略评估将失败。 失败评估是隐式**拒绝**。 在某些情况下失败的**值**的示例：

```json
{
    "policyRule": {
        "if": {
            "value": "[substring(field('name'), 0, 3)]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

上面的示例策略规则使用[substring （）](../../../azure-resource-manager/templates/template-functions-string.md#substring)将**name**的前三个字符与**abc**进行比较。 如果**name**小于三个字符，`substring()` 函数将导致错误。 此错误将导致策略变成**拒绝**效果。

相反，请使用[if （）](../../../azure-resource-manager/templates/template-functions-logical.md#if)函数检查**name**等于**abc**的前三个字符是否不允许**名称**少于三个字符来导致错误：

```json
{
    "policyRule": {
        "if": {
            "value": "[if(greaterOrEquals(length(field('name')), 3), substring(field('name'), 0, 3), 'not starting with abc')]",
            "equals": "abc"
        },
        "then": {
            "effect": "audit"
        }
    }
}
```

在修改后的策略规则中，`if()` 将检查**名称**长度，然后再尝试获取长度少于三个字符的值的 `substring()`。 如果**name**太短，则改为返回值 "not 始于 abc"，并将其与**abc**进行比较。 短名称不以**abc**开头的资源仍未通过策略规则，但在评估过程中不再导致错误。

### <a name="count"></a>计数

计算资源有效负载中数组的多少个成员满足条件表达式的条件可以使用**计数**表达式形成。 常见的方案是检查是否至少有一个 "、" 和 "全部为"，或数组成员是否满足条件。 **count**计算一个条件表达式的每个数组成员并为_true_结果求和，然后将结果与表达式运算符进行比较。

**计数**表达式的结构为：

```json
{
    "count": {
        "field": "<[*] alias>",
        "where": {
            /* condition expression */
        }
    },
    "<condition>": "<compare the count of true condition expression array members to this value>"
}
```

以下属性与**count**一起使用：

- **count. field** （必需）：包含数组的路径，必须是数组别名。 如果缺少数组，则表达式的计算结果为_false_ ，而不考虑条件表达式。
- **count。 where** （可选）：用于分别计算**count** [\[\] alias](#understanding-the--alias)数组成员的每个 \*的条件表达式。 如果未提供此属性，则将路径为 "field" 的所有数组成员均计算为_true_。 任何[条件](../concepts/definition-structure.md#conditions)都可以在此属性内使用。
  可以在此属性内使用[逻辑运算符](#logical-operators)来创建复杂的评估要求。
- **\<条件\>** （必需）：将值与满足计数的项的数目进行比较 **。 where**条件表达式。 应使用数值[条件](../concepts/definition-structure.md#conditions)。

#### <a name="count-examples"></a>计数示例

示例1：检查数组是否为空

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]"
    },
    "equals": 0
}
```

示例2：检查是否只有一个数组成员满足条件表达式

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "My unique description"
        }
    },
    "equals": 1
}
```

示例3：检查是否至少有一个数组成员满足条件表达式

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "My common description"
        }
    },
    "greaterOrEquals": 1
}
```

示例4：检查所有对象数组成员是否满足条件表达式

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].description",
            "equals": "description"
        }
    },
    "equals": "[length(field('Microsoft.Network/networkSecurityGroups/securityRules[*]'))]"
}
```

示例5：检查所有字符串数组成员是否满足条件表达式

```json
{
    "count": {
        "field": "Microsoft.Sql/servers/securityAlertPolicies/emailAddresses[*]",
        "where": {
            "field": "Microsoft.Sql/servers/securityAlertPolicies/emailAddresses[*]",
            "like": "*@contoso.com"
        }
    },
    "equals": "[length(field('Microsoft.Sql/servers/securityAlertPolicies/emailAddresses[*]'))]"
}
```

示例 6 **：使用**值内部**值**检查所有数组成员是否满足条件表达式

```json
{
    "count": {
        "field": "Microsoft.Sql/servers/securityAlertPolicies/emailAddresses[*]",
        "where": {
            "value": "[last(split(first(field('Microsoft.Sql/servers/securityAlertPolicies/emailAddresses[*]')), '@'))]",
            "equals": "contoso.com"
        }
    },
    "equals": "[length(field('Microsoft.Sql/servers/securityAlertPolicies/emailAddresses[*]'))]"
}
```

示例7：检查是否至少有一个数组成员与条件表达式中的多个属性相匹配

```json
{
    "count": {
        "field": "Microsoft.Network/networkSecurityGroups/securityRules[*]",
        "where": {
            "allOf": [
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].direction",
                    "equals": "Inbound"
                },
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].access",
                    "equals": "Allow"
                },
                {
                    "field": "Microsoft.Network/networkSecurityGroups/securityRules[*].destinationPortRange",
                    "equals": "3389"
                }
            ]
        }
    },
    "greater": 0
}
```

### <a name="effect"></a>作用

Azure 策略支持以下类型的影响：

- **Append**：会将定义的字段集添加到请求
- **Audit**：会在活动日志中生成一个警告事件，但不会使请求失败
- **AuditIfNotExists**：如果相关资源不存在，则会在活动日志中生成警告事件
- **Deny**：会在活动日志中生成一个事件，并使请求失败
- **DeployIfNotExists**：部署相关资源（如果尚不存在）
- **Disabled**：不评估资源是否符合策略规则
- **EnforceOPAConstraint** （预览版）：为 Azure 上的自托管 Kubernetes 群集配置打开策略代理招生控制器和网关控制器 v3 （预览版）
- **EnforceRegoPolicy** （预览版）：在 Azure Kubernetes 服务中配置打开策略代理招生控制器和网关守卫 v2
- **修改**：添加、更新或删除资源中定义的标记

有关每个效果、评估顺序、属性和示例的完整详细信息，请参阅[了解 Azure 策略影响](effects.md)。

### <a name="policy-functions"></a>策略函数

除了下面的函数和用户定义的函数外，还可以在策略规则中使用所有[资源管理器模板函数](../../../azure-resource-manager/resource-group-template-functions.md)：

- copyIndex()
- deployment()
- list*
- newGuid （）
- pickZones()
- providers()
- reference()
- resourceId()
- variables()

以下函数可在策略规则中使用，但不同于 Azure 资源管理器模板中的使用：

- addDays(dateTime, numberOfDaysToAdd)
  - **dateTime**： [Required] 通用 ISO 8601 dateTime 格式 "Yyyy-mm-dd-yyyy-mm-ddthh： MM： fffffffZ" 中的字符串
  - **numberOfDaysToAdd**： [必需] 整数-要添加的天数
- utcNow （）-与资源管理器模板不同，此值可在 defaultValue 之外使用。
  - 返回一个字符串，该字符串设置为当前日期和时间，采用通用 ISO 8601 DateTime 格式 "yyyy-mm-dd-Yyyy-mm-ddthh： MM： fffffffZ"

此外，`field` 函数可用于策略规则。 `field` 主要用于 **AuditIfNotExists** 和 **DeployIfNotExists**，以引用所评估资源上的字段。 可以在 [DeployIfNotExists 示例](effects.md#deployifnotexists-example)中看到这种用法的示例。

#### <a name="policy-function-example"></a>策略函数示例

此策略规则示例使用 `resourceGroup` 资源函数获取 **name** 属性，并将该属性与 `concat` 数组和对象函数结合使用以构建 `like` 条件，该条件强制资源名称以资源组名称开头。

```json
{
    "if": {
        "not": {
            "field": "name",
            "like": "[concat(resourceGroup().name,'*')]"
        }
    },
    "then": {
        "effect": "deny"
    }
}
```

## <a name="aliases"></a>别名

可以使用属性别名来访问资源类型的特定属性。 通过别名，可限制允许用于资源属性的值和条件。 每个别名会映射到给定资源类型不同 API 版本的路径。 在策略评估期间，策略引擎会获取该 API 版本的属性路径。

别名列表始终不断增长。 若要找出 Azure Policy 当前支持哪些别名，请使用以下方法之一：

- 适用于 Visual Studio Code 的 Azure 策略扩展（推荐）

  使用[适用于 Visual Studio Code 的 Azure 策略扩展](../how-to/extension-for-vscode.md)来查看和发现资源属性的别名。

  ![适用于 Visual Studio Code 的 Azure 策略扩展](../media/extension-for-vscode/extension-hover-shows-property-alias.png)

- Azure Resource Graph

  使用 `project` 运算符可显示资源的**别名**。

  ```kusto
  Resources
  | where type=~'microsoft.storage/storageaccounts'
  | limit 1
  | project aliases
  ```
  
  ```azurecli-interactive
  az graph query -q "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1 | project aliases"
  ```
  
  ```azurepowershell-interactive
  Search-AzGraph -Query "Resources | where type=~'microsoft.storage/storageaccounts' | limit 1 | project aliases"
  ```

- Azure PowerShell

  ```azurepowershell-interactive
  # Login first with Connect-AzAccount if not using Cloud Shell

  # Use Get-AzPolicyAlias to list available providers
  Get-AzPolicyAlias -ListAvailable

  # Use Get-AzPolicyAlias to list aliases for a Namespace (such as Azure Compute -- Microsoft.Compute)
  (Get-AzPolicyAlias -NamespaceMatch 'compute').Aliases
  ```

- Azure CLI

  ```azurecli-interactive
  # Login first with az login if not using Cloud Shell

  # List namespaces
  az provider list --query [*].namespace

  # Get Azure Policy aliases for a specific Namespace (such as Azure Compute -- Microsoft.Compute)
  az provider show --namespace Microsoft.Compute --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
  ```

- REST API/ARMClient

  ```http
  GET https://management.azure.com/providers/?api-version=2017-08-01&$expand=resourceTypes/aliases
  ```

### <a name="understanding-the--alias"></a>了解 [*] 别名

可用的几个别名具有显示为 "normal" 名称的版本，另一个具有 **\[\*** 附加到它的 \]。 例如：

- `Microsoft.Storage/storageAccounts/networkAcls.ipRules`
- `Microsoft.Storage/storageAccounts/networkAcls.ipRules[*]`

"Normal" 别名将该字段表示为单个值。 当整个值集必须完全按定义时，此字段适用于完全匹配比较方案，不再更少。

利用 **\[\*\]** 别名，可以对照数组中每个元素的值和每个元素的特定属性进行比较。 这种方法可以比较 "if of"、"if any of" 和 "if all of" 方案的元素属性。 对于更复杂的方案，请使用 "[计数](#count)条件" 表达式。 使用**ipRules\[\*\]** ，示例将验证每个_操作_是否为_Deny_，但不必担心存在多少个规则或 IP_值_是多少。
此示例规则检查**ipRules\[\*\]** 的所有匹配项，并且仅当找**不到至少**一个匹配项时应用**effectType** ：

```json
"policyRule": {
    "if": {
        "allOf": [
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules",
                "exists": "true"
            },
            {
                "field": "Microsoft.Storage/storageAccounts/networkAcls.ipRules[*].value",
                "notEquals": "10.0.4.1"
            }
        ]
    },
    "then": {
        "effect": "[parameters('effectType')]"
    }
}
```



有关详细信息，请参阅[评估 [\*] 别名](../how-to/author-policies-for-arrays.md#evaluating-the--alias)。

## <a name="initiatives"></a>计划

使用计划可组合多个相关策略定义，以简化分配和管理，因为可将组作为单个项使用。 例如，可以将相关标记策略组合为单个计划。 将应用计划，而非单独分配每个策略。

下面的示例演示如何创建用于处理 `costCenter` 和 `productName` 这两个标记的计划。 它使用两个内置策略来应用默认标记值。

```json
{
    "properties": {
        "displayName": "Billing Tags Policy",
        "policyType": "Custom",
        "description": "Specify cost Center tag and product name tag",
        "parameters": {
            "costCenterValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for Cost Center tag"
                }
            },
            "productNameValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for product Name tag"
                }
            }
        },
        "policyDefinitions": [{
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            }
        ]
    },
    "id": "/subscriptions/<subscription-id>/providers/Microsoft.Authorization/policySetDefinitions/billingTagsPolicy",
    "type": "Microsoft.Authorization/policySetDefinitions",
    "name": "billingTagsPolicy"
}
```

## <a name="next-steps"></a>后续步骤

- 查看[Azure 策略示例](../samples/index.md)中的示例。
- 查看[了解策略效果](effects.md)。
- 了解如何以[编程方式创建策略](../how-to/programmatically-create.md)。
- 了解如何[获取相容性数据](../how-to/get-compliance-data.md)。
- 了解如何[修正不合规的资源](../how-to/remediate-resources.md)。
- 参阅[使用 Azure 管理组来组织资源](../../management-groups/overview.md)，了解什么是管理组。
