---
title: dayofmonth() - Azure 数据资源管理器 | Microsoft Docs
description: 本文介绍 Azure 数据资源管理器中的 dayofmonth()。
services: data-explorer
author: orspod
ms.author: v-tawe
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
origin.date: 02/13/2020
ms.date: 08/06/2020
ms.openlocfilehash: 1084778f0f8e70fca4dcea46160cfc27683ff618
ms.sourcegitcommit: 7ceeca89c0f0057610d998b64c000a2bb0a57285
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/06/2020
ms.locfileid: "87841597"
---
# <a name="dayofmonth"></a>dayofmonth()

返回一个整数，该整数表示给定月的第几天

```kusto
dayofmonth(datetime(2015-12-14)) == 14
```

**语法**

`dayofmonth(`*a_date*`)`

**参数**

* `a_date`：`datetime`。

**返回**

给定月份的 `day number`。