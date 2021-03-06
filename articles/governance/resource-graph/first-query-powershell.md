---
title: 快速入门：第一个 PowerShell 查询
description: 本快速入门介绍为 Azure PowerShell 启用 Resource Graph 模块并运行第一个查询的步骤。
author: DCtheGeek
ms.author: v-tawe
origin.date: 08/10/2020
ms.date: 09/15/2020
ms.topic: quickstart
ms.openlocfilehash: c4f80d27b878c295832fa7601ebb60357c411e2a
ms.sourcegitcommit: 75299b1cb5540a11149f320edaae82ae8c03c16b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90523145"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-azure-powershell"></a>快速入门：使用 Azure PowerShell 运行首个 Resource Graph 查询

使用 Azure Resource Graph 的第一步是查看是否已为 Azure PowerShell 安装了该模块。 本快速入门将指导你完成将该模块添加到 Azure PowerShell 安装的过程。

在此过程结束时，应该已将模块添加到所选的 Azure PowerShell 安装中，并运行首个 Resource Graph 查询。

## <a name="prerequisites"></a>先决条件

如果没有 Azure 订阅，请在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

## <a name="add-the-resource-graph-module"></a>添加 Resource Graph 模块

要使 Azure PowerShell 能够查询 Azure Resource Graph，则必须添加该模块。 此模块可以与本地安装的 PowerShell 一起使用，也可以与 [PowerShell Docker 映像](https://hub.docker.com/_/microsoft-powershell)一起使用。

### <a name="base-requirements"></a>基本要求

Azure Resource Graph 模块需要以下软件：

- Azure PowerShell 1.0.0 或更高版本。 若尚未安装，请遵循[这些说明](https://docs.microsoft.com/powershell/azure/install-az-ps)。

- PowerShellGet 2.0.1 或更高版本。 若尚未安装或更新，请遵循[这些说明](https://docs.microsoft.com/powershell/scripting/gallery/installing-psget)。

### <a name="install-the-module"></a>安装模块

适用于 PowerShell 的 Resource Graph 模块是 **Az.ResourceGraph**。

1. 从管理 PowerShell 提示符运行以下命令：

   ```azurepowershell
   # Install the Resource Graph module from PowerShell Gallery
   Install-Module -Name Az.ResourceGraph
   ```

1. 验证该模块是否已导入且是否为最新版本 (0.7.5)：

   ```azurepowershell
   # Get a list of commands for the imported Az.ResourceGraph module
   Get-Command -Module 'Az.ResourceGraph' -CommandType 'Cmdlet'
   ```

## <a name="run-your-first-resource-graph-query"></a>运行首个 Resource Graph 查询

将 Azure PowerShell 模块添加到所选环境中后，即可尝试一个简单的 Resource Graph 查询。 该查询返回前五个 Azure 资源，以及每个资源的名称和资源类型 。

1. 使用 `Search-AzGraph` cmdlet 运行首个 Azure Resource Graph 查询：

   ```azurepowershell
   # Login first with Connect-AzAccount -Environment AzureChinaCloud

   # Run Azure Resource Graph query
   Search-AzGraph -Query 'Resources | project name, type | limit 5'
   ```

   > [!NOTE]
   > 由于此查询示例未提供排序修饰符（例如 `order by`），因此多次运行此查询可能会为每个请求生成一组不同的资源。

1. 将查询更新为 `order by` Name 属性：

   ```azurepowershell
   # Run Azure Resource Graph query with 'order by'
   Search-AzGraph -Query 'Resources | project name, type | limit 5 | order by name asc'
   ```

   > [!NOTE]
   > 与第一个查询一样，多次运行此查询可能会为每个请求生成一组不同的资源。 查询命令的顺序非常重要。 在本例中，`order by` 位于 `limit` 之后。 命令按此顺序执行，首先会限制查询结果，然后对它们进行排序。

1. 将查询更新为先 `order by` Name 属性，然后再 `limit` 为前五个结果：

   ```azurepowershell
   # Run Azure Resource Graph query with `order by` first, then with `limit`
   Search-AzGraph -Query 'Resources | project name, type | order by name asc | limit 5'
   ```

假设环境中没有任何变化，则多次运行最后一个查询时，返回的结果将是一致的且按 Name 属性排序，但仍限制为前五个结果。

> [!NOTE]
> 如果查询未从你已有权访问的订阅返回结果，请注意 `Search-AzGraph` cmdlet 默认为默认上下文中的订阅。 若要查看作为默认上下文一部分的订阅 ID 列表，请运行此 `(Get-AzContext).Account.ExtendedProperties.Subscriptions`。如果你希望搜索你有权访问的所有订阅，可以通过运行 `$PSDefaultParameterValues=@{"Search-AzGraph:Subscription"= $(Get-AzSubscription).ID}` 为 `Search-AzGraph` cmdlet 设置 PSDefaultParameterValues
   
## <a name="clean-up-resources"></a>清理资源

若希望从 Azure PowerShell 环境中删除 Resource Graph 模块，可使用以下命令：

```azurepowershell
# Remove the Resource Graph module from the current session
Remove-Module -Name 'Az.ResourceGraph'

# Uninstall the Resource Graph module from the environment
Uninstall-Module -Name 'Az.ResourceGraph'
```

> [!NOTE]
> 该操作不会删除之前下载的模块文件。 只会从正在运行的 PowerShell 会话中将其删除。

## <a name="next-steps"></a>后续步骤

本快速入门介绍了如何将 Resource Graph 模块添加到 Azure PowerShell 环境并运行第一个查询。 若要详细了解 Resource Graph 语言，请继续阅读查询语言详细信息页。

> [!div class="nextstepaction"]
> [获取有关查询语言的详细信息](./concepts/query-language.md)