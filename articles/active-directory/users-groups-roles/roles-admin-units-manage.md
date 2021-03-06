---
title: 添加和删除管理单元（预览版）- Azure Active Directory | Microsoft Docs
description: 在 Azure Active Directory 中使用管理单元来限制角色权限的范围。
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: users-groups-roles
ms.workload: identity
ms.date: 08/26/2020
ms.author: v-junlch
ms.reviewer: anandy
ms.custom: oldportal;it-pro;
ms.collection: M365-identity-device-management
ms.openlocfilehash: d21ca9e23e8fca0efd150d262ffadabbdb47a592
ms.sourcegitcommit: b5ea35dcd86ff81a003ac9a7a2c6f373204d111d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2020
ms.locfileid: "88947520"
---
# <a name="manage-administrative-units-in-azure-active-directory"></a>在 Azure Active Directory 中管理管理单元

若要在 Azure Active Directory (Azure AD) 中实现更精细的管理控制，可将用户分配到作用域限定为一个或多个管理单元 (AU) 的 Azure AD 角色。

## <a name="add-an-administrative-unit"></a>添加管理单元

### <a name="use-the-azure-portal"></a>使用 Azure 门户

1. 在 Azure 门户中转到 Azure AD，然后在左窗格中选择“管理单元”。

    ![Azure AD 中的“管理单元(预览版)”链接的屏幕截图](./media/roles-admin-units-manage/nav-to-admin-units.png)

1. 选择“添加”，然后输入管理单元的名称。 （可选）添加管理单元的说明。

    ![用于输入管理单元名称的“添加”按钮和文本框的屏幕截图](./media/roles-admin-units-manage/add-new-admin-unit.png)

1. 选择“添加”以完成管理单元。

### <a name="use-powershell"></a>使用 PowerShell

在尝试运行以下命令之前，请安装 Azure AD PowerShell（预览版）：

```powershell
Connect-AzureAD -AzureEnvironmentName AzureChinaCloud
New-AzureADAdministrativeUnit -Description "Coast region" -DisplayName "Coast"
```

可以根据需要修改用引号引起来的值。

### <a name="use-microsoft-graph"></a>使用 Microsoft Graph

```http
Http Request
POST /administrativeUnits
Request body
{
  "displayName": "China North Operations",
  "description": "China North Operations administration"
}
```

## <a name="remove-an-administrative-unit"></a>删除管理单元

在 Azure AD 中，可以删除不再需要作为管理角色的作用域的管理单元。

### <a name="use-the-azure-portal"></a>使用 Azure 门户

1. 在 Azure 门户中，转到“Azure AD” > “管理单元”。 
1. 键入要删除的管理单元，然后选择“删除”。 
1. 若要确认是否要删除管理单元，请选择“是”。 此时会删除管理单元。

![管理单元的“删除”按钮和确认窗口的屏幕截图](./media/roles-admin-units-manage/select-admin-unit-to-delete.png)

### <a name="use-powershell"></a>使用 PowerShell

```powershell
$delau = Get-AzureADAdministrativeUnit -Filter "displayname eq 'DeleteMe Admin Unit'"
Remove-AzureADAdministrativeUnit -ObjectId $delau.ObjectId
```

可以根据特定环境的需要修改用引号引起来的值。

### <a name="use-the-graph-api"></a>使用 Graph API

```http
HTTP request
DELETE /administrativeUnits/{Admin id}
Request body
{}
```

## <a name="next-steps"></a>后续步骤

* [对管理单元中的用户的管理](roles-admin-units-add-manage-users.md)
* [管理管理单元中的组](roles-admin-units-add-manage-groups.md)

