# Webhook

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Image from "@theme/IdealImage";
import WebhookArchitecture from '../../../../static/img/build-your-software-catalog/sync-data-to-catalog/webhook/webhook-architecture.png';
import ExampleGithubPRWebhook from './examples/resources/github/_example_github_pr_configuration.mdx';

通过使用 Port 的通用 webhook 集成，您可以将数据从任何提供外发 webhook 的源或服务摄入软件目录，即使 Port 没有为该源提供本地集成。

<center>

<Image img={WebhookArchitecture}></Image>

</center>

## 💡 Webhook 常见用例

例如，我们的通用 webhook 可让您直接从第三方服务中获取实时数据，从而轻松填充软件目录: 

* 映射所有**Snyk 漏洞**、**Jira 问题**、**SonarQube 报告**和其他数据源；
* 进行单一属性更新--根据来自 **Pager Duty** 或 **OpsGenie** 的事件更新服务的当前待命状态；
* 对软件目录进行基于事件的实时更新；
* 为您使用的第三方服务所提供的所有数据创建单一视图；
* 等等。

## 工作原理

Port 可为您提供自定义 webhook 端点，您可以将其用作您所引用服务(如[GitHub](https://docs.github.com/en/webhooks-and-events/webhooks/about-webhooks) 、[Sentry](https://docs.sentry.io/product/integrations/integration-platform/webhooks/) 等)提供的自定义集成的目标。

每个 webhook 端点都能接收[custom mapping](#mapping-configuration) ，这样就能轻松地将来自第三方服务的事件有效载荷转化为软件目录中的实体。

自定义映射利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 webhook 有效负载的现有字段和值进行选择、修改、连接、转换和其他操作。

:::tip 通过被用于 webhook 映射，你可以创建/更新一个完整的实体，也可以选择只更新实体上的一个属性。

:::

## Webhook 配置

webhook 配置就是你指定的方式: 

* 自定义 webhook 集成的基本[metadata](#metadata-configuration) ；
* 控制从有效负载创建实体的[mapping configuration](#mapping-configuration) ；
* 用于确保到达 Port 的有效负载确实是由您授权的第三方发送的[security configuration](#security-configuration) 。

下面是一个 webhook 配置示例: 

<ExampleGithubPRWebhook/>

## 配置结构

### 元数据配置

网络钩子的元数据配置除了控制网络钩子是否处于活动状态外，还包括与网络钩子在 Port 用户界面中的可见性和显示有关的所有属性。

下面是一个元数据配置示例: 

<Tabs groupId="definition" queryString defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "tf"},
]}>

<TabItem value="api">

```json showLineNumbers
{
  // highlight-start
  "identifier": "pullRequestMapper",
  "title": "Pull Request Mapper",
  "description": "A webhook configuration for pull-request events from GitHub",
  "icon": "Github",
  "enabled": true,
  // highlight-end
  "mappings": {
    ...
  },
  "security": {
    ...
  }
}
```

</TabItem>

<TabItem value="tf">

```hcl showLineNumbers
resource "port_webhook" "myWebhook" {
    // highlight-start
    identifier = "pullRequestMapper"
    title = "Pull Request Mapper"
    description = "A webhook configuration for pull-request events from GitHub"
    icon = "Github"
    enabled = true
    // highlight-end
    mappings = {
      ...
    }
    security = {
      ...
    }
  }
```

</TabItem>
</Tabs>

#### 结构表


| Field         | Description           | Notes                                                                                                    |
| ------------- | --------------------- | -------------------------------------------------------------------------------------------------------- |
| `identifier`  | Unique identifier     | The identifier is used for API calls, programmatic access and distinguishing between different webhooks  |
| `title`       | Name                  | **Required**. Human-readable name for the webhook                                                        |
| `description` | Description           |                                                                                                          |
| `icon`        | Icon for the webhook  | See the [full icon list](../../define-your-data-model/setup-blueprint/setup-blueprint.md#full-icon-list) |
| `enabled`     | Is the webhook active | If the integration id disabled (`"enabled": false`) then any incoming event will be dropped              |


### 映射配置

webhook 的映射配置定义了如何将 webhook 事件有效负载映射到一个(或多个)Port实体。

映射配置利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) ，将事件有效载荷中的信息提取到 Port 实体属性中。

下面是一个映射配置示例: 

<Tabs groupId="definition" queryString defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "tf"},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "identifier": "pullRequestMapper",
  "title": "Pull Request Mapper",
  "enabled": true,
  ...
  // highlight-start
  "mappings": [
    {
      "blueprint": "pullRequest",
      "filter": ".headers.\"X-GitHub-Event\" == \"pull_request\"",
      "entity": {
        "identifier": ".body.pull_request.id | tostring",
        "title": ".body.pull_request.title",
        "properties": {
          "author": ".body.pull_request.user.login",
          "url": ".body.pull_request.html_url"
        }
      }
    }
  ],
  // highlight-end
  "security": {
    ...
  }
}
```

</TabItem>

<TabItem value="tf">

```hcl showLineNumbers
resource "port_webhook" "myWebhook" {
    identifier = "pullRequestMapper"
    title = "Pull Request Mapper"
    enabled = true
    ...
    // highlight-start
    mappings = [
      {
        blueprint = "pullRequest"
        filter = ".headers.\"X-GitHub-Event\" == \"pull_request\""
        entity = {
          identifier = ".body.pull_request.id | tostring"
          title = ".body.pull_request.title"
          properties = {
            author = ".body.pull_request.user.login"
            url = ".body.pull_request.html_url"
          }
        }
      }
    ]
    // highlight-end
    security = {
      ...
    }
  }
```

</TabItem>

</Tabs>

#### 结构

* 映射配置的根键是 `mappings` 键: 

```json showLineNumbers
{
  ...
  // highlight-next-line
  "mappings": [
    {
      # mapping
    }
  ]
  ...
}
```

映射键存储映射的***数组，这样就可以通过同一有效载荷在多个蓝图中创建/更新多个实体。

现在，让我们来探讨一下单个映射对象的结构: 

* 蓝图 "键被用来引用蓝图的标识符，以便根据 webhook 有效负载创建/更新实体: 

```json showLineNumbers
{
  ...
  "mappings": [
    {
      // highlight-next-line
      "blueprint": "pullRequest",
      "filter": ".headers.\"X-GitHub-Event\" == \"pull_request\"",
      ...
    }
  ]
  ...
}
```

* 通过 `filter` 键，您可以准确过滤哪些发送到 webhook 的有效负载会被处理: 

```json showLineNumbers
{
  ...
  "mappings": [
    {
      "blueprint": "pullRequest",
      // highlight-next-line
      "filter": ".headers.\"X-GitHub-Event\" == \"pull_request\"" # JQ boolean query. If evaluated to false - skip the payload.
      ...
    }
  ]
  ...
}
```

* 通过`itemsToParse`键，可以从一个 webhook 事件中创建多个实体: 

```json showLineNumbers
{
  ...
  "mappings": [
    {
      "blueprint": "commits",
      // highlight-start
      "itemsToParse": ".body.pull_request.commits",
      // highlight-end
      // Checks if any of the modified files are in the frontend/src folder.
      "filter": ".item.modified | any(test(\"/frontend/src\"))",
      "entity": {
        "identifier": ".item.id | tostring",
        "title": ".item.message",
        "properties": {
          "author": ".item.author.email",
          "url": ".item.url",
          "repository": ".body.pusher.email"
        }
      }
    }
  ]
  ...
}
```

:::note 

* 任何 JQ 表达式都可以在此被用于，只要它的 evaluated 指向一个项目数组。
* 项目 "将作为一个键添加到 JQ 上下文中，该键包含对 "itemsToParse "中指定的数组中的项目的引用。可以使用 `.item.KEY_NAME`语法访问数组中对象的键，更多信息请参阅示例 JSON。

:::

* 实体 "键被用来通过 JQ 将 Webhook 有效负载中的信息映射到 Port 实体属性: 

```json showLineNumbers
{
  ...
  "mappings": [
    {
      ...
      "filter": ".headers.\"X-GitHub-Event\" == \"pull_request\"",
      // highlight-start
      "entity": {
        "identifier": ".body.pull_request.id | tostring",
        "title": ".body.pull_request.title",
        "properties": {
          "author": ".body.pull_request.user.login",
          "url": ".body.pull_request.html_url"
        }
      }
      // highlight-end
    }
  ]
  ...
}
```

#### 安全配置

当第三方服务向指定的 webhook URL 发送有效负载时，通常也会包含一个包含有效负载签名的标头，或一些约定的字符串，用于验证发送方。

签名可以通过在有效负载上运行 SHA-X(例如 SHA-1 或 SHA-256)散列函数，结合用户指定的或第三方服务在创建 webhook 时提供的secret值生成。

由于某些第三方服务不提供发送有效载荷签名的服务，而只提供直接发送约定字符串的服务，因此可使用安全选项 "plain"。 使用该选项时，签名将不加任何修改地与secret值进行比较。 它允许用户直接将签名与 Provider 提供的secret值进行比较。 这在需要更简单的安全机制的情况下非常有用。

webhook 的安全配置用于告诉 Port 如何验证与第三方请求一起发送的哈希签名。

下面是一个安全配置示例: 

<Tabs groupId="definition" queryString defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "tf"},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "identifier": "pullRequestMapper",
  ...
  "mappings": [
    ...
  ],
  // highlight-start
  "security": {
    "secret": "WEBHOOK_SECRET",
    "signatureHeaderName": "X-Hub-Signature-256",
    "signatureAlgorithm": "sha256",
    "signaturePrefix": "sha256=",
    "requestIdentifierPath": ".headers.\"X-GitHub-Delivery\""
  }
  // highlight-end
}
```

</TabItem>

<TabItem value="tf">

```hcl showLineNumbers
resource "port_webhook" "myWebhook" {
    identifier = "pullRequestMapper"
    ...
    mappings = [
      ...
    ]
    // highlight-start
    security = {
      secret = "WEBHOOK_SECRET"
      signature_header_name = "X-Hub-Signature-256"
      signature_algorithm = "sha256"
      signature_prefix = "sha256="
      request_identifier_path = ".headers.\"X-GitHub-Delivery\""
    }
    // highlight-end
}
```

</TabItem>

</Tabs>

:::tip 安全配置不是强制性的，但它确实提供了额外的安全保障，确保 Provider 只处理真正由第三方 webhooks 发送的有效负载。

如果不想在配置 webhook 时提供安全配置，只需在配置 webhook 时传递一个空对象: `"security": {}`。

:::

#### 结构

* 安全配置的根是 `security` 密钥: 

```json showLineNumbers
{
  ...
  // highlight-next-line
  "security": {
    "secret": "WEBHOOK_SECRET",
    "signatureHeaderName": "X-Hub-Signature-256",
    "signatureAlgorithm": "sha256",
    "signaturePrefix": "sha256=",
    "requestIdentifierPath": ".headers.\"X-GitHub-Delivery\""
  }
  ...
}
```

* secret "密钥用于指定用于验证接收到的有效负载的哈希签名的secret值: 
    - 根据服务的不同，secret值可能由第三方自动生成，也可能由您手动提供给第三方。

```json showLineNumbers
...
  "security": {
    // highlight-next-line
    "secret": "WEBHOOK_SECRET",
    "signatureHeaderName": "X-Hub-Signature-256",
    "signatureAlgorithm": "sha256",
    "signaturePrefix": "sha256=",
    "requestIdentifierPath": ".headers.\"X-GitHub-Delivery\""
  }
  ...
}
```

* signatureHeaderName "键被用来引用存储有效负载哈希签名的头的名称: 
    - 当 webhook 端点接收到新的有效负载时，它会将此标头的值与它将从接收到的有效负载中计算出的散列签名进行比较。

```json showLineNumbers
...
  "security": {
    "secret": "WEBHOOK_SECRET",
    // highlight-next-line
    "signatureHeaderName": "X-Hub-Signature-256",
    "signatureAlgorithm": "sha256",
    "signaturePrefix": "sha256=",
    "requestIdentifierPath": ".headers.\"X-GitHub-Delivery\""
  }
  ...
}
```

* 签名算法 "键被用来引用哈希算法，以创建有效载荷的哈希签名: 
    - **可用 Values: ** `sha1`、`sha256`、`plain`；
    - 当 webhook 端点接收到新的有效负载时，它将使用指定的算法来计算接收到的有效负载的散列签名。

```json showLineNumbers
...
  "security": {
    "secret": "WEBHOOK_SECRET",
    "signatureHeaderName": "X-Hub-Signature-256",
    // highlight-next-line
    "signatureAlgorithm": "sha256",
    "signaturePrefix": "sha256=",
    "requestIdentifierPath": ".headers.\"X-GitHub-Delivery\""
  }
  ...
}
```

:::info 使用 "plain "算法时，将不执行散列，保存在 Port webhook 配置中的secret值将与指定标头中的值进行比较，不做任何修改。

:::

* signaturePrefix "键用于指定一个静态前缀字符串，出现在 "signatureHeaderName "键中的 hashedSignature 之前: 
    - 例如，在 GitHub webhook 中，包含散列签名的头总是以 `sha256=` 开头，因此 webhook 应配置为签名前缀"sha256="`；

```json showLineNumbers
...
  "security": {
    "secret": "WEBHOOK_SECRET",
    "signatureHeaderName": "X-Hub-Signature-256",
    "signatureAlgorithm": "sha256",
    // highlight-next-line
    "signaturePrefix": "sha256=",
    "requestIdentifierPath": ".headers.\"X-GitHub-Delivery\""
  }
  ...
}
```

* requestIdentifierPath "键被用来引用 JQ 模式，该模式会产生 webhook 有效负载的唯一标识符: 
    - 该键用于防止 Port 对一个事件进行多次处理；
    - 例如，在 GitHub webhook 中，`X-GitHub-Delivery` 标头包含一个用于标识交付的 GUID。因此，webhook 应配置为`"requestIdentifierPath": ".headers.\"X-GitHub-Delivery/""`；

