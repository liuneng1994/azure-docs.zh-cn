---
title: 访问和查看审核日志
titleSuffix: Azure AD B2C
description: 如何在 Azure 门户中以编程方式访问 Azure AD B2C 审核日志。
services: active-directory-b2c
author: mmacy
manager: celestedg
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.date: 10/16/2019
ms.author: marsma
ms.subservice: B2C
ms.custom: fasttrack-edit
ms.openlocfilehash: feefe7cf6d559360defd7c7f830a9e3f2e583cd6
ms.sourcegitcommit: 5b9287976617f51d7ff9f8693c30f468b47c2141
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/09/2019
ms.locfileid: "74948226"
---
# <a name="accessing-azure-ad-b2c-audit-logs"></a>访问 Azure AD B2C 审核日志

Azure Active Directory B2C （Azure AD B2C）会发出审核日志，其中包含有关 B2C 资源、颁发的令牌以及管理员访问的活动信息。 本文简要概述了审核日志中提供的信息，以及如何访问 Azure AD B2C 租户的此数据的说明。

审核日志事件只保留**7 天**。 如果需要保留更长时间，请使用下面所示的方法计划下载并存储日志。

> [!NOTE]
> 在 Azure 门户的 " **Azure Active Directory** " 或 " **Azure AD B2C** " 页下的 "**用户**" 部分下，不能看到用户登录的单个 Azure AD B2C 应用程序。 登录事件显示用户活动，但不能将其与用户登录到的 B2C 应用程序相关联。 如本文中所述，你必须使用审核日志。

## <a name="overview-of-activities-available-in-the-b2c-category-of-audit-logs"></a>审核日志的 B2C 类别中提供的活动的概述

审核日志中的“B2C”类别包含以下类型的活动：

|活动类型 |描述  |
|---------|---------|
|授权 |与访问 B2C 资源的用户授权有关的活动（例如，管理员访问 B2C 策略列表）。         |
|目录 |当管理员使用 Azure 门户登录时，与检索到的目录属性相关的活动。 |
|Application | 在 B2C 应用程序上创建、读取、更新和删除（CRUD）操作。 |
|密钥 |对 B2C 密钥容器中存储的密钥进行 CRUD 操作。 |
|资源 |B2C 资源上的 CRUD 操作。 例如，策略和标识提供者。
|Authentication |验证用户凭据和令牌颁发。|

有关用户对象 CRUD 活动，请参阅“核心目录”类别。

## <a name="example-activity"></a>示例活动

Azure 门户中的此示例图像显示用户使用外部标识提供者（在本例中为 Facebook）登录时捕获的数据：

![Azure 门户中的审核日志活动详细信息页的示例](./media/active-directory-b2c-reference-audit-logs/audit-logs-example.png)

"活动详细信息" 面板包含下列相关信息：

|部分|字段|描述|
|-------|-----|-----------|
| 活动 | 名称 | 发生的活动。 例如，*向应用程序发出 id_token*，这将结束实际用户登录。 |
| 发起者（参与者） | ObjectId | 用户登录到的 B2C 应用程序的**对象 ID** 。 此标识符在 Azure 门户中不可见，但可通过 Microsoft Graph API 访问。 |
| 发起者（参与者） | Spn | 用户登录到的 B2C 应用程序的**应用程序 ID** 。 |
| 目标 | ObjectId | 正在登录的用户的**对象 ID** 。 |
| 更多详细信息 | TenantId | Azure AD B2C 租户的**租户 ID** 。 |
| 更多详细信息 | PolicyId | 用于签署用户的用户流（策略）的**策略 ID** 。 |
| 更多详细信息 | ApplicationId | 用户登录到的 B2C 应用程序的**应用程序 ID** 。 |

## <a name="view-audit-logs-in-the-azure-portal"></a>查看 Azure 门户中的审核日志

Azure 门户提供对 Azure AD B2C 租户中的审核日志事件的访问。

