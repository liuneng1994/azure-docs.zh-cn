---
title: 应用程序同意操作的工作原理 |Microsoft 文档
description: 详细了解 Azure AD 同意框架的工作原理以便了解如何在 Azure AD上开发应用程序时使用该框架
services: active-directory
author: rwike77
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: ryanwi
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0c79d9226b3a8965ad6ce853350f2e4c46617b5b
ms.sourcegitcommit: c38a1f55bed721aea4355a6d9289897a4ac769d2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/05/2019
ms.locfileid: "74843421"
---
# <a name="how-application-consent-works"></a>应用程序同意操作的工作原理

本文旨在帮助读者深入了解 Azure AD 同意框架的工作原理以便更有效地开发应用程序。

## <a name="recommended-documents"></a>建议的文档

- 概括性了解：[允许资源所有者如何通过同意操作管理应用程序对资源的访问权限](https://docs.microsoft.com/azure/active-directory/develop/active-directory-dev-glossary#consent)。
- 获取有关 [Azure AD 同意框架如何实现同意](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)的分步概述。
- 有关详细信息，请参阅[多租户应用程序如何使用同意框架](https://docs.microsoft.com/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview)实现“用户”和“管理员”同意（支持更多高级多层应用程序模式）。
- 有关详细信息，请参阅[在授权代码授权流期间如何在 OAuth 2.0 协议层支持同意](https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-oauth-code#request-an-authorization-code)

## <a name="next-steps"></a>后续步骤
[AzureAD StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)
