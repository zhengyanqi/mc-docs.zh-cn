---
author: rockboyfor
ms.service: virtual-machines-linux
ms.topic: include
origin.date: 10/26/2018
ms.date: 05/18/2020
ms.author: v-yeche
ms.openlocfilehash: 7ac202576866fe5c4470f17888a6f164ed2fa313
ms.sourcegitcommit: 8d56bc6baeb42d675695ecef1909d76f5c4a6ae3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/14/2020
ms.locfileid: "83406221"
---
将数据磁盘添加到 Linux VM 时，如果 LUN 0 位置没有磁盘，则你可能会遇到错误。 如果使用 `az vm disk attach -new` 命令并指定 LUN (`--lun`) 来手动添加磁盘，而不是让 Azure 平台确定适当的 LUN，则请注意，LUN 0 已经有磁盘或者将有磁盘。 

请考虑以下示例，其中显示了 `lsscsi` 输出的代码片段：

```bash
[5:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc 
[5:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdd 
```

两个数据磁盘位于 LUN 0 和 LUN 1（`lsscsi` 中的第一列输出了详细信息 `[host:channel:target:lun]`）。 两个磁盘都应该可从 VM 内部访问。 如果手动指定了要在 LUN 1 位置添加第一个磁盘并在 LUN 2 位置添加第二个磁盘，则可能无法从 VM 内部正常查看这些磁盘。

> [!NOTE]
> 在这些示例中，Azure `host` 值为 5，但此值可能根据所选存储类型的不同而异。
> 
> 

此磁盘行为不是 Azure 的问题，而是因为 Linux 内核遵循了 SCSI 规范。 当 Linux 内核在 SCSI 总线中扫描附加的设备时，必须能够在 LUN 0 位置找到设备，系统才能继续扫描是否有其他设备。 因此：

* 在添加数据磁盘之后，请查看 `lsscsi` 的输出，验证 LUN 0 位置是否有磁盘。
* 如果磁盘未在 VM 内正确显示，请验证 LUN 0 位置是否有磁盘。

<!-- Update_Description: update meta properties, wording update, update link -->