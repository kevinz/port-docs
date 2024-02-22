---

sidebar_position: 6
description: URL 是用于保存网站链接的输入法

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# URL

URL 是用于保存网站链接的输入法。

## 💡 常用网址 Usage

例如，URL 输入类型可被用来存储指向任何网络资源的链接: 

* Datadog 仪表板链接；
* 配置文件链接；
* 链接到拉取请求；
* 等等。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Select (Enum)", value: "enum"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myUrlInput": {
    "title": "My url input",
    "icon": "My icon",
    "description": "My url input",
    // highlight-start
    "type": "string",
    "format": "url",
    // highlight-end
    "default": "https://example.com"
  }
}
```

</TabItem>
<TabItem value="enum">

```json showLineNumbers
{
  "myUrlSelectInput": {
    "title": "My url select input",
    "icon": "My icon",
    "description": "My url select input",
    "type": "string",
    "format": "url",
    // highlight-next-line
    "enum": ["https://example.com", "https://getport.io"]
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myUrlArrayInput": {
    "title": "My url array input",
    "icon": "My icon",
    "description": "My url array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "format": "url"
    }
    // highlight-end
  }
}
```

</TabItem>
</Tabs>

<ApiRef />

## Terraform 定义

<Tabs groupId="tf-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Select (Enum)", value: "enum"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    string_props = {
      "myUrlInput" = {
        title       = "My url input"
        icon        = "My icon"
        description = "My url input"
        format      = "url"
        default     = "https://example.com"
      }
    }
  }
  # highlight-end
}
```

</TabItem>

<TabItem value="enum">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    string_props = {
      "myUrlSelectInput" = {
        title       = "My url select input"
        icon        = "My icon"
        description = "My url select input"
        format      = "url"
        enum        = ["https://example.com", "https://getport.io"]
      }
    }
  }
  # highlight-end
}
```

</TabItem>

<TabItem value="array">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    array_props = {
      "myUrlArrayInput" = {
        title       = "My url array input"
        icon        = "My icon"
        description = "My url array input"
        format      = "url"
      }
    }
  }
  # highlight-end
}
```

</TabItem>
</Tabs>