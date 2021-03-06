---
title: 在 Azure Stack Hub 上安装事件中心的先决条件
description: 在 Azure Stack Hub 上安装事件中心资源提供程序之前，先了解所需的先决条件。
author: WenJason
ms.author: v-jay
ms.service: azure-stack
ms.topic: how-to
origin.date: 12/09/2019
ms.date: 08/31/2020
ms.reviewer: jfggdl
ms.lastreviewed: 12/09/2019
ms.openlocfilehash: 9a87cf735f6cab2e99e5458d8b4bbe451529475f
ms.sourcegitcommit: 4e2d781466e54e228fd1dbb3c0b80a1564c2bf7b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/26/2020
ms.locfileid: "88868095"
---
# <a name="prerequisites-for-installing-event-hubs-on-azure-stack-hub"></a>在 Azure Stack Hub 上安装事件中心的先决条件

[!INCLUDE [preview-banner](../includes/event-hubs-preview.md)]

必须先完成以下先决条件，然后才能在 Azure Stack Hub 上安装事件中心。 可能需要几天或几周的提前期才能完成所有步骤。

> [!IMPORTANT]
> 这些先决条件假定你至少已部署了 4 节点 Azure Stack Hub 集成系统。 Azure Stack 开发工具包 (ASDK) 不支持事件中心资源提供程序。

> [!IMPORTANT]
> 事件中心需要 Azure Stack Hub 2002 内部版本或更高版本。 请注意，Azure Stack Hub 的内部版本是递增的。 例如，如果已安装[版本 1910](./release-notes.md?view=azs-1910#1910-build-reference)，必须首先升级到 [2002](./release-notes.md?view=azs-2002#2002-build-reference)，然后再升级到 2005。 也就是说，不能跳过中间的内部版本。

## <a name="common-prerequisites"></a>常见的先决条件

[!INCLUDE [Common RP prerequisites](../includes/marketplace-resource-provider-prerequisites.md)]

## <a name="event-hubs-prerequisites"></a>事件中心先决条件

1. 采购事件中心的公钥基础结构 (PKI) SSL 证书。 使用者备用名称 (SAN) 必须遵循以下命名模式：`CN=*.eventhub.<region>.<fqdn>`。 可以指定“使用者名称”，但在处理证书时事件中心不会使用它。 仅使用使用者备用名称。 有关详细要求的完整列表，请参阅 [PKI 证书要求](azure-stack-pki-certs.md)。  

   ![示例证书](media/event-hubs-rp-prerequisites/certificate-example.png)

   > [!NOTE]
   > PFX 文件必须受密码保护。 稍后将在安装过程中请求密码。

2. 请务必查看[验证你的证书](azure-stack-validate-pki-certs.md)。 本文介绍如何准备和验证用于事件中心资源提供程序的证书。 

## <a name="next-steps"></a>后续步骤

接下来，[安装事件中心资源提供程序](event-hubs-rp-install.md)。
