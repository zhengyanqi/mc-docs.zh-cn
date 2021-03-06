---
title: include 文件
description: include 文件
services: virtual-machines
author: rockboyfor
ms.service: virtual-machines
ms.topic: include
origin.date: 03/18/2020
ms.date: 07/06/2020
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 117d43871a8ec79bdc67b5b33082b7371abcf569
ms.sourcegitcommit: 89118b7c897e2d731b87e25641dc0c1bf32acbde
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/03/2020
ms.locfileid: "85945760"
---
## <a name="create-an-image-gallery"></a>创建映像库 

映像库是用于启用映像共享的主要资源。 允许用于库名称的字符为大写或小写字母、数字、点和句点。 库名称不能包含短划线。 库名称在你的订阅中必须唯一。 

使用 [New-AzGallery](https://docs.microsoft.com/powershell/module/az.compute/new-azgallery) 创建映像库。 以下示例在“myGalleryRG”资源组中创建名为“myGallery”的库 。

```powershell
$resourceGroup = New-AzResourceGroup `
   -Name 'myGalleryRG' `
   -Location 'China North'  
$gallery = New-AzGallery `
   -GalleryName 'myGallery' `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -Description 'Shared Image Gallery for my organization'  
```

## <a name="share-the-gallery"></a>共享库

建议在映像库级别共享访问权限。 使用电子邮件地址和 [Get-AzADUser](https://docs.microsoft.com/powershell/module/az.resources/get-azaduser) cmdlet 获取用户的对象 ID，然后使用 [New-AzRoleAssignment](https://docs.microsoft.com/powershell/module/Az.Resources/New-AzRoleAssignment) 为用户授予对库的访问权限。 请将此示例中的示例电子邮件地址 alinne_montes@contoso.com 替换为你自己的信息。

```powershell
# Get the object ID for the user
$user = Get-AzADUser -StartsWith alinne_montes@contoso.com
# Grant access to the user for our gallery
New-AzRoleAssignment `
   -ObjectId $user.Id `
   -RoleDefinitionName Reader `
   -ResourceName $gallery.Name `
   -ResourceType Microsoft.Compute/galleries `
   -ResourceGroupName $resourceGroup.ResourceGroupName
```

<!-- Update_Description: update meta properties, wording update, update link -->