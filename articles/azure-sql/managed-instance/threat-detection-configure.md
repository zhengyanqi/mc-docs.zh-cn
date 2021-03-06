---
title: 配置高级威胁防护
titleSuffix: Azure SQL Managed Instance
description: 高级威胁防护可检测异常的数据库活动，这些活动指示 Azure SQL 托管实例中存在对数据库的潜在安全威胁。
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: WenJason
ms.author: v-jay
ms.reviewer: vanto
origin.date: 08/05/2019
ms.date: 07/13/2020
ms.openlocfilehash: eca1f75da98adeb492fd7e12827ffbb9dc61e4b9
ms.sourcegitcommit: fa26665aab1899e35ef7b93ddc3e1631c009dd04
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/10/2020
ms.locfileid: "86228105"
---
# <a name="configure-advanced-threat-protection-in-azure-sql-managed-instance"></a>在 Azure SQL 托管实例中配置高级威胁防护
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

[Azure SQL 托管实例](sql-managed-instance-paas-overview.md)的[高级威胁防护](../database/threat-detection-overview.md)可检测异常活动，这些活动指示存在访问或恶意利用数据库的异常且可能有害的企图。 高级威胁防护可以识别**潜在的 SQL 注入**、**来自异常位置或数据中心的访问**、**来自陌生主体或可能有害的应用程序的访问**以及**暴力破解 SQL 凭据** - 请在[高级威胁防护警报](../database/threat-detection-overview.md#alerts)中查看更多详细信息。

你可以通过[电子邮件通知](../database/threat-detection-overview.md#explore-detection-of-a-suspicious-event)或 [Azure 门户](../database/threat-detection-overview.md#explore-alerts-in-the-azure-portal)接收有关检测到的威胁的通知

[高级威胁防护](../database/threat-detection-overview.md)包含在[高级数据安全](../database/advanced-data-security.md)产品/服务中，这是用于高级 SQL 安全功能的统一软件包。 可通过中心 SQL ADS 门户访问和管理高级威胁防护。

##  <a name="azure-portal"></a>Azure 门户

1. 登录 [Azure 门户](https://portal.azure.cn)。 
2. 导航到要保护的 SQL 托管实例的实例配置页面。 在“设置”页中，选择“高级数据安全” 。
3. 在“高级数据安全”配置页中，执行以下操作：
   - **启用**高级数据安全。
   - 配置在检测到异常数据库活动时需要接收安全警报的**电子邮件列表**。
   - 选择保存异常的威胁审核记录的 **Azure 存储帐户**。
   - 选择要配置的**高级威胁防护类型**。 详细了解[高级威胁防护警报](../database/threat-detection-overview.md)。
4. 单击“保存”，保存新的或更新的高级数据安全策略。

   ![高级威胁防护](./media/threat-detection-configure/threat-detection.png)


## <a name="next-steps"></a>后续步骤

- 详细了解[高级威胁防护](../database/threat-detection-overview.md)。
- 如需了解有关托管实例的信息，请参阅[什么是 Azure SQL 托管实例](sql-managed-instance-paas-overview.md)。
- 详细了解 [Azure SQL 数据库的高级威胁防护](../database/threat-detection-configure.md)。
- 详细了解 [SQL 托管实例审核](/sql-database/sql-database-managed-instance-auditing)。
- 详细了解 [Azure 安全中心](/security-center/security-center-intro)。
