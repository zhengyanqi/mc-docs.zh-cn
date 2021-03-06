---
title: include 文件
description: include 文件
services: virtual-machines
author: rockboyfor
ms.service: virtual-machines
ms.topic: include
origin.date: 03/09/2018
ms.date: 04/16/2018
ms.author: v-yeche
ms.custom: include file
ms.openlocfilehash: 5c3733558a7c1125e22574499f0fe6d37cc1b7d7
ms.sourcegitcommit: 6e80951b96588cab32eaff723fe9f240ba25206e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/16/2018
---
1. 单击“连接”创建和下载远程桌面协议文件（.rdp 文件）。 单击“打开”以使用此文件。
2. 此时会出现“`.rdp` 文件来自未知发布者”的警告。 这是一般警报。 在“远程桌面”窗口中，单击“连接”继续。

    ![有关未知发布者的警告的屏幕截图。](./media/virtual-machines-log-on-win-server/rdp-warn.png)
3. 在“Windows 安全性”窗口中，依次选择“更多选择”、“使用其他帐户”。 键入虚拟机上帐户的凭据，并单击“确定”。

     **本地帐户** - 通常，这是创建虚拟机时指定的本地帐户用户名和密码。 在本示例中，域是虚拟机的名称，输入格式为 *vmname*&#92;*username*。  

    **已加入域的 VM** - 如果 VM 属于某个域，请以*域*&#92;*用户名*格式输入用户名。 该帐户还需要属于管理员组或已被授予 VM 的远程访问权限。

    **域控制器** - 如果 VM 是域控制器，请键入该域的域管理员帐户的用户名和密码。
4. 单击“是”验证虚拟机的 ID 并完成登录。

   ![显示有关验证 VM 标识的消息的屏幕截图。](./media/virtual-machines-log-on-win-server/cert-warning.png)
<!-- Update_Description: update meta properties, wording udpate -->