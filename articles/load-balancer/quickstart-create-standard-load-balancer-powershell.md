---
title: 快速入门：创建公共负载均衡器 - Azure PowerShell
titleSuffix: Azure Load Balancer
description: 本快速入门介绍如何使用 Azure PowerShell 创建负载均衡器
services: load-balancer
documentationcenter: na
author: WenJason
manager: digimobile
Customer intent: I want to create a load balancer so that I can load balance internet traffic to VMs.
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
origin.date: 01/27/2020
ms.date: 04/06/2020
ms.author: v-jay
ms:custom: seodec18
ms.openlocfilehash: 90db6b157ea758eec11e6d918ba810da4a94ca67
ms.sourcegitcommit: f8ed85740f873c15c239ab6ba753e4b76e030ba7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/28/2020
ms.locfileid: "89045815"
---
# <a name="quickstart-create-a-public-load-balancer-to-load-balance-vms-using-azure-powershell"></a>快速入门：使用 Azure PowerShell 创建公共负载均衡器以对 VM 进行负载均衡

使用 Azure PowerShell 创建公共负载均衡器和三个虚拟机，通过这种方式开始使用 Azure 负载均衡器。

## <a name="prerequisites"></a>先决条件

- 具有活动订阅的 Azure 帐户。 [创建试用帐户](https://wd.azure.cn/zh-cn/pricing/1rmb-trial-full/?form-type=identityauth)。
- 在本地安装了 Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

本文需要 Azure PowerShell 模块 5.4.1 或更高版本。 运行 `Get-Module -ListAvailable Az` 查找已安装的版本。 如果需要升级，请参阅[安装 Azure PowerShell 模块](https://docs.microsoft.com/powershell/azure/install-Az-ps)。 如果在本地运行 PowerShell，则还需运行 `Connect-AzAccount -Environment AzureChinaCloud` 以创建与 Azure 的连接。

## <a name="create-a-resource-group"></a>创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。

使用 [New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup) 创建资源组：

* 命名为“myResourceGroupLB”。
* 在“chinaeast2”位置。

```azurepowershell
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'chinaeast2'

New-AzResourceGroup -Name $rg -Location $loc
```
---

# <a name="option-1-default-create-a-public-load-balancer-standard-sku"></a>[选项 1（默认）：创建公共负载均衡器（标准 SKU）](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>对于生产型工作负载，建议使用标准 SKU 负载均衡器。 有关 sku 的详细信息，请参阅 [Azure 负载均衡器 SKU](skus.md)。

## <a name="create-a-public-ip-address"></a>创建公共 IP 地址

若要通过 Internet 访问 Web 应用，需要负载均衡器有一个公共 IP 地址。 

使用 [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) 执行以下操作：

* 创建名为“myPublicIP”的标准公共 IP 地址。
* 在“myResourceGroupLB”中。

```azurepowershell
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'chinaeast2'
$pubIP = 'myPublicIP'
$sku = 'Standard'
$all = 'static'

$publicIp = 
New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $all -SKU $sku
```

## <a name="create-standard-load-balancer"></a>创建标准负载均衡器

本部分详细介绍如何创建和配置负载均衡器的以下组件：

  * 前端 IP 池，用于在负载均衡器上接收传入网络流量。
  * 后端 IP 池，前端池将负载均衡的网络流量发送到此处。
  * 运行状况探测，用于确定后端 VM 实例的运行状况。
  * 负载均衡器规则，用于定义如何将流量分配给 VM。

### <a name="create-frontend-ip"></a>创建前端 IP

使用 [New-AzLoadBalancerFrontendIpConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerfrontendipconfig) 创建一个前端 IP：

* 命名为“myFrontEnd”。
* 附加到公共 IP“myPublicIP”。

```azurepowershell
## Variables for the commands ##
$fe = 'myFrontEnd'
$rg = 'MyResourceGroupLB'
$loc = 'chinaeast2'
$pubIP = 'myPublicIP'

$publicIp = 
Get-AzPublicIpAddress -Name $pubIP -ResourceGroupName $rg

$feip = 
New-AzLoadBalancerFrontendIpConfig -Name $fe -PublicIpAddress $publicIp
```

### <a name="configure-back-end-address-pool"></a>配置后端地址池

使用 [New-AzLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) 创建一个后端地址池： 

* 命名为“myBackEndPool”。
* 在剩余步骤中，VM 将连接到此后端池。

```azurepowershell
## Variable for the command ##
$be = 'myBackEndPool'

$bepool = 
New-AzLoadBalancerBackendAddressPoolConfig -Name $be
```

### <a name="create-the-health-probe"></a>创建运行状况探测

运行状况探测会检查所有虚拟机实例，以确保它们可以发送网络流量。 

从负载均衡器中删除未通过探测检查的虚拟机。 解决故障后，虚拟机将重新添加到负载均衡器中。

使用 [Add-AzLoadBalancerProbeConfig](https://docs.microsoft.com/powershell/module/az.network/add-azloadbalancerprobeconfig) 创建运行状况探测：

* 监视虚拟机的运行状况。
* 命名为“myHealthProbe”。
* 协议为“TCP”。
* 监视“端口 80”。

```azurepowershell
## Variables for the command ##
$hp = 'myHealthProbe'
$pro = 'http'
$port = '80'
$int = '360'
$cnt = '5'

$probe = 
New-AzLoadBalancerProbeConfig -Name $hp -Protocol $pro -Port $port -RequestPath / -IntervalInSeconds $int -ProbeCount $cnt
```

### <a name="create-the-load-balancer-rule"></a>创建负载均衡器规则

负载均衡器规则定义：

* 针对传入流量的前端 IP 配置。
* 用于接收流量的后端 IP 池。
* 所需的源和目标端口。 

使用 [Add-AzLoadBalancerRuleConfig](https://docs.microsoft.com/powershell/module/az.network/add-azloadbalancerruleconfig) 创建负载均衡器规则： 

* 命名为“myHTTPRule”
* 对前端池“myFrontEnd”中的“端口 80”进行侦听 。
* 使用“端口 80”将负载均衡的网络流量发送到后端地址池“myBackEndPool” 。 
* 使用运行状况探测“myHealthProbe”。
* 协议为“TCP”。

```azurepowershell
## Variables for the command ##
$lbr = 'myHTTPRule'
$pro = 'tcp'
$port = '80'

## $feip and $bePool are the variables from previous steps. ##

$rule = 
New-AzLoadBalancerRuleConfig -Name $lbr -Protocol $pro -Probe $probe -FrontendPort $port -BackendPort $port -FrontendIpConfiguration $feip -BackendAddressPool $bePool -DisableOutboundSNAT
```

### <a name="create-load-balancer-resource"></a>创建负载均衡器资源

使用 [New-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancer) 创建公共负载均衡器：

* 命名为“myLoadBalancer”
* 在 **chinaeast2** 中。
* 在资源组“myResourceGroupLB”中。

```azurepowershell
## Variables for the command ##
$lbn = 'myLoadBalancer'
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$sku = 'Standard'

## $feip, $bepool, $probe, $rule are variables with configuration information from previous steps. ##

$lb = 
New-AzLoadBalancer -ResourceGroupName $rg -Name $lbn -SKU $sku -Location $loc -FrontendIpConfiguration $feip -BackendAddressPool $bepool -Probe $probe -LoadBalancingRule $rule
```

## <a name="configure-virtual-network"></a>配置虚拟网络

需要先创建支持的虚拟网络资源，然后才能部署 VM 和测试负载均衡器。

### <a name="create-a-virtual-network"></a>创建虚拟网络

使用 [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork) 创建虚拟网络：

* 命名为“myVNet”。
* 在资源组“myResourceGroupLB”中。
* 子网命名为“myBackendSubnet”。
* 虚拟网络为“10.0.0.0/16”。
* 子网为“10.0.0.0/24”。

```azurepowershell
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$sub = 'myBackendSubnet'
$spfx = '10.0.0.0/24'
$vnm = 'myVNet'
$vpfx = '10.0.0.0/16'


## Create backend subnet config ##
$subnetConfig = 
New-AzVirtualNetworkSubnetConfig -Name $sub -AddressPrefix $spfx


## Create the virtual network ##
$vnet = 
New-AzVirtualNetwork -ResourceGroupName $rg -Location $loc -Name $vnm -AddressPrefix $vpfx -Subnet $subnetConfig
```
### <a name="create-network-security-group"></a>创建网络安全组
创建网络安全组，以定义虚拟网络的入站连接。

#### <a name="create-a-network-security-group-rule-for-port-80"></a>为端口 80 创建网络安全组规则
使用 [New-AzNetworkSecurityRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecurityruleconfig) 创建网络安全组规则：

* 命名为“myNSGRuleHTTP”。
* 说明为“允许 HTTP”。
* 访问权限为“允许”。
* 协议为“TCP”。
* 方向为“入站”。
* 优先级为“2000”。
* 源为 Internet。
* 源端口范围为“(*)”。
* 目标地址前缀为“(*)”。
* 目标为“端口 80”。

```azurepowershell
## Variables for command ##
$rnm = 'myNSGRuleHTTP'
$des = 'Allow HTTP'
$acc = 'Allow'
$pro = 'Tcp'
$dir = 'Inbound'
$pri = '2000'
$spfx = 'Internet'
$spr = '*'
$dpfx = '*'
$dpr = '80'

$rule1 = 
New-AzNetworkSecurityRuleConfig -Name $rnm -Description $des -Access $acc -Protocol $pro -Direction $dir -Priority $pri -SourceAddressPrefix $spfx -SourcePortRange $spr -DestinationAddressPrefix $dpfx -DestinationPortRange $dpr
```

#### <a name="create-a-network-security-group"></a>创建网络安全组

使用 [New-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecuritygroup) 创建网络安全组：

* 命名为“myNSG”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置中。
* 安全规则在之前的步骤中创建，存储在变量中。

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$nmn = 'myNSG'

## $rule1 contains configuration information from the previous steps. ##
$nsg = 
New-AzNetworkSecurityGroup -ResourceGroupName $rg -Location $loc -Name $nmn -SecurityRules $rule1
```

### <a name="create-network-interfaces"></a>创建网络接口

使用 [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) 创建三个网络接口：

#### <a name="vm-1"></a>VM 1

* 命名为“myNicVM1”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置中。
* 在虚拟网络“myVNet”中。
* 在子网“myBackendSubnet”中。
* 在网络安全组“myNSG”中。
* 附加到 myBackEndPool 中的负载均衡器 myLoadBalancer 。

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$nic1 = 'myNicVM1'
$vnt = 'myVNet'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM1 ##
$nicVM1 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic1 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

#### <a name="vm-2"></a>VM 2

* 命名为“myNicVM2”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置中。
* 在虚拟网络“myVNet”中。
* 在子网“myBackendSubnet”中。
* 在网络安全组“myNSG”中。
* 附加到 myBackEndPool 中的负载均衡器 myLoadBalancer 。

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$nic2 = 'myNicVM2'
$vnt = 'myVNet'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM2 ##
$nicVM2 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic2 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

#### <a name="vm-3"></a>VM 3

* 命名为“myNicVM3”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置中。
* 在虚拟网络“myVNet”中。
* 在子网“myBackendSubnet”中。
* 在网络安全组“myNSG”中。
* 附加到 myBackEndPool 中的负载均衡器 myLoadBalancer 。

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$nic3 = 'myNicVM3'
$vnt = 'myVNet'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM3 ##
$nicVM3 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic3 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```
### <a name="create-virtual-machines"></a>创建虚拟机

使用 [New-AzureRmNetworkInterface](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 设置 VM 的管理员用户名和密码：

```azurepowershell
$cred = Get-Credential
```

使用以下命令创建虚拟机：

* [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm)
* [New-AzVMConfig](https://docs.microsoft.com/powershell/module/az.compute/new-azvmconfig)
* [Set-AzVMOperatingSystem](https://docs.microsoft.com/powershell/module/az.compute/set-azvmoperatingsystem)
* [Set-AzVMSourceImage](https://docs.microsoft.com/powershell/module/az.compute/set-azvmsourceimage)
* [Add-AzVMNetworkInterface](https://docs.microsoft.com/powershell/module/az.compute/add-azvmnetworkinterface)


#### <a name="vm1"></a>VM1

* 命名为“myVM1”。
* 在资源组“myResourceGroupLB”中。
* 附加到网络接口“myNicVM1”。
* 附加到负载均衡器“myLoadBalancer”。
* 在“chinaeast2”位置。

```azurepowershell
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM1'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'chinaeast2'

## Create a virtual machine configuration. $cred and $nicVM1 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM1.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig
```


#### <a name="vm2"></a>VM2

* 命名为“myVM2”。
* 在资源组“myResourceGroupLB”中。
* 附加到网络接口“myNicVM2”。
* 附加到负载均衡器“myLoadBalancer”。
* 在“chinaeast2”位置。

```azurepowershell
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM2'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'chinaeast2'

## Create a virtual machine configuration. $cred and $nicVM2 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM2.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig
```

#### <a name="vm3"></a>VM3

* 命名为“myVM3”。
* 在资源组“myResourceGroupLB”中。
* 附加到网络接口“myNicVM3”。
* 附加到负载均衡器“myLoadBalancer”。
* 在“chinaeast2”位置。

```azurepowershell
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM3'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'chinaeast2'

## Create a virtual machine configuration. $cred and $nicVM3 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM3.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig
```

## <a name="create-outbound-rule-configuration"></a>创建出站规则配置
负载均衡器出站规则为后端池中的 VM 配置出站源网络地址转换 (SNAT)。 

有关出站连接的详细信息，请参阅 [Azure 中的出站连接](load-balancer-outbound-connections.md)。

### <a name="create-outbound-public-ip-address"></a>创建出站公共 IP 地址

使用 [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) 执行以下操作：

* 创建名为“myPublicIPOutbound”的标准公共 IP 地址。
* 在“myResourceGroupLB”中。

```azurepowershell
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'chinaeast2'
$pubIP = 'myPublicIPOutbound'
$sku = 'Standard'
$all = 'static'

$publicIp = 
New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $all -SKU $sku
```

### <a name="create-outbound-frontend-ip-configuration"></a>创建出站前端 IP 配置

使用 [Add-AzLoadBalancerFrontendIpConfig](https://docs.microsoft.com/powershell/module/az.network/add-azloadbalancerfrontendipconfig) 创建新的前端 IP 配置：

* 命名为“myFrontEndOutbound”。
* 与公共 IP 地址 myPublicIPOutbound 关联。

```azurepowershell
## Variables for the command ##
$fen = 'myFrontEndOutbound'

## Get the load balancer configuration  and apply the frontend config##
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg | Add-AzLoadBalancerFrontendIPConfig -Name $fen -PublicIpAddress $publicIP | Set-AzLoadBalancer
```

### <a name="create-outbound-pool"></a>创建出站池

使用 [Add-AzLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/powershell/module/az.network/add-azloadbalancerbackendaddresspoolconfig) 创建新出站池。 

使用 [Set-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/set-azloadbalancer) 将池和前端 IP 地址应用到负载均衡器：

* 命名为“myBackEndPoolOutbound”。

```azurepowershell
## Variables for the command ##
$ben = 'myBackEndPoolOutbound'
$lbn = 'myLoadBalancer'
$rg = 'myResourceGroupLB'

## Get the load balancer configuration and create the outbound backend address pool##
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg | Add-AzLoadBalancerBackendAddressPoolConfig -Name $ben | Set-AzLoadBalancer
```
### <a name="create-outbound-rule-and-apply-to-load-balancer"></a>创建出站规则并应用于负载均衡器

使用 [Add-AzLoadBalancerOutboundRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalanceroutboundruleconfig) 为出站后端池创建新的出站规则。 

使用 [Set-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/set-azloadbalancer) 将规则应用到负载均衡器：

* 命名为“myOutboundRule”。
* 与负载均衡器 myLoadBalancer 关联。
* 与前端 myFrontEndOutbound 关联。
* 协议为“所有”。
* 空闲超时时间为 15。
* 出站端口为 10000 个。
* 与后端池 myBackEndPoolOutbound 关联。
* 在资源组“myResourceGroupLB”中。

```azurepowershell
## Variables for the commands ##
$rg = 'myResourceGroupLB'
$lbn = 'myLoadBalancer'
$brn = 'myOutboundRule'
$pro = 'All'
$idl = '15'
$por = '10000'

## Get the load balancer configuration ##
$lb = 
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg 

## Apply the outbound rule configuration to the load balancer. ##
$lb | Add-AzLoadBalancerOutBoundRuleConfig -Name $brn -FrontendIPConfiguration $lb.FrontendIpConfigurations[1] -BackendAddressPool $lb.BackendAddressPools[1] -Protocol $pro -IdleTimeoutInMinutes $idl -AllocatedOutboundPort $por | Set-AzLoadBalancer
```

### <a name="add-virtual-machines-to-outbound-pool"></a>向出站池添加虚拟机

使用 [Add-AzNetworkInterfaceIpConfig](https://docs.microsoft.com/powershell/module/az.network/add-aznetworkinterfaceipconfig) 将虚拟机网络接口添加到负载均衡器的出站池：


#### <a name="vm1"></a>VM1
* 在后端地址池“myBackEndPoolOutbound”中。
* 在资源组“myResourceGroupLB”中。
* 与网络接口 myNicVM1 和 ipconfig1 关联 。
* 与负载均衡器 myLoadBalancer 关联。

```azurepowershell
## Variables for the commands ##
$rg = 'myResourceGroupLB'
$lbn = 'myLoadBalancer'
$bep = 'myBackEndPoolOutbound'
$nic1 = 'myNicVM1'
$ipc = 'ipconfig1'

## Get the load balancer configuration ##
$lb = 
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg

## Get the network interface configuration ##
$nic = 
Get-AzNetworkInterface -Name $nic1 -ResourceGroupName $rg

## Apply the backend to the network interface ##
$nic | Set-AzNetworkInterfaceIpConfig -Name $ipc -LoadBalancerBackendAddressPoolId $lb.BackendAddressPools[0].id,$lb.BackendAddressPools[1].id | Set-AzNetworkInterface
```

#### <a name="vm2"></a>VM2
* 在后端地址池“myBackEndPoolOutbound”中。
* 在资源组“myResourceGroupLB”中。
* 与网络接口 myNicVM2 和 ipconfig1 关联 。
* 与负载均衡器 myLoadBalancer 关联。

```azurepowershell
## Variables for the commands ##
$rg = 'myResourceGroupLB'
$lbn = 'myLoadBalancer'
$bep = 'myBackEndPoolOutbound'
$nic2 = 'myNicVM2'
$ipc = 'ipconfig1'

## Get the load balancer configuration ##
$lb = 
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg

## Get the network interface configuration ##
$nic = 
Get-AzNetworkInterface -Name $nic2 -ResourceGroupName $rg

## Apply the backend to the network interface ##
$nic | Set-AzNetworkInterfaceIpConfig -Name $ipc -LoadBalancerBackendAddressPoolId $lb.BackendAddressPools[0].id,$lb.BackendAddressPools[1].id | Set-AzNetworkInterface
```

#### <a name="vm3"></a>VM3
* 在后端地址池“myBackEndPoolOutbound”中。
* 在资源组“myResourceGroupLB”中。
* 与网络接口 myNicVM3 和 ipconfig1 关联 。
* 与负载均衡器 myLoadBalancer 关联。

```azurepowershell
## Variables for the commands ##
$rg = 'myResourceGroupLB'
$lbn = 'myLoadBalancer'
$bep = 'myBackEndPoolOutbound'
$nic3 = 'myNicVM3'
$ipc = 'ipconfig1'

## Get the load balancer configuration ##
$lb = 
Get-AzLoadBalancer -Name $lbn -ResourceGroupName $rg

## Get the network interface configuration ##
$nic = 
Get-AzNetworkInterface -Name $nic3 -ResourceGroupName $rg

## Apply the backend to the network interface ##
$nic | Set-AzNetworkInterfaceIpConfig -Name $ipc -LoadBalancerBackendAddressPoolId $lb.BackendAddressPools[0].id,$lb.BackendAddressPools[1].id | Set-AzNetworkInterface

```

# <a name="option-2-create-a-public-load-balancer-basic-sku"></a>[选项 2：创建公共负载均衡器（基本 SKU）](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>对于生产型工作负载，建议使用标准 SKU 负载均衡器。 有关 sku 的详细信息，请参阅 [Azure 负载均衡器 SKU](skus.md)。

## <a name="create-a-public-ip-address"></a>创建公共 IP 地址

若要通过 Internet 访问 Web 应用，需要负载均衡器有一个公共 IP 地址。 

使用 [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) 执行以下操作：

* 创建名为“myPublicIP”的标准公共 IP 地址。
* 在“myResourceGroupLB”中。

```azurepowershell
## Variables for the command ##
$rg = 'MyResourceGroupLB'
$loc = 'chinaeast2'
$pubIP = 'myPublicIP'
$sku = 'Basic'
$all = 'static'

$publicIp = 
New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $all -SKU $sku
```

## <a name="create-basic-load-balancer"></a>创建基本负载均衡器

本部分详细介绍如何创建和配置负载均衡器的以下组件：

  * 前端 IP 池，用于在负载均衡器上接收传入网络流量。
  * 后端 IP 池，前端池将负载均衡的网络流量发送到此处。
  * 运行状况探测，用于确定后端 VM 实例的运行状况。
  * 负载均衡器规则，用于定义如何将流量分配给 VM。

### <a name="create-frontend-ip"></a>创建前端 IP

使用 [New-AzLoadBalancerFrontendIpConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerfrontendipconfig) 创建一个前端 IP：

* 命名为“myFrontEnd”。
* 附加到公共 IP“myPublicIP”。

```azurepowershell
## Variables for the commands ##
$fe = 'myFrontEnd'
$rg = 'MyResourceGroupLB'
$loc = 'chinaeast2'
$pubIP = 'myPublicIP'

$publicIp = 
Get-AzPublicIpAddress -Name $pubIP -ResourceGroupName $rg

$feip = 
New-AzLoadBalancerFrontendIpConfig -Name $fe -PublicIpAddress $publicIp
```

### <a name="configure-back-end-address-pool"></a>配置后端地址池

使用 [New-AzLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) 创建一个后端地址池： 

* 命名为“myBackEndPool”。
* 在剩余步骤中，VM 将连接到此后端池。

```azurepowershell
## Variable for the command ##
$be = 'myBackEndPool'

$bepool = 
New-AzLoadBalancerBackendAddressPoolConfig -Name $be
```

### <a name="create-the-health-probe"></a>创建运行状况探测

运行状况探测会检查所有虚拟机实例，以确保它们可以发送网络流量。 

从负载均衡器中删除未通过探测检查的虚拟机。 解决故障后，虚拟机将重新添加到负载均衡器中。

使用 [Add-AzLoadBalancerProbeConfig](https://docs.microsoft.com/powershell/module/az.network/add-azloadbalancerprobeconfig) 创建运行状况探测：

* 监视虚拟机的运行状况。
* 命名为“myHealthProbe”。
* 协议为“TCP”。
* 监视“端口 80”。

```azurepowershell
## Variables for the command ##
$hp = 'myHealthProbe'
$pro = 'http'
$port = '80'
$int = '360'
$cnt = '5'

$probe = 
New-AzLoadBalancerProbeConfig -Name $hp -Protocol $pro -Port $port -RequestPath / -IntervalInSeconds $int -ProbeCount $cnt
```

### <a name="create-the-load-balancer-rule"></a>创建负载均衡器规则

负载均衡器规则定义：

* 针对传入流量的前端 IP 配置。
* 用于接收流量的后端 IP 池。
* 所需的源和目标端口。 

使用 [Add-AzLoadBalancerRuleConfig](https://docs.microsoft.com/powershell/module/az.network/add-azloadbalancerruleconfig) 创建负载均衡器规则： 

* 命名为“myHTTPRule”
* 对前端池“myFrontEnd”中的“端口 80”进行侦听 。
* 使用“端口 80”将负载均衡的网络流量发送到后端地址池“myBackEndPool” 。 
* 使用运行状况探测“myHealthProbe”。
* 协议为“TCP”。
* 使用前端 IP 地址启用出站源网络地址转换 (SNAT)。


```azurepowershell
## Variables for the command ##
$lbr = 'myHTTPRule'
$pro = 'tcp'
$port = '80'

## $feip and $bePool are the variables from previous steps. ##

$rule = 
New-AzLoadBalancerRuleConfig -Name $lbr -Protocol $pro -Probe $probe -FrontendPort $port -BackendPort $port -FrontendIpConfiguration $feip -BackendAddressPool $bePool
```

### <a name="create-load-balancer-resource"></a>创建负载均衡器资源

使用 [New-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancer) 创建公共负载均衡器：

* 命名为“myLoadBalancer”
* 在 **chinaeast2** 中。
* 在资源组“myResourceGroupLB”中。

```azurepowershell
## Variables for the command ##
$lbn = 'myLoadBalancer'
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$sku = 'Basic'

## $feip, $bepool, $probe, $rule are variables with configuration information from previous steps. ##

$lb = 
New-AzLoadBalancer -ResourceGroupName $rg -Name $lbn -SKU $sku -Location $loc -FrontendIpConfiguration $feip -BackendAddressPool $bepool -Probe $probe -LoadBalancingRule $rule
```

## <a name="configure-virtual-network"></a>配置虚拟网络

需要先创建支持的虚拟网络资源，然后才能部署 VM 和测试负载均衡器。

### <a name="create-a-virtual-network"></a>创建虚拟网络

使用 [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork) 创建虚拟网络：

* 命名为“myVNet”。
* 在资源组“myResourceGroupLB”中。
* 子网命名为“myBackendSubnet”。
* 虚拟网络为“10.0.0.0/16”。
* 子网为“10.0.0.0/24”。

```azurepowershell
## Variables for the command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$sub = 'myBackendSubnet'
$spfx = '10.0.0.0/24'
$vnm = 'myVNet'
$vpfx = '10.0.0.0/16'


## Create backend subnet config ##
$subnetConfig = 
New-AzVirtualNetworkSubnetConfig -Name $sub -AddressPrefix $spfx


## Create the virtual network ##
$vnet = 
New-AzVirtualNetwork -ResourceGroupName $rg -Location $loc -Name $vnm -AddressPrefix $vpfx -Subnet $subnetConfig
```

### <a name="create-network-security-group"></a>创建网络安全组
创建网络安全组，以定义虚拟网络的入站连接。

#### <a name="create-a-network-security-group-rule-for-port-80"></a>为端口 80 创建网络安全组规则
使用 [New-AzNetworkSecurityRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecurityruleconfig) 创建网络安全组规则：

* 命名为“myNSGRuleHTTP”。
* 说明为“允许 HTTP”。
* 访问权限为“允许”。
* 协议为“TCP”。
* 方向为“入站”。
* 优先级为“2000”。
* 源为 Internet。
* 源端口范围为“(*)”。
* 目标地址前缀为“(*)”。
* 目标为“端口 80”。

```azurepowershell
## Variables for command ##
$rnm = 'myNSGRuleHTTP'
$des = 'Allow HTTP'
$acc = 'Allow'
$pro = 'Tcp'
$dir = 'Inbound'
$pri = '2000'
$spfx = 'Internet'
$spr = '*'
$dpfx = '*'
$dpr = '80'

$rule2 = 
New-AzNetworkSecurityRuleConfig -Name $rnm -Description $des -Access $acc -Protocol $pro -Direction $dir -Priority $pri -SourceAddressPrefix $spfx -SourcePortRange $spr -DestinationAddressPrefix $dpfx -DestinationPortRange $dpr
```

#### <a name="create-a-network-security-group"></a>创建网络安全组

使用 [New-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecuritygroup) 创建网络安全组：

* 命名为“myNSG”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置中。
* 安全规则在之前的步骤中创建，存储在变量中。

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$nmn = 'myNSG'

## $rule1 and $rule2 are variables with configuration information from the previous steps. ##
$nsg = 
New-AzNetworkSecurityGroup -ResourceGroupName $rg -Location $loc -Name $nmn -SecurityRules $rule1,$rule2
```

### <a name="create-network-interfaces"></a>创建网络接口

使用 [New-AzNetworkInterface](https://docs.microsoft.com/powershell/module/az.network/new-aznetworkinterface) 创建三个网络接口：

#### <a name="vm-1"></a>VM 1

* 命名为“myNicVM1”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置中。
* 在虚拟网络“myVNet”中。
* 在子网“myBackendSubnet”中。
* 在网络安全组“myNSG”中。
* 附加到 myBackEndPool 中的负载均衡器 myLoadBalancer 。

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$nic1 = 'myNicVM1'
$vnt = 'myVNet'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM1 ##
$nicVM1 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic1 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

#### <a name="vm-2"></a>VM 2

* 命名为“myNicVM2”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置中。
* 在虚拟网络“myVNet”中。
* 在子网“myBackendSubnet”中。
* 在网络安全组“myNSG”中。
* 附加到 myBackEndPool 中的负载均衡器 myLoadBalancer 。

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$nic2 = 'myNicVM2'
$vnt = 'myVNet'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM2 ##
$nicVM2 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic2 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

#### <a name="vm-3"></a>VM 3

* 命名为“myNicVM3”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置中。
* 在虚拟网络“myVNet”中。
* 在子网“myBackendSubnet”中。
* 在网络安全组“myNSG”中。
* 附加到 myBackEndPool 中的负载均衡器 myLoadBalancer 。

```azurepowershell
## Variables for command ##
$rg = 'myResourceGroupLB'
$loc = 'chinaeast2'
$nic3 = 'myNicVM3'
$vnt = 'myVNet'
$lb = 'myLoadBalancer'
$ngn = 'myNSG'

## Command to get virtual network configuration. ##
$vnet = 
Get-AzVirtualNetwork -Name $vnt -ResourceGroupName $rg

## Command to get load balancer configuration
$bepool = 
Get-AzLoadBalancer -Name $lb -ResourceGroupName $rg | Get-AzLoadBalancerBackendAddressPoolConfig

## Command to get network security group configuration ##
$nsg = 
Get-AzNetworkSecurityGroup -Name $ngn -ResourceGroupName $rg

## Command to create network interface for VM3 ##
$nicVM3 = 
New-AzNetworkInterface -ResourceGroupName $rg -Location $loc -Name $nic3 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg -Subnet $vnet.Subnets[0]
```

### <a name="create-availability-set-for-virtual-machines"></a>创建虚拟机的可用性集

使用 [New-AzAvailabilitySet](https://docs.microsoft.com/powershell/module/az.compute/new-azvm) 创建虚拟机的可用性集：

* 命名为“myAvSet”。
* 在资源组“myResourceGroupLB”中。
* 在“chinaeast2”位置。

```azurepowershell
## Variables used for the command. ##
$rg = 'myResourceGroupLB'
$avs = 'myAvSet'
$loc = 'chinaeast2'

New-AzAvailabilitySet -ResourceGroupName $rg -Name $avs -Location $loc
```

### <a name="create-virtual-machines"></a>创建虚拟机

使用 [New-AzureRmNetworkInterface](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 设置 VM 的管理员用户名和密码：

```azurepowershell
$cred = Get-Credential
```

使用以下命令创建虚拟机：

* [New-AzVM](https://docs.microsoft.com/powershell/module/az.compute/new-azvm)
* [New-AzVMConfig](https://docs.microsoft.com/powershell/module/az.compute/new-azvmconfig)
* [Set-AzVMOperatingSystem](https://docs.microsoft.com/powershell/module/az.compute/set-azvmoperatingsystem)
* [Set-AzVMSourceImage](https://docs.microsoft.com/powershell/module/az.compute/set-azvmsourceimage)
* [Add-AzVMNetworkInterface](https://docs.microsoft.com/powershell/module/az.compute/add-azvmnetworkinterface)


#### <a name="vm1"></a>VM1

* 命名为“myVM1”。
* 在资源组“myResourceGroupLB”中。
* 附加到网络接口“myNicVM1”。
* 附加到负载均衡器“myLoadBalancer”。
* 在“chinaeast2”位置。
* 在“myAvSet”可用性集中。

```azurepowershell
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM1'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'chinaeast2'
$avs = 'myAvSet'

## Create a virtual machine configuration. $cred and $nicVM1 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM1.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig -AvailabilitySetName $avs
```


#### <a name="vm2"></a>VM2

* 命名为“myVM2”。
* 在资源组“myResourceGroupLB”中。
* 附加到网络接口“myNicVM2”。
* 附加到负载均衡器“myLoadBalancer”。
* 在“chinaeast2”位置。
* 在“myAvSet”可用性集中。

```azurepowershell
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM2'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'chinaeast2'
$avs = 'myAvSet'

## Create a virtual machine configuration. $cred and $nicVM2 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM2.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig -AvailabilitySetName $avs
```

#### <a name="vm3"></a>VM3

* 命名为“myVM3”。
* 在资源组“myResourceGroupLB”中。
* 附加到网络接口“myNicVM3”。
* 附加到负载均衡器“myLoadBalancer”。
* 在“chinaeast2”位置。
* 在“myAvSet”可用性集中。

```azurepowershell
## Variables used for command. ##
$rg = 'myResourceGroupLB'
$vm = 'myVM3'
$siz = 'Standard_DS1_v2'
$pub = 'MicrosoftWindowsServer'
$off = 'WindowsServer'
$sku = '2019-Datacenter'
$ver = 'latest'
$loc = 'chinaeast2'
$avs = 'myAvSet'

## Create a virtual machine configuration. $cred and $nicVM3 are variables with configuration from the previous steps. ##

$vmConfig = 
New-AzVMConfig -VMName $vm -VMSize $siz | Set-AzVMOperatingSystem -Windows -ComputerName $vm -Credential $cred | Set-AzVMSourceImage -PublisherName $pub -Offer WindowsServer -Skus $sku -Version $ver | Add-AzVMNetworkInterface -Id $nicVM3.Id

## Create the virtual machine ##
New-AzVM -ResourceGroupName $rg -Location $loc -VM $vmConfig -AvailabilitySetName $avs
```

创建和配置这三个 VM 需要几分钟的时间来完成。

---

## <a name="install-iis"></a>安装 IIS

1. [登录](https://portal.azure.cn) Azure 门户。

2. 在左侧菜单中选择“所有服务”，选择“所有资源”，然后在资源列表中选择位于“myResourceGroupLB”资源组中的“myVM1”。

3. 在“概览”页上，选择“连接”。********

4. 输入在 VM 创建过程中输入的用户名和密码。

5. 选择“连接”。

6. 在服务器桌面上，导航到“Windows 管理工具” > “Windows PowerShell”。

7. 在 PowerShell 窗口中，运行以下命令以：

    * 安装 IIS 服务器
    * 删除默认的 iisstart.htm 文件
    * 添加显示 VM 名称的新 iisstart.htm 文件：

   ```powershell
    
    # install IIS server role
     Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # remove default htm file
     Remove-Item  C:\inetpub\wwwroot\iisstart.htm
    
    # Add a new htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
   ```
8. 关闭 RDP 连接。

9. 重复步骤 1 到步骤 8，在 myVM2 和 myVM3 上安装 IIS 和更新后的 iisstart.htm 文件 。

## <a name="test-the-load-balancer"></a>测试负载均衡器

1. 在 Azure 门户中，在“概览”屏幕上找到负载均衡器的公共 IP 地址。 在左侧菜单中选择“所有服务”，选择“所有资源”，然后选择“myPublicIP”。

2. 复制该公共 IP 地址，并将其粘贴到浏览器的地址栏。 IIS Web 服务器的默认页会显示在浏览器上。

   ![IIS Web 服务器](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)

要查看所有三个 VM 中的负载均衡器分配流量，可以自定义每个 VM 的 IIS Web 服务器的默认页面，然后从客户端计算机强制刷新 Web 浏览器。


## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、负载均衡器和剩余资源，可以使用 [Remove-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/remove-azresourcegroup) 命令将其删除。

```azurepowershell
## Variable for command. ##
$rg = 'myResourceGroupLB'

Remove-AzResourceGroup -Name $rg
```

## <a name="next-steps"></a>后续步骤

在本快速入门中

* 你创建了一个标准或基本公共负载均衡器
* 附加了虚拟机。 
* 配置了负载均衡器流量规则和运行状况探测。
* 测试了负载均衡器。

若要了解有关 Azure 负载均衡器的更多信息，请进一步阅读[什么是 Azure 负载均衡器？](load-balancer-overview.md)和[负载均衡器常见问题](load-balancer-faqs.md)。
