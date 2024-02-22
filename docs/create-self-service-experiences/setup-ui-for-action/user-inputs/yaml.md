---
sidebar_position: 12
description: Yaml 是一种输入法，被用于用于以 YAML 保存对象定义

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# Yaml

Yaml 是用于将对象定义保存为 YAML 格式的输入。

## 💡 常见的 yaml Usage

yaml 输入类型可被用来存储任何基于键/值的数据，例如

* 配置；
* Helm 图表；
* 字典/哈希图
* 配置清单；
* `values.yml`；
* 等。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myYamlInput": {
    "title": "My yaml input",
    "icon": "My icon",
    "description": "My yaml input",
    // highlight-start
    "type": "string",
    "format": "yaml"
    // highlight-end
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myYamlArrayInput": {
    "title": "My yaml array input",
    "icon": "My icon",
    "description": "My yaml array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "format": "yaml"
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
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    string_props = {
      "myYamlInput" = {
        title       = "My yaml input"
        description = "My yaml input"
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
      "myYamlArrayInput" = {
        title       = "My yaml array input"
        description = "My yaml array input"
        string_items = {
          format = "yaml"
        }
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>