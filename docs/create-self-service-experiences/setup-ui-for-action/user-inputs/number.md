---
sidebar_position: 2
description: 数字Number 是数字数据的基本输入法

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 编号

数字是数值数据的基本输入。

## 💡 常用数字用法

数字输入类型可用于存储任何数字数据，例如

* 内存/存储分配；
* 副本计数；
* 保留数据的天数；
* 等等。

在[live demo](https://demo.getport.io/self-serve) 自助服务中枢页面，我们可以看到**scaffold new service** 操作，其 `K8s Replica Count` 输入是一个数字输入。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Select (Enum)", value: "enum"},
{label: "Array", value: "array"},
{label: "Enum Array", value: "enumArray"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myNumberInput": {
    "title": "My number input",
    "icon": "My icon",
    "description": "My number input",
    // highlight-start
    "type": "number",
    // highlight-end
    "default": 7
  }
}
```

</TabItem>
<TabItem value="enum">

```json showLineNumbers
{
  "myNumberSelectInput": {
    "title": "My number select input",
    "icon": "My icon",
    "description": "My number select input",
    "type": "number",
    // highlight-next-line
    "enum": [1, 2, 3, 4]
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myNumberArrayInput": {
    "title": "My number array input",
    "icon": "My icon",
    "description": "My number array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "number"
    }
    // highlight-end
  }
}
```

</TabItem>
<TabItem value="enumArray">

```json showLineNumbers
{
  "myNumberArray": {
    "title": "My number-selection array input",
    "icon": "My icon",
    "description": "My number-selection array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "number",
      "enum": [1, 2, 3, 4],
      "enumColors": {
        "1": "red",
        "2": "green",
        "3": "blue"
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
    number_props = {
      "myNumberInput" = {
        title       = "My number input"
        description = "My number input"
        default     = 7
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
    number_props = {
      "myNumberInput" = {
        title       = "My number input"
        description = "My number input"
        enum        = [1, 2, 3, 4]
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
      "myNumberArrayInput" = {
        title       = "My number array input"
        description = "My number array input"
        number_items = {}
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>

## 验证号码

数字验证支持以下操作符: 

* 范围

使用 "最小值 "和 "最大值 "关键字(或 "独占最小值 "和 "独占最大值 "表达独占范围)的组合指定数字范围。

如果 _x_ 是要验证的值，则以下条件必须成立: 

* _x_ ≥ `最小值
* _x_ > `专属最小值
* _x_ ≤ `最大值
* _x_ < `独占最大值

<Tabs groupId="validation-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"},
{label: "Terraform", value: "tf"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myNumberInput": {
    "title": "My number input",
    "icon": "My icon",
    "description": "My number input",
    "type": "number",
    // highlight-start
    "minimum": 0,
    "maximum": 50
    // highlight-end
  }
}
```

</TabItem>

<TabItem value="array">

```json showLineNumbers
{
  "myNumberArrayInput": {
    "title": "My number array input",
    "icon": "My icon",
    "description": "My number array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "number",
      "exclusiveMinimum": 0,
      "exclusiveMaximum": 50
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
    number_props = {
      "myNumberInput" = {
        title       = "My number input"
        description = "My number input"
        minimum     = 0
        maximum     = 50
      }
    }
  }
  # highlight-end
}
```

</TabItem>
</Tabs>