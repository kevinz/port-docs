---
sidebar_position: 5
---

# 定制 GitOps 集成

如果 Port 现有的 GitOps Provider 无法满足您的使用需求，您可以使用我们的 API 创建自定义的 GitOps 集成。

:::tip 在编写自定义逻辑将 Port 集成到 GitOps 流程之前，您应该查看我们现有的[Git providers](./git.md) 和[CI/CD integrations](../ci-cd/ci-cd.md) 。

:::

## 💡 自定义 GitOps 常见用例

* 将 Git Provider 作为**微服务**、**包**、**库**和其他软件目录资产的真实来源；
* 允许开发人员通过更新其 Git 仓库中的文件来保持目录的最新状态；
* 创建一种标准化的方式来记录组织内的软件目录资产；
* 等等。

## 使用自定义 GitOps 管理实体

要使用 GitOps 管理实体，可在微服务仓库中添加一个包含实体 JSON 的 `json` 文件。

然后，每当您的 CI/CD 流程运行时，自动检查将审查文件内容是否发生变化，并通过简单的 API 调用将新文件内容发送到 Port 的 API，从而始终保持微服务实体的最新状态。

## 示例

:::info 在本例中，您将为[the quickstart](../../../quickstart.md#define-a-blueprint) 中的 "Microservice "蓝图创建一个实体。

:::

创建一个本地 JSON 文件，命名为 `entity.json`，内容如下: 

```json showLineNumbers
{
  "identifier": "notification-microservice",
  "title": "Notification Service",
  "properties": {
    "url": "https://www.github.com/user/notification"
  }
}
```

然后作为 CI/CD 的一部分，使用[create entities](../sync-data-to-catalog.md#creating-entities) route 添加新的 `Microservice`: 

```bash showLineNumbers
blueprint_id='microservice'

curl --location --request POST "https://api.getport.io/v1/blueprints/${blueprint_id}/entities" \
    --header "Authorization: Bearer $access_token" \
    --header "Content-Type: application/json" \
    --data @entity.json

# The output of the command contains the content of the resulting entity
```

现在，只要您的 CI/CD 运行，它就会向 Port 报告实体的最新定义，直接在 Port 中为您提供最新信息。

:::note 请记住，API 请求需要访问令牌，如果需要生成一个新的访问令牌，请参阅[Getting an API token](../api/api.md#get-api-token) 。

:::