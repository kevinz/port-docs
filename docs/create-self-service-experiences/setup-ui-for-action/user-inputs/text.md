---
sidebar_position: 1
description: 文本文本是文本信息的基本输入

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 文本

文字是文本信息的基本输入方式。

## 💡 常用文本 Usage

文本输入类型可被用于来存储任何基于文本的数据，例如: 

* 镜像标签；
* 可变密钥
* 提交 SHA；
* 文件名；
* 等等。

在[live demo](https://demo.getport.io/self-serve) 自助中心页面中，我们可以看到**新服务脚手架**操作，其 "服务名称 "输入为文本输入。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Select (Enum)", value: "enum"},
{label: "Array", value: "array"},
{label: "Enum Array", value: "enumArray"},
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myTextInput": {
    "title": "My text input",
    "icon": "My icon",
    "description": "My text input",
    // highlight-start
    "type": "string",
    // highlight-end
    "default": "My default"
  }
}
```

</TabItem>
<TabItem value="enum">

```json showLineNumbers
{
  "myTextSelectInput": {
    "title": "My text select input",
    "icon": "My icon",
    "description": "My text select input",
    "type": "string",
    // highlight-next-line
    "enum": ["my-option-1", "my-option-2"]
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myTextArrayInput": {
    "title": "My text array input",
    "icon": "My icon",
    "description": "My text array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string"
    }
    // highlight-end
  }
}
```

</TabItem>
<TabItem value="enumArray">

```json showLineNumbers
{
  "myStringArray": {
    "title": "My text-selection array input",
    "icon": "My icon",
    "description": "My text-selection array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "enum": ["my-option-1", "my-option-2"],
      "enumColors": {
        "my-option-1": "red",
        "my-option-2": "green"
      }
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
      myTextInput = {
        title       = "My text input"
        description = "My text input"
        default     = "My default"
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
      myTextSelectInput = {
        title       = "My text select input"
        description = "My text select input"
        enum        = ["my-option-1", "my-option-2"]
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
      myTextArrayInput = {
        title       = "My text array input"
        description = "My text array input"
        string_items = {}
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>

## 验证文本

文本验证支持以下操作符: 

* `minLength` - 执行最小字符串长度；
* `maxLength` - 执行最大字符串长度；
* `pattern` - 执行 Regex 模式。

<Tabs groupId="validation-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"},
{label: "Terraform", value: "tf"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myTextInput": {
    "title": "My text input",
    "icon": "My icon",
    "description": "My text input",
    "type": "string",
    // highlight-start
    "minLength": 1,
    "maxLength": 32,
    "pattern": "^[a-zA-Z0-9-]*-service$"
    // highlight-end
  }
}
```

</TabItem>

<TabItem value="array">

```json showLineNumbers
{
  "myTextArrayInput": {
    "title": "My text array input",
    "icon": "My icon",
    "description": "My text array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "minLength": 1,
      "maxLength": 32,
      "pattern": "^[a-zA-Z0-9-]*-service$"
    }
    // highlight-end
  }
}
```

</TabItem>

<TabItem value="tf">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    string_props = {
      myTextInput = {
        title       = "My text input"
        description = "My text input"
        default     = "My default"
        minLength   = 1
        maxLength   = 32
        pattern     = "^[a-zA-Z0-9-]*-service$"
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>