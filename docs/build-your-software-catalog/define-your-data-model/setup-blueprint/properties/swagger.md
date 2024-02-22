---

sidebar_position: 15

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# Swagger UI

Swagger UI` 属性被用来在 Port<PortTooltip id="entity">实体</PortTooltip>中引用和显示[OpenAPI](https://www.openapis.org/) 和/或[AsyncAPI](https://www.asyncapi.com/) 规范文件。

使用该属性将自动在每个[entity page](/customize-pages-dashboards-and-plugins/page/entity-page.md) 中创建一个附加选项卡，以[Swagger UI](https://swagger.io/) 格式显示规范文件。在该选项卡中，您可以直接从 Port 对规范目标执行 HTTP 调用。

下面是实体页面中 "Swagger UI "选项卡的示例: 

<img src='/img/software-catalog/blueprint/swaggerUiExample.png' width='85%' />

## OpenAPI

#### 定义

<Tabs groupId="definition" defaultValue="url" values={[
{label: "URL", value: "url"},
{label: "JSON", value: "json"},
{label: "YAML", value: "yaml"}
]}>

<TabItem value="url">

使用 URL 格式时，Port 将查询所引用 URL 的 OpenAPI 规范，并希望得到 JSON OpenAPI 规范

:::note 

在使用 URL 进行 `open-api` 显示时，请确保您的服务器允许来自 `app.getport.io` 的跨源 (CORS) 请求。

要从 AWS S3 存储桶提供 OpenAPI 规范，请向存储桶添加 CORS 策略，允许来自 `app.getport.io` 的请求，请查看[AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/enabling-cors-examples.html?icmpid=docs_amazons3_console) 了解更多信息。

:::

<Tabs groupId="lang" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myOpenApi": {
    "title": "My Open API",
    // highlight-start
    "type": "string",
    "format": "url",
    "spec": "open-api",
    // highlight-end
    "description": "Open-API Prop"
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  properties {
    identifier = "myOpenApi"
    title      = "My Open Api"
    required   = false
    type       = "string"
    format     = "url"
    spec       = "open-api"
  }
  # highlight-end
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="json">

使用 JSON 类型时，您需要将完整的 JSON OpenAPI 规范作为一个对象提供给实体: 

<Tabs groupId="lang" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myOpenApi": {
    "title": "My Open API",
    // highlight-start
    "type": "object",
    "spec": "open-api",
    // highlight-end
    "description": "Open-API Prop"
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  properties = {
    props_object = {
      myOpenApi = {
        title      = "My Open Api"
        required   = false
        spec       = "open-api"
      }
    }
  }
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="yaml">

使用 YAML 类型时，您需要向实体提供完整的 YAML OpenAPI 规范: 

<Tabs groupId="lang" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myOpenApi": {
    "title": "My Open API",
    // highlight-start
    "type": "string",
    "format": "yaml",
    "spec": "open-api",
    // highlight-end
    "description": "Open-API Prop"
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  properties = {
    string_props = {
      "myYamlProp" = {
        title      = "My yaml"
        required   = false
        format     = "yaml"
      }
    }
  }
}
```

</TabItem>

</Tabs>

</TabItem>

</Tabs>

---

### 示例

以下是提供 OpenAPI 规范时，特定实体页面中 Swagger 标签的显示方式: 

![OpenAPI Example](/img/software-catalog/widgets/openAPI.png)

## AsyncAPI

#### 定义

<Tabs groupId="definition" defaultValue="url" values={[
{label: "URL", value: "url"},
{label: "JSON", value: "json"},
{label: "YAML", value: "yaml"}
]}>

<TabItem value="url">

当使用 URL 格式时，Port 将查询 Provider 提供的 URL 以获取 AsyncAPI 规范，并期望得到 JSON AsyncAPI 规范

:::note 

当使用 URL 显示 "async-api "时，请确保您的服务器允许来自 "app.getport.io "的跨源 (CORS) 请求

要从 AWS S3 存储桶提供 OpenAPI 规范，请向存储桶添加 CORS 策略，允许来自 `app.getport.io` 的请求，请查看[AWS documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/enabling-cors-examples.html?icmpid=docs_amazons3_console) 了解更多信息。

:::

<Tabs groupId="lang" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myAsyncApi": {
    "title": "My Async API",
    // highlight-start
    "type": "string",
    "format": "url",
    "spec": "async-api",
    // highlight-end
    "description": "async-api Prop"
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  properties {
    identifier = "myAsyncApi"
    title      = "My Async API"
    required   = false
    type       = "string"
    format     = "url"
    spec       = "async-api"
  }
  # highlight-end
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="json">

使用 JSON 类型时，需要将完整的 JSON AsyncAPI 规范作为一个对象提供给 Entity: 

<Tabs groupId="lang" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myAsyncApi": {
    "title": "My Async API",
    // highlight-start
    "type": "object",
    "spec": "async-api",
    // highlight-end
    "description": "async-api Prop"
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  properties = {
    props_object = {
      myAsyncApi = {
        title      = "My Async Api"
        required   = false
        spec       = "async-api"
      }
    }
  }
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="yaml">

使用 YAML 类型时，您需要向实体提供完整的 YAML AsyncAPI 规范: 

<Tabs groupId="lang" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myOpenApi": {
    "title": "My Async API",
    // highlight-start
    "type": "string",
    "format": "yaml",
    "spec": "async-api",
    // highlight-end
    "description": "Async-API Prop"
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  properties = {
    string_props = {
      "myYamlProp" = {
        title      = "My yaml"
        required   = false
        format     = "yaml"
      }
    }
  }
}
```

</TabItem>

</Tabs>

</TabItem>

</Tabs>

---

### 示例

以下是提供 AsyncAPI 规范时，特定实体页面中 Swagger 标签的显示方式: 

![AsyncAPI Example](/img/software-catalog/widgets/asyncAPI.png)