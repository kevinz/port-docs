import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# 嵌入式 URL

嵌入的 URL "属性用于在 Port<PortTooltip id="entity">实体</PortTooltip>中嵌入和显示网页。使用该属性将自动在每个[entity page](/customize-pages-dashboards-and-plugins/page/entity-page.md) 中创建一个额外的选项卡，显示嵌入的内容。

在下面的示例中，我们看到的是 "发货 "实体页面，它是 "域 "蓝图上的一个实例。 该蓝图有一个名为 "架构 "的 "嵌入式 URL "属性，它会自动显示在专用标签页中: 

<img src='/img/software-catalog/blueprint/embeddedUrlExample.png' width='85%' />

## URL 类型

Port 支持以下 URL 类型: 

* **公共链接** - 指向公共网页的链接，不需要身份验证。
* **私人链接** -指向受 SSO 身份验证保护的网页的链接。要使用这种类型，您需要提供所需的参数，更多信息和示例请参阅[authentication](./authentication) 部分。

## 💡 常见嵌入式 URL Usage

* 显示服务的架构
* 显示和跟踪服务的 Datadog 仪表板
* 显示来自外部工具的图表

## 模式定义

<Tabs groupId="definition" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myEmbeddedUrl": {
    "title": "My Embedded URL",
    // highlight-start
    "type": "string",
    "format": "url",
    "spec": "embedded-url",
    // highlight-end
    "description": "embedded-url Prop",
    // specAuthentication is needed only when using a protected/private URL
    "specAuthentication": {
        "authorizationUrl": "https://app.com",
        "tokenUrl": "https://app.com",
        "clientId": "1234",
        "authorizationScope": [
          "api://xxxx-xxxx-xxxx-xxxx-xxxx/user.read"
        ]
      }

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
    identifier = "myEmbeddedUrl"
    title      = "My Embedded URL"
    required   = false
    type       = "string"
    format     = "url"
    spec       = "embedded-url"
  }
  # highlight-end
}
```

</TabItem>

</Tabs>

## 示例

### Datadog 仪表板

在本例中，我们嵌入了[Datadog](https://docs.datadoghq.com/dashboards/sharing/) 仪表板，以便直接在 Port 中获取应用程序指标。

在蓝图中添加 `embedded-URL` 属性: 

<details>
<summary>Blueprint property definition</summary>

```json showLineNumbers
{
  "datadog": {
    "title": "Datadog",
    "type": "string",
    "format": "url",
    "spec": "embedded-url"
  }
}
```

</details>

创建或编辑添加了 `Datadog` 属性的蓝图实体，并指定 Datadog 面板的 URL: 

![Datadog Entity edit example](/img/software-catalog/widgets/editEntityDatadog.png)

现在转到实体的特定实体页面，Datadog 仪表板将在专用选项卡中显示: 

![Datadog dashboard example](/img/software-catalog/widgets/datadog.png)

### 新 Relic 图表

在本例中，我们嵌入了一个 CPU 使用率[New Relic Chart](https://one.eu.newrelic.com/) ，以便直接在 Port 中获取基础设施指标。

在蓝图中添加 `embedded-URL` 属性: 

<details>
<summary>Blueprint property definition</summary>

```json showLineNumbers
{
  "cpuUsage": {
    "type": "string",
    "title": "CPU usage",
    "spec": "embedded-url",
    "format": "url"
  }
}
```

</details>
Go to new relic and extract the chart URL of a specific chart

![New Relic get embed URL](/img/software-catalog/widgets/GetEmbedUrlNewRelic.png)

创建或编辑您添加了 `cpuUsage` 属性的蓝图实体，并指定 CPU Usage 图表的 URL: 

![New Relic Entity edit example](/img/software-catalog/widgets/editEntityNewRelic.png)

现在进入实体的特定实体页面，CPU Usage 图表就会在专门的标签页中显示出来: 

![New Relic dashboard example](/img/software-catalog/widgets/new-relic.png)