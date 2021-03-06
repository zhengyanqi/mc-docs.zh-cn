---
title: Azure 安全中心的威胁防护
description: 本主题介绍受 Azure 安全中心威胁防护功能保护的资源
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 33c45447-3181-4b75-aa8e-c517e76cd50d
ms.service: security-center
ms.topic: conceptual
ms.date: 05/12/2020
ms.author: v-tawe
origin.date: 03/15/2020
ms.openlocfilehash: f5e4ba521f32663bbebeee82f7ff3db95d22465c
ms.sourcegitcommit: 41e986cd4a2879d8767dc6fc815c805e782dc7e6
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/20/2020
ms.locfileid: "90822356"
---
# <a name="threat-protection-in-azure-security-center"></a>Azure 安全中心的威胁防护

当安全中心检测到环境中的任何区域遭到威胁时，会生成警报。 这些警报会描述受影响资源的详细信息、建议的修正步骤，在某些情况下还会提供触发逻辑应用作为响应的选项。

Azure 安全中心威胁防护为环境提供全面的防御：

* **针对 Azure 计算资源的威胁防护**：Windows 计算机、Linux 计算机、Azure 应用服务和 Azure 容器

* **针对 Azure 数据资源的威胁防护**：SQL 数据库和 SQL 数据仓库、Azure 存储以及 Azure Cosmos DB

* **针对 Azure 服务层的威胁防护**：Azure 网络层、Azure 管理层（Azure 资源管理器）（预览版）和 Azure Key Vault（预览版）

<!-- not available-->




## <a name="threat-protection-for-windows-machines"></a>Windows 计算机的威胁防护 <a name="windows-machines"></a>

Azure 安全中心与 Azure 服务集成，可以监视和保护基于 Windows 的计算机。 安全中心以一种易于使用的格式显示所有这些服务的警报和修正建议。

