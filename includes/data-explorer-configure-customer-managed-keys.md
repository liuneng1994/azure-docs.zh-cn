---
author: orspod
ms.service: data-explorer
ms.topic: include
ms.date: 01/07/2020
ms.author: orspodek
ms.openlocfilehash: 5c51a32c9dd82f2efe469d7a8844ed518b8f4d59
ms.sourcegitcommit: 02160a2c64a5b8cb2fb661a087db5c2b4815ec04
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/07/2020
ms.locfileid: "75725710"
---
Azure 数据资源管理器加密静态存储帐户中的所有数据。 默认情况下，使用 Microsoft 托管的密钥对数据进行加密。 为了进一步控制加密密钥，你可以提供客户管理的密钥用于数据加密。 客户管理的密钥必须存储在[Azure Key Vault](/azure/key-vault/key-vault-overview)中。 你可以创建自己的密钥并将其存储在密钥保管库中，也可以使用 Azure Key Vault API 来生成密钥。 Azure 数据资源管理器群集和密钥保管库必须位于同一区域，但它们可以位于不同的订阅中。 有关客户托管密钥的详细说明，请参阅[Azure Key Vault 的客户托管密钥](/azure/storage/common/storage-service-encryption)。 本文介绍如何配置客户管理的密钥。

> [!Note]
> 若要配置 Azure 数据资源管理器的客户托管密钥，必须[在密钥保管库上设置两个属性](/azure/key-vault/key-vault-ovw-soft-delete)：**软删除**和不**清除**。 默认情况下不启用这些属性。 若要启用这些属性，请使用[PowerShell](/azure/key-vault/key-vault-soft-delete-powershell)或[Azure CLI](/azure/key-vault/key-vault-soft-delete-cli)。 仅支持 RSA 密钥和密钥大小2048。

## <a name="assign-an-identity-to-the-cluster"></a>为群集分配标识

若要为群集启用客户管理的密钥，请首先向群集分配系统分配的托管标识。 你将使用此托管标识授予群集访问密钥保管库的权限。 若要配置系统分配的托管标识，请参阅[托管标识](/azure/data-explorer/managed-identities)。

## <a name="create-a-new-key-vault"></a>创建新的密钥保管库

若要使用 PowerShell 创建新的密钥保管库，请调用[AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault.md)。 用于存储 Azure 数据资源管理器加密的客户托管密钥的密钥保管库必须启用两个密钥保护设置，**软删除**并不**清除**。 在下面的示例中，将占位符中的占位符值替换为你自己的值。

```azurepowershell-interactive
$keyVault = New-AzKeyVault -Name <key-vault> `
    -ResourceGroupName <resource_group> `
    -Location <location> `
    -EnableSoftDelete `
    -EnablePurgeProtection
```

## <a name="configure-the-key-vault-access-policy"></a>配置密钥保管库访问策略

接下来，为密钥保管库配置访问策略，以便群集有权访问它。 在此步骤中，你将使用之前分配给群集的系统分配的托管标识。 若要设置密钥保管库的访问策略，请调用[AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy.md)。 将括号中的占位符值替换为自己的值，并使用在前面的示例中定义的变量。

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy `
    -VaultName $keyVault.VaultName `
    -ObjectId $cluster.Identity.PrincipalId `
    -PermissionsToKeys wrapkey,unwrapkey,get,recover
```

## <a name="create-a-new-key"></a>新建密钥

接下来，在密钥保管库中创建新密钥。 若要创建新密钥，请调用[AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey.md)。 将括号中的占位符值替换为自己的值，并使用在前面的示例中定义的变量。

```azurepowershell-interactive
$key = Add-AzKeyVaultKey -VaultName $keyVault.VaultName -Name <key> -Destination 'Software'
```
