---
title: series_greater() - Azure 数据资源管理器
description: 本文介绍 Azure 数据资源管理器中的 series_greater()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 04/01/2020
ms.date: 08/06/2020
ms.openlocfilehash: cbdb624d5339ca6b3710e5e8f01922cda449d9f6
ms.sourcegitcommit: 7ceeca89c0f0057610d998b64c000a2bb0a57285
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2020
ms.locfileid: "87841240"
---
# <a name="series_greater"></a>series_greater()

计算两个数值序列输入的元素对应大于 (`>`) 逻辑运算。

**语法**

`series_greater (`*Series1*`,` *Series2*`)`

**参数**

* Series1、Series2：要按元素比较的输入数值数组。 所有参数都必须是动态数组。 

**返回**

布尔值的动态数组，包括在两个输入之间计算的元素对应大于逻辑运算。 任何非数值元素或非现有元素（不同大小的数组）都会生成 `null` 元素值。

**示例**

<!-- csl: https://help.kusto.chinacloudapi.cn:443/Samples -->
```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_greater_s2 = series_greater(s1, s2)
```

|s1|s2|s1_greater_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[false,false,true]|

**另请参阅**

若要进行整个序列统计信息的比较，请参阅：
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