* **Microsoft Defender ATP** <a name="windows-atp"></a> - 安全中心通过与 Microsoft Defender 高级威胁防护 (ATP) 集成来扩展其云工作负载保护平台。 它们共同提供了全面的终结点检测和响应 (EDR) 功能。

    > [!IMPORTANT]
    > 使用安全中心的 Windows 服务器上已自动启用 Microsoft Defender ATP 传感器。

    Microsoft Defender ATP 在检测到威胁时会触发警报。 警报显示在安全中心仪表板上。 在仪表板中，可以透视 Microsoft Defender ATP 控制台，并执行详细调查来发现攻击范围。 有关 Microsoft Defender ATP 的详细信息，请参阅[将服务器载入到 Microsoft Defender ATP 服务](https://docs.microsoft.com/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints)。

* **故障转储分析** <a name="windows-dump"></a> - 当软件出现故障时，故障转储可捕获发生故障时的部分内存。

    故障可能是由恶意软件造成的，或可能包含恶意软件。 为避免被安全产品检测到，各种形式的恶意软件都会使用无文件攻击，这可避免写入磁盘或对写入磁盘的软件组件进行加密。 使用传统的基于磁盘的方法很难检测到这种类型的攻击。

    但是，通过使用内存分析，可以检测到这种攻击。 通过分析故障转储中的内存，安全中心可以检测到攻击所使用的技术。 例如，攻击可能试图利用软件中的漏洞，访问机密数据以及隐秘地持久保存在受到攻击的计算机中。 安全中心可以在对主机性能产生最小影响的情况下执行此操作。

    有关故障转储分析警报的详细信息，请参阅[警报引用表](alerts-reference.md#alerts-windows)。

* **无文件攻击检测** <a name="windows-fileless"></a> - 针对终结点的无文件攻击很常见。 为了避免被检测到，无文件攻击会将恶意有效负载注入到内存中。 攻击者有效负载会持久保存在遭到入侵的进程的内存中，执行各种恶意活动。

    自动内存取证技术使用无文件攻击检测来识别无文件攻击工具包、方法和行为。 此解决方案会定期在运行时扫描计算机，并直接从安全关键型进程的内存中提取见解。

    它会找出恶意利用、代码注入和恶意有效负载执行的证据。 无文件攻击检测会生成详细的安全警报，以加速警报会审、关联和下游响应速度。 此方法是对基于事件的 EDR 解决方案的补充，提供更大的检测范围。

    有关无文件攻击检测警报的详细信息，请参阅[警报参考表](alerts-reference.md#alerts-windows)。

> [!TIP]
> 可以下载 [Azure 安全中心 Playbook：安全警报](https://gallery.technet.microsoft.com/Azure-Security-Center-f621a046)来模拟 Windows 警报。






## <a name="threat-protection-for-linux-machines"></a>针对 Linux 计算机的威胁防护 <a name="linux-machines"></a>

安全中心使用 **auditd**（最常见的 Linux 审核框架之一）从 Linux 计算机收集审核记录。 auditd 驻留在主线内核中。 

* **Linux auditd 警报和 Log Analytics 代理集成** <a name="linux-auditd"></a> - auditd 系统包含一个负责监视系统调用的内核级子系统。 该子系统会按照指定的规则集筛选这些调用，并将针对这些调用生成的消息写入到套接字。 安全中心在 Log Analytics 代理中集成了 auditd 包的功能。 通过这种集成，无需满足任何先决条件，就能在所有受支持的 Linux 发行版中收集 auditd 事件。

    可以使用适用于 Linux 的 Log Analytics 代理收集、扩充 auditd 记录并将其聚合到事件中。 安全中心会持续添加使用 Linux 信号检测云和本地 Linux 计算机上的恶意行为的新分析。 与 Windows 功能类似，这些分析涵盖可疑进程、可疑登录尝试、内核模块加载以及其他活动。 这些活动可能表明计算机正在受到攻击或已被破坏。  

    有关 Linux 警报的列表，请参阅[警报参考表](alerts-reference.md#alerts-linux)。

> [!TIP]
> 可以下载 [Azure 安全中心 Playbook：Linux 检测](https://gallery.technet.microsoft.com/Azure-Security-Center-0ac8a5ef)来模拟 Linux 警报。





## <a name="threat-protection-for-azure-app-service"></a>针对 Azure 应用服务的威胁防护 <a name="app-services"></a>

> [!NOTE]
> 此服务目前在 Azure 政府云和主权云区域中不可用。

安全中心使用云的规模来识别针对应用服务运行的应用程序受到的攻击。 攻击者会探查 Web 应用程序，以找出并恶意利用其中的弱点。 向 Azure 中运行的应用程序发出的请求在路由到特定的环境之前会流经多个网关，网关会对其进行检查并记录其状态。 然后，将使用这些数据来识别恶意利用行为和攻击者，并了解稍后要使用的新模式。

凭借云提供商 Azure 提供的可见性，安全中心可以分析应用服务的内部日志，以识别多个目标上的攻击方法。 例如，方法包括大范围扫描和分布式攻击。 此类攻击通常来自一小部分 IP，会展示多个主机上类似终结点的爬网模式。 攻击会搜索有漏洞的页面或插件，从单个主机的角度无法识别到。

如果运行基于 Windows 的应用服务计划，则安全中心还可以访问底层沙盒与 VM。 结合上面所述的日志数据，基础结构可以讲述从四处传播的新攻击，到客户计算机遭到入侵的整个历程。 因此，即使是在 Web 应用被恶意利用后再部署安全中心，它也能够检测不断发生的攻击。

有关 Azure 应用服务警报的列表，请参阅[警报引用表](alerts-reference.md#alerts-azureappserv)。






## <a name="threat-protection-for-azure-containers"></a>针对 Azure 容器的威胁防护 <a name="azure-containers"></a>

> [!NOTE]
> 此服务目前在 Azure 政府云和主权云区域中不可用。

安全中心为容器化环境提供实时威胁防护，并针对可疑活动生成警报。 可以使用此信息快速补救安全问题，并提高容器的安全性。

安全中心在不同的级别提供威胁防护： 

* **主机级别** - 安全中心的代理（在“标准”层中提供。有关详细信息，请参阅[定价](security-center-pricing.md)）监视 Linux 中的可疑活动。 该代理针对源自某个节点或其上运行的容器的可疑活动触发警报。 此类活动的示例包括使用已知可疑的 IP 地址进行 Web shell 检测和连接。

    为了更深入地洞察容器化环境的安全性，该代理会监视容器特定的分析结果。 它会针对创建特权容器、以可疑方式访问 API 服务器以及在 Docker 容器内部运行安全外壳 (SSH) 服务器等事件触发警报。

    >[!IMPORTANT]
    > 如果你选择不在主机上安装代理，则只能收到一部分威胁防护权益和安全警报。 你仍会收到与网络分析以及与恶意服务器通信相关的警报。

    有关主机级别的警报列表，请参阅[警报参考表](alerts-reference.md#alerts-containerhost)。


* 在 **AKS 群集级别**，威胁防护基于 Kubernetes 审核日志的分析。 若要启用此**无代理**监视功能，请在“定价和设置”页中将“Kubernetes”选项添加到订阅（请参阅[定价](security-center-pricing.md)）。 为了在此级别生成警报，安全中心将使用 AKS 检索到的日志来监视 AKS 管理的服务。 此级别的事件示例包括公开 Kubernetes 仪表板、创建高特权角色，以及创建敏感的装入点。

    >[!NOTE]
    > 安全中心针对在订阅设置中启用“Kubernetes”选项后发生的 Azure Kubernetes 服务操作和部署生成安全警报。 

    有关 AKS 群集级别的警报列表，请参阅[警报参考表](alerts-reference.md#alerts-akscluster)。

此外，我们的全球安全研究团队会不断监视威胁态势。 一旦发现威胁，他们就会添加容器特定的警报和漏洞。

> [!TIP]
> 可以按照[此博客文章](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-demonstrate-the-new-containers-features-in-azure-security/ba-p/1011270)中的说明来模拟容器警报。








## <a name="threat-protection-for-sql-database-and-sql-data-warehouse"></a>针对 SQL 数据库和 SQL 数据仓库的威胁防护 <a name="data-sql"></a>

针对 Azure SQL 数据库的高级威胁防护会检测异常活动，这些活动指示存在访问或恶意利用数据库的非寻常且可能有害的企图。

出现可疑的数据库活动、潜在漏洞，或者 SQL 注入攻击以及异常的数据库访问和查询模式时，你会看到警报。

针对 Azure SQL 数据库和 SQL 的高级威胁防护包含在高级 SQL 安全功能的[高级数据安全 (ADS)](https://docs.azure.cn/sql-database/sql-database-advanced-data-security) 统一包中，其保护范围涵盖了 Azure SQL 数据库、Azure SQL 数据库托管实例、Azure SQL 数据仓库数据库，以及 Azure 虚拟机上的 SQL 服务器。

有关详细信息，请参阅：

* [如何为 Azure SQL 数据库启用高级威胁防护](https://docs.azure.cn/sql-database/sql-database-threat-detection-overview)
* [针对 SQL 数据库和 SQL 数据仓库的威胁防护警报列表](alerts-reference.md#alerts-sql-db-and-warehouse)




## <a name="threat-protection-for-azure-storage"></a>针对 Azure 存储的威胁防护 <a name="azure-storage"></a>

针对存储的高级威胁防护会检测访问或恶意利用存储帐户的非寻常的和可能有害的企图。 这一层保护使你无需成为安全专家便可以解决威胁，并可帮助你管理安全监视系统。

如需定价详细信息（包括 30 天免费试用版的信息），请参阅 [Azure 安全中心定价页](https://www.azure.cn/pricing/details/security-center/)。

有关详细信息，请参阅：

<!-- not available-->
* [Azure 存储的威胁防护警报列表](alerts-reference.md#alerts-azurestorage)


> [!TIP]
> 可以按照[此博客文章](https://techcommunity.microsoft.com/t5/azure-security-center/validating-atp-for-azure-storage-detections-in-azure-security/ba-p/1068131)中的说明来模拟 Azure 存储警报。




## <a name="threat-protection-for-azure-cosmos-db"></a>针对 Azure Cosmos DB 的威胁防护 <a name="cosmos-db"></a>

当有人企图以非寻常和可能有害的方式访问或恶意利用 Azure Cosmos DB 帐户时，会生成 Azure Cosmos DB 警报。

有关详细信息，请参阅：

<!-- not available-->
* [Azure Cosmos DB 的威胁防护警报列表（预览）](alerts-reference.md#alerts-azurecosmos)




## <a name="threat-protection-for-azure-network-layer"></a>Azure 网络层的威胁防护 <a name="network-layer"></a>

安全中心网络层分析基于示例 IPFIX 数据（即 Azure 核心路由器收集的数据包标头）。 根据此数据馈送，安全中心将使用机器学习模型来识别和标记恶意流量活动。 安全中心还使用 Microsoft 威胁情报数据库来扩充 IP 地址。

某些网络配置可能会限制安全中心对可疑网络活动生成警报。 要使安全中心生成网络警报，请确保：

- 虚拟机有一个公共 IP 地址（或位于使用公共 IP 地址的负载均衡器上）。

- 虚拟机的网络出口流量未被外部 IDS 解决方案阻止。

- 在发生可疑通信的整小时内，为虚拟机分配了同一个 IP 地址。 这也适用于作为托管服务（例如 AKS、Databricks）的一部分创建的 VM。

有关 Azure 网络层警报的列表，请参阅[警报参考表](alerts-reference.md#alerts-azurenetlayer)。

若要详细了解安全中心如何使用网络相关信号来应用威胁防护，请参阅[安全中心内的启发式 DNS 检测](https://azure.microsoft.com/blog/heuristic-dns-detections-in-azure-security-center/)。



## <a name="threat-protection-for-azure-management-layer-azure-resource-manager-preview"></a>针对 Azure 管理层（Azure 资源管理器）（预览版）的威胁防护 <a name ="management-layer"></a>

基于 Azure 资源管理器的安全中心保护层目前为预览版。

安全中心通过 Azure 资源管理器事件提供额外的保护层，该保护层被视为 Azure 的控制平面。 通过分析 Azure 资源管理器记录，安全中心可以检测 Azure 订阅环境中的非寻常或可能有害的操作。

有关 Azure 资源管理器（预览版）警报的列表，请参阅[警报参考表](alerts-reference.md#alerts-azureresourceman)。



>[!NOTE]
> 上述几项分析由 Microsoft Cloud App Security 提供支持。 若要从这些分析中获益，必须激活 Cloud App Security 许可证。 如果你有 Cloud App Security 许可证，则默认会启用这些警报。 若要禁用警报：
>
> 1. 在“安全中心”边栏选项卡中选择“安全策略”。  针对要更改的订阅，选择“编辑设置”。
> 2. 选择“威胁检测”。
> 3. 在“启用集成”下，清除“允许 Microsoft Cloud App Security 访问我的数据”，然后选择“保存”。  

>[!NOTE]
>安全中心将安全相关的客户数据存储在其资源所在的地理区域。 如果 Microsoft 尚未在该资源所在的地理位置部署安全中心，则数据将存储在美国。 启用 Cloud App Security 后，将根据 Cloud App Security 的地理位置规则存储此信息。 有关详细信息，请参阅[非区域性服务的数据存储](https://azuredatacentermap.azurewebsites.net/)。








## <a name="threat-protection-for-azure-key-vault-preview"></a>针对 Azure Key Vault（预览版）的威胁防护<a name="azure-keyvault"></a>

> [!NOTE]
> 此服务目前在 Azure 政府云和主权云区域中不可用。

Azure 密钥保管库是一种云服务，用于保护加密密钥和机密（例如证书、连接字符串和密码）。 

Azure 安全中心包含针对 Azure Key Vault 的 Azure 原生高级威胁防护，提供额外的安全情报层。 安全中心会检测以非寻常和可能有害方式访问或恶意利用 Key Vault 帐户的企图。 借助此保护层，无需成为安全专家，也无需管理第三方安全监视系统，即可应对威胁。  

当发生异常活动时，安全中心会显示警报，并选择性地通过电子邮件将警报发送给订阅管理员。 这些警报包含可疑活动的详细信息，以及有关如何调查和修正威胁的建议。 

有关 Azure Key Vault 警报的列表，请参阅[警报参考表](alerts-reference.md#alerts-azurekv)。





## <a name="threat-protection-for-other-microsoft-services"></a>针对其他 Microsoft 服务的威胁防护 <a name="alerts-other"></a>

### <a name="threat-protection-for-azure-waf"></a>针对 Azure WAF 的威胁防护 <a name="azure-waf"></a>

Azure 应用程序网关提供的 Web 应用程序防火墙 (WAF) 可以对 Web 应用程序进行集中保护，避免其受到常见的攻击和漏洞伤害。

Web 应用程序已逐渐成为利用常见已知漏洞的恶意攻击的目标。 应用程序网关 WAF 基于开放 Web 应用程序安全项目中的核心规则集 3.0 或 2.2.9。 WAF 会自动更新，以便在出现新漏洞后提供保护。 

如果你有 Azure WAF 许可证，则无需进行额外的配置，就会将 WAF 警报流式传输到安全中心。 有关 WAF 生成的警报的详细信息，请参阅 [Web 应用程序防火墙 CRS 规则组和规则](../web-application-firewall/ag/application-gateway-crs-rulegroups-rules.md?tabs=owasp31#crs911-31)。


<!-- DDoS Protection is not available -->


## <a name="next-steps"></a>后续步骤
若要详细了解这些威胁防护功能生成的安全警报，请参阅以下文章：

* [所有 Azure 安全中心警报的参考表](alerts-reference.md)
* [Azure 安全中心中的安全警报](security-center-alerts-overview.md)
* [在 Azure 安全中心内管理和响应安全警报](security-center-managing-and-responding-alerts.md)
