---
title: include 文件
description: include 文件
services: vpn-gateway
author: WenJason
ms.service: vpn-gateway
ms.topic: include
origin.date: 03/21/2018
ms.date: 12/24/2018
ms.author: v-jay
ms.custom: include file
ms.openlocfilehash: e55d548fc8ff919dd1806ccb9177ed625a97c05f
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "63860339"
---
### <a name="to-modify-local-network-gateway-ip-address-prefixes---no-gateway-connection"></a><a name="noconnection"></a>修改本地网关 IP 地址前缀 - 无网关连接

#### <a name="to-add-additional-address-prefixes"></a>添加其他地址前缀：

1. 在“本地网络网关”资源的“设置”  部分中，单击“配置”  。
2. 在“添加更多地址范围”  框中添加 IP 地址空间。
3. 单击“保存”  以保存设置。

#### <a name="to-remove-address-prefixes"></a>删除地址前缀：

1. 在“本地网络网关”资源的“设置”  部分中，单击“配置”  。
2. 在包含要删除的前缀的行上，单击“...”  。
3. 单击“删除”。 
4. 单击“保存”  以保存设置。

### <a name="to-modify-local-network-gateway-ip-address-prefixes---existing-gateway-connection"></a><a name="withconnection"></a>修改本地网关 IP 地址前缀 - 现有网关连接

如果有一个网关连接并且想要添加或删除包含在本地网关中的 IP 地址前缀，则需要按顺序执行以下步骤。 这会导致 VPN 连接中断一段时间。 修改 IP 地址前缀时，不需删除 VPN 网关。 只需删除连接。

#### <a name="1-remove-the-connection"></a>1.删除连接。

1. 在“本地网络网关”资源的“设置”  部分中，单击“连接”  。
2. 在每个连接的行上单击“...”  ，然后单击“删除”  。
3. 单击“保存”  以保存设置。

#### <a name="2-modify-the-address-prefixes"></a>2.修改地址前缀。

添加其他地址前缀：

1. 在“本地网络网关”资源的“设置”  部分中，单击“配置”  。
2. 添加 IP 地址空间。
3. 单击“保存”  以保存设置。

删除地址前缀：

1. 在“本地网络网关”资源的“设置”  部分中，单击“配置”  。
2. 在含有要删除的前缀的行上，单击“...”  。
3. 单击“删除”。 
4. 单击“保存”  以保存设置。

#### <a name="3-recreate-the-connection"></a>3.重新创建连接。

1. 导航到 VNet 的虚拟网络网关。 （不是本地网络网关。）
2. 在“虚拟网络网关”的“设置”  部分中，单击“连接”  。
3. 单击“+ 添加”  打开“添加连接”  边栏选项卡。
4. 重新创建连接。
5. 单击“确定”创建连接。 