1. 登录到 [Azure 门户](https://portal.azure.com)
1. 切换到包含 Azure AD B2C 租户的目录，然后浏览到**Azure AD B2C**。
1. 在左侧菜单中的 "**活动**" 下，选择 "**审核日志**"。

将显示过去七天内记录的活动事件的列表。

![示例筛选器，其中包含两个活动事件 Azure 门户](media/active-directory-b2c-reference-audit-logs/audit-logs-example-filter.png)

提供了几个筛选选项，其中包括：

* **活动资源类型**-按活动 "[可用的概述](#overview-of-activities-available-in-the-b2c-category-of-audit-logs)" 部分中的表中显示的活动类型进行筛选。
* **日期**-筛选显示的活动的日期范围。

如果在列表中选择某一行，则将显示该事件的活动详细信息。

若要在逗号分隔值（CSV）文件中下载活动事件列表，请选择 "**下载**"。

## <a name="get-audit-logs-with-the-azure-ad-reporting-api"></a>获取具有 Azure AD 报告 API 的审核日志

审核日志将发布到与 Azure Active Directory 其他活动相同的管道，因此可通过 [Azure Active Directory 报告 API](https://docs.microsoft.com/graph/api/directoryaudit-list)进行访问。 有关详细信息，请参阅[Azure Active Directory 报告 API 入门](../active-directory/reports-monitoring/concept-reporting-api.md)。

### <a name="enable-reporting-api-access"></a>启用报表 API 访问

若要允许对 Azure AD 报告 API 进行基于脚本或应用程序的访问，需要使用以下 API 权限在 Azure AD B2C 租户中注册 Azure Active Directory 应用程序：

* Microsoft Graph > 应用程序权限 > 审核日志

你可以在 B2C 租户内的现有 Azure Active Directory 应用程序注册中启用这些权限，或者创建专用于审核日志自动化的新权限。

按照以下步骤注册应用程序，向其授予所需的 Microsoft Graph API 权限，然后创建客户端机密。

### <a name="register-application-in-azure-active-directory"></a>在 Azure Active Directory 中注册应用程序

[!INCLUDE [active-directory-b2c-appreg-mgmt](../../includes/active-directory-b2c-appreg-mgmt.md)]

### <a name="assign-api-access-permissions"></a>分配 API 访问权限

#### <a name="applicationstabapplications"></a>[应用程序](#tab/applications/)

1. 在 "**已注册的应用**概述" 页上，选择 "**设置**"。
1. 在 " **API 访问**" 下，选择 "**所需权限**"。
1. 选择 "**添加**"，然后**选择一个 API**。
1. 选择 " **Microsoft Graph**"，然后**选择**""。
1. 在 "**应用程序权限**" 下，选择 "**读取所有审核日志数据**"。
1. 选择 "**选择**" 按钮，然后选择 "**完成**"。
1. 选择“授予权限”，然后选择“是”。

#### <a name="app-registrations-previewtabapp-reg-preview"></a>[应用注册（预览版）](#tab/app-reg-preview/)

1. 在“管理”下选择“API 权限”。
1. 在“已配置权限”下，选择“添加权限”。
1. 选择 " **Microsoft api** " 选项卡。
1. 选择“Microsoft Graph”。
1. 选择“应用程序权限”。
1. 展开 "**审核日志**"，然后选中 "**审核日志**" 复选框。
1. 选择“添加权限”。 按照指示等待几分钟，然后继续下一步。
1. 选择“向(租户名称)授予管理员许可”。
1. 如果已为你分配了*全局管理员*角色，请选择你当前登录的帐户，或者使用你的 Azure AD B2C 租户中被分配了*全局管理员*角色的帐户登录。
1. 选择“接受”。
1. 选择 "**刷新**"，然后验证 "授权给 ..."显示在*审核日志*权限的 "**状态**" 下。 传播权限可能需要几分钟时间。

* * *

### <a name="create-client-secret"></a>创建客户端密码

[!INCLUDE [active-directory-b2c-client-secret](../../includes/active-directory-b2c-client-secret.md)]

你现在已有一个应用程序，其中包含所需的 API 访问权限、一个应用程序 ID 和一个可在自动化脚本中使用的密钥。 有关如何使用脚本获取活动事件的示例，请参阅本文后面的 "PowerShell 脚本" 一节。

### <a name="access-the-api"></a>访问 API

若要通过 API 下载 Azure AD B2C 审核日志事件，请筛选 `B2C` 类别的日志。 若要按类别进行筛选，请在调用 Azure AD 报告 API 终结点时使用 `filter` 查询字符串参数。

```HTTP
https://graph.microsoft.com/v1.0/auditLogs/directoryAudits?$filter=loggedByService eq 'B2C' and activityDateTime gt 2019-09-10T02:28:17Z
```

### <a name="powershell-script"></a>PowerShell 脚本

下面的 PowerShell 脚本演示了如何查询 Azure AD 报告 API 的示例。 查询 API 后，它会将记录的事件打印到标准输出，然后将 JSON 输出写入文件。

可以在[Azure Cloud Shell](../cloud-shell/overview.md)中尝试此脚本。 请确保将其更新为你的应用程序 ID、客户端密钥和 Azure AD B2C 租户的名称。

```powershell
# This script requires the registration of a Web Application in Azure Active Directory:
# https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-reporting-api

# Constants
$ClientID       = "your-client-application-id-here"       # Insert your application's client ID, a GUID (registered by Global Admin)
$ClientSecret   = "your-client-application-secret-here"   # Insert your application's client secret
$tenantdomain   = "your-b2c-tenant.onmicrosoft.com"       # Insert your Azure AD B2C tenant; for example, contoso.onmicrosoft.com
$loginURL       = "https://login.microsoftonline.com"
$resource       = "https://graph.microsoft.com"           # Microsoft Graph API resource URI
$7daysago       = "{0:s}" -f (get-date).AddDays(-7) + "Z" # Use 'AddMinutes(-5)' to decrement minutes, for example
Write-Output "Searching for events starting $7daysago"

# Create HTTP header, get an OAuth2 access token based on client id, secret and tenant domain
$body       = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
$oauth      = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=1.0 -Body $body

# Parse audit report items, save output to file(s): auditX.json, where X = 0 thru n for number of nextLink pages
if ($oauth.access_token -ne $null) {
    $i=0
    $headerParams = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}
    $url = "https://graph.microsoft.com/v1.0/auditLogs/directoryAudits?`$filter=loggedByService eq 'B2C' and activityDateTime gt  " + $7daysago

    # loop through each query page (1 through n)
    Do {
        # display each event on the console window
        Write-Output "Fetching data using Uri: $url"
        $myReport = (Invoke-WebRequest -UseBasicParsing -Headers $headerParams -Uri $url)
        foreach ($event in ($myReport.Content | ConvertFrom-Json).value) {
            Write-Output ($event | ConvertTo-Json)
        }

        # save the query page to an output file
        Write-Output "Save the output to a file audit$i.json"
        $myReport.Content | Out-File -FilePath audit$i.json -Force
        $url = ($myReport.Content | ConvertFrom-Json).'@odata.nextLink'
        $i = $i+1
    } while($url -ne $null)
} else {
    Write-Host "ERROR: No Access Token"
}
```

下面是本文前面部分中所示的示例活动事件的 JSON 表示形式：

```JSON
{
    "id": "B2C_DQO3J_4984536",
    "category": "Authentication",
    "correlationId": "00000000-0000-0000-0000-000000000000",
    "result": "success",
    "resultReason": "N/A",
    "activityDisplayName": "Issue an id_token to the application",
    "activityDateTime": "2019-09-14T18:13:17.0618117Z",
    "loggedByService": "B2C",
    "operationType": "",
    "initiatedBy": {
        "user": null,
        "app": {
            "appId": "00000000-0000-0000-0000-000000000000",
            "displayName": null,
            "servicePrincipalId": null,
            "servicePrincipalName": "00000000-0000-0000-0000-000000000000"
        }
    },
    "targetResources": [
        {
            "id": "00000000-0000-0000-0000-000000000000",
            "displayName": null,
            "type": "User",
            "userPrincipalName": null,
            "groupType": null,
            "modifiedProperties": []
        }
    ],
    "additionalDetails": [
        {
            "key": "TenantId",
            "value": "test.onmicrosoft.com"
        },
        {
            "key": "PolicyId",
            "value": "B2C_1A_signup_signin"
        },
        {
            "key": "ApplicationId",
            "value": "00000000-0000-0000-0000-000000000000"
        },
        {
            "key": "Client",
            "value": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.132 Safari/537.36"
        },
        {
            "key": "IdentityProviderName",
            "value": "facebook"
        },
        {
            "key": "IdentityProviderApplicationId",
            "value": "0000000000000000"
        },
        {
            "key": "ClientIpAddress",
            "value": "127.0.0.1"
        }
    ]
}
```

## <a name="next-steps"></a>后续步骤

可以自动执行其他管理任务，例如，[用 .net 管理用户](active-directory-b2c-devquickstarts-graph-dotnet.md)。
