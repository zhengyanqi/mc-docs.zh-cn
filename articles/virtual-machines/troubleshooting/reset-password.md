---
title: 如何在 Azure VM 上重置本地 Linux 密码 | Azure
description: 在 Azure VM 上重置本地 Linux 密码的步骤简介
services: virtual-machines-linux
documentationcenter: ''
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
origin.date: 08/20/2019
author: rockboyfor
ms.date: 09/07/2020
ms.testscope: yes
ms.testdate: 08/31/2020
ms.author: v-yeche
ms.openlocfilehash: dee623db57daf9cad4a3c7a9595ca33ec61a5358
ms.sourcegitcommit: 42d0775781f419490ceadb9f00fb041987b6b16d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/04/2020
ms.locfileid: "89456859"
---
# <a name="how-to-reset-local-linux-password-on-azure-vms"></a>如何在 Azure VM 上重置本地 Linux 密码

本文介绍了几种重置本地 Linux 虚拟机 (VM) 密码的方法。 如果用户帐户已过期或者只是想要创建新帐户，则可以使用以下方法来创建新的本地管理员帐户并重新获得对 VM 的访问权限。

## <a name="symptoms"></a>症状

无法登录到 VM 时会收到一条消息，指示所使用的密码不正确。 此外，无法在 Azure 门户上使用 VMAgent 重置密码。

## <a name="manual-password-reset-procedure"></a>手动密码重置过程

> [!NOTE]
> 以下步骤不适用于包含非托管磁盘的 VM。

1. 为受影响的 VM 的 OS 磁盘拍摄快照，从快照创建磁盘，然后将该磁盘附加到故障排除 VM。 有关详细信息，请参阅[通过使用 Azure 门户将 OS 磁盘附加到恢复 VM 来对 Windows VM 进行故障排除](troubleshoot-recovery-disks-portal-linux.md)。

2. 使用远程桌面连接到故障排除 VM。

3. 在故障排除 VM 上运行以下 SSH 命令，成为超级用户。

    ```bash
    sudo su
    ```

4. 运行 fdisk -l  ，或查看系统日志以查找新附加的磁盘。 找到要装载的驱动器名称。 然后在临时 VM 上，查找相关的日志文件。

    ```bash
    grep SCSI /var/log/kern.log (ubuntu)
    grep SCSI /var/log/messages (centos, suse, oracle)
    ```

    下面是 grep 命令的输出示例：

    ```bash
    kernel: [ 9707.100572] sd 3:0:0:0: [sdc] Attached SCSI disk
    ```

5. 创建名为 tempmount  的装入点。

    ```bash
    mkdir /tempmount
    ```

6. 在该装入点上装载 OS 磁盘。 通常需要装载 sdc1  或 sdc2  。 这将取决于损坏的计算机磁盘的 /etc  目录中的托管分区。

    ```bash
    mount /dev/sdc1 /tempmount
    ```

7. 在进行任何更改之前，请创建核心凭据文件的副本：

    ```bash
    cp /etc/passwd /etc/passwd_orig    
    cp /etc/shadow /etc/shadow_orig    
    cp /tempmount/etc/passwd /etc/passwd
    cp /tempmount/etc/shadow /etc/shadow 
    cp /tempmount/etc/passwd /tempmount/etc/passwd_orig
    cp /tempmount/etc/shadow /tempmount/etc/shadow_orig
    ```

8. 重置所需的用户密码：

    ```bash
    passwd <<USER>> 
    ```

9. 将已修改的文件移动到断开的计算机磁盘上的正确位置。

    ```bash
    cp /etc/passwd /tempmount/etc/passwd
    cp /etc/shadow /tempmount/etc/shadow
    cp /etc/passwd_orig /etc/passwd
    cp /etc/shadow_orig /etc/shadow
    ```

10. 返回到根目录并卸载磁盘。

    ```bash
    cd /
    umount /tempmount
    ```

11. 在 Azure 门户中，从故障排除 VM 分离该磁盘。

12. [更改受影响 VM 的 OS 磁盘](troubleshoot-recovery-disks-portal-linux.md#swap-the-os-disk-for-the-vm)。

## <a name="next-steps"></a>后续步骤

* [Troubleshoot Azure VM by attaching OS disk to another Azure VM](https://social.technet.microsoft.com/wiki/contents/articles/18710.troubleshoot-azure-vm-by-attaching-os-disk-to-another-azure-vm.aspx)（通过将 OS 磁盘附加到另一个 Azure VM 对 Azure VM 进行故障排除）

* [Azure CLI: How to delete and re-deploy a VM from VHD](https://docs.microsoft.com/archive/blogs/linuxonazure/azure-cli-how-to-delete-and-re-deploy-a-vm-from-vhd)（Azure CLI：如何从 VHD 删除和重新部署 VM）

<!-- Update_Description: update meta properties, wording update, update link -->