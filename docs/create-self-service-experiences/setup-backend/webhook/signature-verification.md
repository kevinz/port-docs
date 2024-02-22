---
sidebar_position: 5
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 验证 Webhook 签名

在本指南中，我们将介绍如何验证 Webhook 收到的消息是否由 Port 发送。

## 签名验证安全效益

当 HTTP 服务器暴露在互联网上时，任何拥有该 URL 的人都可以向其发送网络请求。

对 webhook 的意外触发可能会产生不良影响，例如: ..: 

* 由于大量不必要的触发而导致使用成本过高；
* 执行的操作会让攻击者访问您的环境，或改变您的基础架构或导致停机；
* 以及更多。

被用于请求头验证 webhook 请求有以下好处: 

* 确保请求有效载荷未被篡改
* 确保信息发送方是 Port
* 确保收到的信息不是旧信息的重放

## 自定义标题

每个 webhook 请求都包括以下由 Port 提供的自定义标头: 

* `x-port-timestamp` - 文件中的时间戳[seconds since epoch](https://en.wikipedia.org/wiki/Epoch)
* `x-port-signature` -[Base64](https://en.wikipedia.org/wiki/Base64) 编码的签名

## 签名结构

webhook 签名会引用信息生成的 "时间戳 "和请求的 "支付负载"，并使用点 (`.`)将它们连接起来: 

```js
const signatureContent = `${timestamp}.${payload}`;
```

Port 接收需要签名的内容，使用 `HMAC-SHA-256` 对其进行散列，然后用 `Base64` 进行编码。 然后，Port 接收输出，并将版本作为前缀加上逗号 (`,`)，例如 `x-port-signature` 头应该如下所示: `v1,2ehMaSsW+OTSDFERA/SmIKSSySlE3uaJELVlNIOLJ1OE=`。

## 时间戳验证

如上所述，Port 还会在 `x-port-timestamp` 标头中发送 webhook 请求的时间戳。 为防止重放攻击，您可以将时间戳的值与系统的时间戳进行比较，并验证两者的差异是否在允许的容差范围内。

## 代码示例

下面是一些示例，说明如何使用请求头验证 webhook 请求签名: 

<Tabs groupId="code-examples" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "Javascript", value: "javascript"}
]}>

<TabItem value="python">

```python showLineNumbers
import base64
import hashlib
import hmac
import json

body = await request.body()
port_signature = request.headers.get('x-port-signature')
port_timestamp = request.headers.get('x-port-timestamp')

data = body if isinstance(body, str) else body.decode()

to_sign = f"{port_timestamp}.{data}".encode()
signature = hmac.new(settings.PORT_CLIENT_SECRET.encode(), to_sign, hashlib.sha256).digest()
expected_sig = base64.b64encode(signature).decode()

if expected_sig != port_signature.split(",")[1]:
    raise Exception('Invalid signature')

print('Success!')
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
const { createHmac } = require("crypto");

const portSignature = request.headers["x-port-signature"];
const portTimestamp = request.headers["x-port-timestamp"];
const signed = createHmac("sha256", "<CLIENT_SECRET>")
  .update(`${portTimestamp}.${JSON.stringify(request.body)}`)
  .digest("base64");

if (signed !== portSignature.split(",")[1]) {
  throw new Error("Invalid signature");
}

console.log("Success!");
```

</TabItem>

</Tabs>