---
title: 对将事件发布到事件网格自定义主题或域的客户端进行身份验证
description: 本文介绍对将事件发布到事件网格自定义主题的客户端进行身份验证的不同方法。
ms.topic: conceptual
author: Johnnytechn
ms.author: v-johya
ms.date: 08/10/2020
ms.openlocfilehash: be62da7f92f32e6a565ba047218b1251c05b0073
ms.sourcegitcommit: 9d9795f8a5b50cd5ccc19d3a2773817836446912
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/14/2020
ms.locfileid: "88228939"
---
# <a name="authenticate-publishing-clients-azure-event-grid"></a>对发布客户端进行身份验证（Azure 事件网格）
本文提供有关使用访问密钥或共享访问签名 (SAS) 令牌对将事件发布到 Azure 事件网格主题或域的客户端进行身份验证的信息。 我们建议使用 SAS 令牌，但密钥身份验证提供简单的编程，并与多个现有 Webhook 发布服务器兼容。  

## <a name="authenticate-using-an-access-key"></a>使用访问密钥进行身份验证
访问密钥身份验证是最简单的身份验证形式。 可以将访问密钥作为 HTTP 标头或 URL 查询参数传递。 

### <a name="access-key-in-a-http-header"></a>HTTP 标头中的访问密钥
将访问密钥作为 HTTP 标头的值传递：`aeg-sas-key`。

```
aeg-sas-key: XXXXXXXXXXXXXXXXXX0GXXX/nDT4hgdEj9DpBeRr38arnnm5OFg==
```

### <a name="access-key-as-a-query-parameter"></a>用作查询参数的访问密钥
还可以将 `aeg-sas-key` 指定为查询参数。 

```
https://<yourtopic>.<region>.eventgrid.azure.cn/eventGrid/api/events?api-version=2018-01-01&&aeg-sas-key=XXXXXXXX53249XX8XXXXX0GXXX/nDT4hgdEj9DpBeRr38arnnm5OFg==
```

有关如何获取主题或域的访问密钥的说明，请参阅[获取访问密钥](get-access-keys.md)。

## <a name="authenticate-using-a-sas-token"></a>使用 SAS 令牌进行身份验证
事件网格资源的 SAS 令牌包括资源、过期时间和签名。 SAS 令牌的格式是：`r={resource}&e={expiration}&s={signature}`。

资源是要将事件发送到的事件网格主题的路径。 例如，有效的资源路径是 `https://<yourtopic>.<region>.eventgrid.azure.cn/eventGrid/api/events?api-version=2018-01-01`。 若要查看所有受支持的 API 版本，请参阅 [Microsoft.EventGrid 资源类型](https://docs.microsoft.com/azure/templates/microsoft.eventgrid/allversions)。 
<!--Correct in MC: https://<yourtopic>.<region>.eventgrid.azure.cn/eventGrid/api/events?api-version=2018-01-01-->
<!--Not available in MC: api-version=2019-06-01-->

首先，以编程方式生成 SAS 令牌，然后使用 `aeg-sas-token` 标头或 `Authorization SharedAccessSignature` 标头向事件网格进行身份验证。 

### <a name="generate-sas-token-programmatically"></a>以编程方式生成 SAS 令牌
以下示例会创建用于事件网格的 SAS 令牌：

```cs
static string BuildSharedAccessSignature(string resource, DateTime expirationUtc, string key)
{
    const char Resource = 'r';
    const char Expiration = 'e';
    const char Signature = 's';

    string encodedResource = HttpUtility.UrlEncode(resource);
    var culture = CultureInfo.CreateSpecificCulture("en-US");
    var encodedExpirationUtc = HttpUtility.UrlEncode(expirationUtc.ToString(culture));

    string unsignedSas = $"{Resource}={encodedResource}&{Expiration}={encodedExpirationUtc}";
    using (var hmac = new HMACSHA256(Convert.FromBase64String(key)))
    {
        string signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(unsignedSas)));
        string encodedSignature = HttpUtility.UrlEncode(signature);
        string signedSas = $"{unsignedSas}&{Signature}={encodedSignature}";

        return signedSas;
    }
}
```

### <a name="using-aeg-sas-token-header"></a>使用 aeg-sas-token 标头
以下是将 SAS 令牌作为 `aeg-sas-toke` 标头的值传递的示例。 

```http
aeg-sas-token: r=https%3a%2f%2fmytopic.eventgrid.azure.cn%2feventGrid%2fapi%2fevent&e=6%2f15%2f2017+6%3a20%3a15+PM&s=XXXXXXXXXXXXX%2fBPjdDLOrc6THPy3tDcGHw1zP4OajQ%3d
```

### <a name="using-authorization-header"></a>使用 Authorization 标头
以下是将 SAS 令牌作为 `Authorization` 标头的值传递的示例。 

```http
Authorization: SharedAccessSignature r=https%3a%2f%2fmytopic.eventgrid.azure.cn%2feventGrid%2fapi%2fevent&e=6%2f15%2f2017+6%3a20%3a15+PM&s=XXXXXXXXXXXXX%2fBPjdDLOrc6THPy3tDcGHw1zP4OajQ%3d
```

## <a name="next-steps"></a>后续步骤
请参阅[事件传递身份验证](security-authentication.md)，了解如何使用事件处理程序进行身份验证以传递事件。 

