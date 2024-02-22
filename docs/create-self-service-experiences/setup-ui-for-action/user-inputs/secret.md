---

sidebar_position: 14
description: Secret 是一种输入法，其值在发送到后端时会用客户secret加密，在其传输过程中绝不会保存或记录。

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# Secret

secret输入是一种用于向操作后端传递secret和敏感信息的输入类型，通过secret输入发送的值会使用您的私人密钥进行额外的加密。 此外，通过secret输入发送的值不会被 Port 记录或保存。

## 💡 常用secret Usage

secret输入类型可被引用于敏感信息，如: "......": 

* 云secret；
* 密码
* API 密钥
* Secret 令牌
* 等等。

## secret输入结构

secret输入的定义与普通输入相同，但多了一个指定加密算法的 "加密 "字段: 

```json showLineNumbers
{
  "mySecretInput": {
    "title": "My secret input",
    "icon": "My icon",
    "type": "string",
    // highlight-start
    "encryption": "aes256-gcm",
    // highlight-end
    "description": "My entity input"
  }
}
```

* [aes256-gcm](https://www.nist.gov/publications/advanced-encryption-standard-aes) - 这将在[GCM mode](https://csrc.nist.gov/glossary/term/aes_gcm) 中使用 256 位 AES 对属性数据进行加密。加密值前缀为 16 位 IV，后缀为 16 位 MAC，编码为基序 64。加密密钥将是贵组织[Client Secret](../../../build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials) 的前 32 个字节。

#### 支持的类型

<Tabs groupId="supported-types" queryString defaultValue="string" values={[
{label: "String Secret", value: "string"},
{label: "Object Secret", value: "object"}
]}>

<TabItem value="string">

```json showLineNumbers
{
  "mySecretInput": {
    "title": "My secret input",
    "icon": "My icon",
    // highlight-start
    "type": "string",
    // highlight-end
    "encryption": "aes256-gcm",
    "description": "My entity input"
  }
}
```

**注意: ** 不支持为 secrets 输入设置 `format` 格式。

</TabItem>
<TabItem value="object">

```json showLineNumbers
{
  "mySecretInput": {
    "title": "My secret input",
    "icon": "My icon",
    // highlight-start
    "type": "object",
    // highlight-end
    "encryption": "aes256-gcm",
    "description": "My entity input"
  }
}
```

</TabItem>
</Tabs>

## 处理有效载荷

发送到基础架构的有效载荷将包含secret属性输入的加密值。 要使用secret输入，您需要对其进行解密: 

### 示例

<Tabs groupId="algorithm" queryString defaultValue="aes256-gcm" values={[
{label: "AES 256 GCM", value: "aes256-gcm"},
]}>

<TabItem value="aes256-gcm">

解密使用 "aes256-gcm "算法加密的属性的示例。

<Tabs groupId="language" queryString defaultValue="aes256-gcm-python" values={[
{label: "Python Webhook", value: "aes256-gcm-python"},
{label: "NodeJs Webhook", value: "aes256-gcm-nodeJs"}
]}>

<TabItem value="aes256-gcm-python">

下面的示例被引用了 `flask` 和 `pycryptodome` 软件包: 

```python showLineNumbersimport base64
import base64
import json
import os

from flask import Flask, request
from Crypto.Cipher import AES

PORT_CLIENT_SECRET = 'YOUR PORT CLIENT SECRET'
PROPERY_IS_JSON = False # whether the property is defined as json or not (string otherwise)

app = Flask(__name__)

@app.route('/', methods=['POST'])
def webhook():
    # initialize the aes cipher
    key = PORT_CLIENT_SECRET[:32].encode()

    req = request.get_json(silent=True, force=True)
    encrypted_property_value = base64.b64decode(req.get('payload').get('properties').get('secret-property'))

    iv = encrypted_property_value[:16]
    ciphertext = encrypted_property_value[16:-16]
    mac = encrypted_property_value[-16:]

    cipher = AES.new(key, AES.MODE_GCM, iv)

    # decrypt the property
    decrypted_property_value = cipher.decrypt_and_verify(ciphertext, mac)
    property_value = json.loads(decrypted_property_value) if PROPERY_IS_JSON else decrypted_property_value

    return property_value # this is the original value the user sent

if __name__ == '__main__':
    port = int(os.getenv('PORT', 80))

    print("Starting app on port %d" % port)

    app.run(debug=False, port=port, host='0.0.0.0')
```

</TabItem>
<TabItem value="aes256-gcm-nodeJs">

下面的示例被引用了 `express` 软件包和节点内置的加密模块: 

```js showLineNumbers
const express = require("express");
const bodyParser = require("body-parser");
const crypto = require("node:crypto");

const PORT_CLIENT_SECRET = "YOUR PORT CLIENT SECRET";
const PROPERY_IS_JSON = false; // whether the property is defined as json or not (string otherwise)

const port = 80;

const ENCODING = "utf8";
const ALGORITHM_NAME = "aes-256-gcm";
const ALGORITHM_IV_SIZE = 16;
const ALGORITHM_TAG_SIZE = 16;

const app = express();

app.post("/", bodyParser.json(), (req, res) => {
  // deconstruct the property
  const raw_property_value = req.body.payload.properties["secret-property"];
  const property_value_buffer = Buffer.from(raw_property_value, "base64");

  const iv = property_value_buffer.subarray(0, ALGORITHM_IV_SIZE);
  const data = property_value_buffer.subarray(
    ALGORITHM_IV_SIZE,
    property_value_buffer.length - ALGORITHM_TAG_SIZE
  );
  const authTag = property_value_buffer.subarray(
    property_value_buffer.length - ALGORITHM_TAG_SIZE
  );

  // initialize the aes decipher
  const encodedSecret = Buffer.from(PORT_CLIENT_SECRET.substring(0, 32));
  const decipher = crypto.createDecipheriv(ALGORITHM_NAME, encodedSecret, iv);
  decipher.setAuthTag(authTag);

  // decrypt the property
  const decrypted_property_buffer = Buffer.concat([
    decipher.update(data),
    decipher.final(),
  ]);

  // encode the value
  const decrypted_property_value = decrypted_property_buffer.toString(ENCODING);
  const property_value = PROPERY_IS_JSON
    ? JSON.parse(decrypted_property_value)
    : decrypted_property_value;

  return property_value; // this is the original value the user sent
});

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`);
});
```

</TabItem>
</Tabs>

</TabItem>
</Tabs>