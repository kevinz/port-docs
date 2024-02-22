# 为服务运行部署

在下面的指南中，您将在 Port 中构建一个自助操作，在幕后执行 Azure 管道。

### 先决条件

* 您需要[Port credentials](../../../../build-your-software-catalog/sync-data-to-catalog/api/api.md#find-your-port-credentials) 来创建操作。
* 您需要您的 Azure DevOps 组织。
* 您需要在 Azure Pipelines yaml 中配置的 webhook 名称。

#### 创建一个 Azure Pipeline

首先，我们需要设置一个 Azure Pipelines，实现我们的业务逻辑，以创建新的部署。

下面是工作流程的骨架--`deploy.yml`(包括部署逻辑的占位符): 

<details>
<summary>Deployment Azure pipeline</summary>

```yaml showLineNumbers
trigger: none

resources:
  webhooks:
    - webhook: { WEBHOOK_NAME }
      connection: { SERVICE_CONNECTION_NAME }
stages:
  # ADD YOUR DEPLOYMENT LOGIC HERE!
```

</details>

## 创建部署蓝图

让我们配置一个 "部署 "蓝图，它的基本结构是

```json showLineNumbers
{
  "identifier": "deployment",
  "title": "Deployment",
  "icon": "Deployment",
  "schema": {
    "properties": {
      "jobUrl": {
        "type": "string",
        "format": "url",
        "title": "Job URL"
      },
      "deployingUser": {
        "type": "string",
        "title": "Deploying User"
      },
      "imageTag": {
        "type": "string",
        "title": "Image Tag"
      },
      "commitSha": {
        "type": "string",
        "title": "Commit SHA"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

## 创建 Port 操作

现在让我们配置一个自助服务操作。 添加一个 `CREATE` 操作，开发人员每次要为服务启动新的部署时都会触发该操作。

以下是该操作的 JSON 文件: 

```json showLineNumbers
{
  "identifier": "deploy_service",
  "title": "Deploy Service",
  "icon": "DeployedAt",
  "userInputs": {
    "properties": {
      "service": {
        "type": "string",
        "description": "The service to deploy"
      },
      "environment": {
        "type": "string",
        "description": "The environment to deploy the service to"
      }
    },
    "required": ["service", "environment"]
  },
  // highlight-start
  "invocationMethod": {
    "type": "AZURE-DEVOPS",
    "org": "<AZURE-DEVOPS-ORG>",
    "webhook": "<AZURE-DEVOPS-WEBHOOK-NAME>"
  },
  // highlight-end
  "trigger": "CREATE",
  "description": "Deploy a service to the environment"
}
```

:::note 不要忘记替换 `AZURE-DEVOPS-ORG` 和 `AZURE-DEVOPS-WEBHOOK-NAME` 的占位符。

:::

要了解如何生成与 Port 交互的 API 令牌，请访问[here](../../../../build-your-software-catalog/sync-data-to-catalog/api/#generate-an-api-token) 。

要更新 Port 中的操作状态，请在 Azure 管道中被引用以下 API 调用: 

```bash
curl -X PATCH \
 -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
 -H "Content-Type: application/json" \
 -d '{"status": "SUCCESS"}' \
 https://api.getport.io/v1/actions/runs/YOUR_RUN_ID
```

## 摘要

在这个示例中，我们向你介绍了自助服务操作的概念，以及如何创建一个触发 Azure Pipelines 的简单操作。 你可以将这个示例作为你自己的自助服务操作的一个起点。