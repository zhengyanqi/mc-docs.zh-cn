---
title: azcopy | Microsoft Docs
description: 本文提供有关 azcopy 命令的参考信息。
author: WenJason
ms.service: storage
ms.topic: reference
origin.date: 07/24/2020
ms.date: 08/24/2020
ms.author: v-jay
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: 3bac28a729b8355ed6f88ce10400290038404c22
ms.sourcegitcommit: ecd6bf9cfec695c4e8d47befade8c462b1917cf0
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/23/2020
ms.locfileid: "88753479"
---
# <a name="azcopy"></a>azcopy

AzCopy 是一个命令行工具，用于将数据移入和移出 Azure 存储。

## <a name="synopsis"></a>概要

命令的常规格式为：`azcopy [command] [arguments] --[flag-name]=[flag-value]`。

若要报告问题或详细了解该工具，请参阅 [https://github.com/Azure/azure-storage-azcopy](https://github.com/Azure/azure-storage-azcopy)。

## <a name="related-conceptual-articles"></a>相关概念性文章

- [AzCopy 入门](storage-use-azcopy-v10.md)
- [使用 AzCopy 和 Blob 存储传输数据](storage-use-azcopy-blobs.md)
- [使用 AzCopy 和文件存储传输数据](storage-use-azcopy-files.md)
- [对 AzCopy 进行配置、优化和故障排除](storage-use-azcopy-configure.md)

## <a name="options"></a>选项

**--cap-mbps**（浮动）以兆位/秒为单位限制传输速率。 瞬间吞吐量可能与上限略有不同。 如果此选项设置为零，或者省略，则吞吐量不受限制。

**--help** azcopy 命令的帮助
      
**--output-type**（字符串）命令输出的格式。 选项包括：text、json。 默认值为 `text`。 （默认 `text`）

**--trusted-microsoft-suffixes**（字符串）指定可向其中发送 Azure Active Directory 登录令牌的其他域后缀。  默认值为“.core.windows.net;.core.chinacloudapi.cn;.core.cloudapi.de;.core.usgovcloudapi.net” 。 此处列出的任何内容都会添加到默认值。 为安全起见，应只在此处放置 Azure 域。 用分号分隔多个条目。

## <a name="see-also"></a>另请参阅

- [AzCopy 入门](storage-use-azcopy-v10.md)
- [azcopy bench](storage-ref-azcopy-bench.md)
- [azcopy copy](storage-ref-azcopy-copy.md)
- [azcopy doc](storage-ref-azcopy-doc.md)
- [azcopy env](storage-ref-azcopy-env.md)
- [azcopy jobs](storage-ref-azcopy-jobs.md)
- [azcopy jobs clean](storage-ref-azcopy-jobs-clean.md)
- [azcopy jobs list](storage-ref-azcopy-jobs-list.md)
- [azcopy jobs remove](storage-ref-azcopy-jobs-remove.md)
- [azcopy jobs resume](storage-ref-azcopy-jobs-resume.md)
- [azcopy jobs show](storage-ref-azcopy-jobs-show.md)
- [azcopy list](storage-ref-azcopy-list.md)
- [azcopy login](storage-ref-azcopy-login.md)
- [azcopy logout](storage-ref-azcopy-logout.md)
- [azcopy make](storage-ref-azcopy-make.md)
- [azcopy remove](storage-ref-azcopy-remove.md)
- [azcopy sync](storage-ref-azcopy-sync.md)