```json showLineNumbers
...
  "security": {
    "secret": "WEBHOOK_SECRET",
    "signatureHeaderName": "X-Hub-Signature-256",
    "signatureAlgorithm": "sha256",
    "signaturePrefix": "sha256=",
    // highlight-next-line
    "requestIdentifierPath": ".headers.\"X-GitHub-Delivery\""
  }
  ...
}
```

## 配置 webhook 端点

<Tabs queryString="operation">
<TabItem label="Using API" value="api">
<Tabs>
<TabItem label="Create webhook" value="create">

要创建新的 webhook，请向 `https://api.getport.io/v1/webhooks` 发送 HTTP POST 请求，并在请求正文中输入您的[webhook configuration](#configuration-structure) 。

API 响应将包括 webhook 的完整配置，包括以下重要字段: 

* `webhookKey` - 这是您创建的新通用 webhook 路由被用于的唯一标识符；
* `url` - 这是您需要传递给第三方网络钩子配置的完整 URL。与您创建的 webhook 配置相匹配的事件有效载荷应发送到此 URL；
    - url "的格式为https://ingest.getport.io/{webhookKey}`；
    - **注意: ** `https://ingest.getport.io`是不变的，只有`webhookKey`会在不同的网络钩子配置之间发生变化。

</TabItem>
<TabItem label="Update webhook" value="update">

要更新现有的 webhook，请向 "https://api.getport.io/v1/webhooks/{WEBHOOK_IDENTIFIER}"发出 HTTP PATCH 请求，并在请求正文中包含更新后的[webhook configuration](#configuration-structure) 。

在更新 webhook 配置时，支持部分更新，也就是说，你可以只在请求正文中传递需要更新的密钥。 任何你没有指定的密钥都将保持不变。

API 响应将包括 webhook 的更新配置。

</TabItem>
<TabItem label="Delete webhook" value="delete">

要删除现有的 webhook，请向 `https://api.getport.io/v1/webhooks/{WEBHOOK_IDENTIFIER}` 发送 HTTP DELETE 请求。

</TabItem>
</Tabs>
</TabItem>
<TabItem label="Using Port UI" value="ui">
<Tabs>
<TabItem label="Create webhook" value="create-ui">

以下是使用 Port UI 创建新 webhook 的详细步骤: 

1. 登录您的[Port account](https://app.getport.io)
2. 从顶部菜单中选择**生成器**
3. 选择您现有的[blueprint](/docs/quickstart.md#define-a-blueprint)
4. 点击蓝图**展开**按钮
5. 点击"... "图标，选择**最原始数据**。
6. 向下滚动到 "自定义集成 "部分。
7. 选择**自定义集成**；
    - 为您的 webhook 提供**标题**；
    - 选择是使用**标识符自动生成**选项还是指定您自己的标识符；
    - 为您的 webhook 提供描述；
    - 从下拉菜单中选择一个图标来代表你的 webhook；
    - 点击**下一步**
8.向下滚动到**JQ 映射**部分；
    - 该部分显示设置蓝图时创建的属性；
    - 查看映射，必要时进行修改；
9.最后，点击**创建**创建新的 webhook。

请注意，本明细表根据 Provider 提供的叙述，记录了使用 Port 用户界面创建 webhook 所涉及的步骤。

</TabItem>
<TabItem label="Update webhook" value="update-ui">

以下是使用 Port UI 更新 webhook 的详细步骤: 

1. 登录您的[Port account](https://app.getport.io)
2. 从顶部菜单中选择**生成器**
3. 选择您现有的[blueprint](/docs/quickstart.md#define-a-blueprint)
4. 点击蓝图**展开**按钮
5. 点击"... "图标并选择**最原始数据**
6. 向下滚动到**自定义集成**部分
7. 选择要修改的 webhook
8. 对 webhook 配置进行必要的更改
9. 单击**保存**保存更改

按照以下步骤，您就能根据 Providers 提供的叙述使用 Port UI 更新 webhook。

</TabItem>

<TabItem label="Delete webhook" value="delete-ui">

以下是使用 Port UI 删除 webhook 的详细步骤: 

1. 登录您的[Port account](https://app.getport.io)
2. 从顶部菜单中选择**生成器**
3. 选择您现有的[blueprint](/docs/quickstart.md#define-a-blueprint)
4. 点击蓝图**展开**按钮
5. 点击"... "图标并选择**最原始数据**
6. 向下滚动到**自定义集成**部分
7. 将鼠标悬停在要删除的 webhook 上
8. 将出现一个**删除**图标
9. 点击**删除**图标删除网络钩子

按照以下步骤，您就可以根据 Providers 提供的叙述，使用 Port UI 删除 webhook。

</TabItem>
</Tabs>
</TabItem>
</Tabs>

## 使用自定义 webhook

创建并配置好自定义 webhook 后，请转到第三方 Provider(如 GitHub、Sentry、Jira 等)，按照以下步骤完成 webhook 设置: 

* 进入第三方 Providers 的新 webhook 设置菜单
    - 例如在 GitHub: 转到所需的组织/资源库 -> 设置 -> Webhooks -> 添加 webhook。
* 将从 Port 收到的 webhook URL (`https://ingest.getport.io/{webhookKey}`)粘贴到指定 webhook 目标 URL 的字段中；
    - 例如在 GitHub 中: 将 webhook URL 粘贴到 `Payload URL` 字段中。
* 内容类型请选择 `application/json`(如适用)；
* 如果 "secret "值是由第三方生成的，请务必返回[update](?operation=update#configuring-webhook-endpoints) ，并在[security configuration](#security-configuration) 中添加secret值。

## 示例

有关实用配置及其相应的蓝图定义，请参阅[examples](./examples/examples.md) 页面。