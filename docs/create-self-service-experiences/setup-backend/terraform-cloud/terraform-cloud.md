# Terraform 云操作

Port 的 Terraform Cloud Action 可使用客户提供的输入和[`port_payload`](../../self-service-actions-deep-dive/self-service-actions-deep-dive.md#action-message-structure) 触发[Terraform Cloud run](https://developer.hashicorp.com/terraform/cloud-docs/api-docs/run#create-a-run) 。

![Port Kafka Architecture](../../../../static/img/self-service-actions/setup-backend/terraform-cloud/terraform-cloud-agent-architecture.png)

上图所示步骤如下: 

1. Port 向主题发布调用的包含管道详细信息的 "Action "消息；
2. 安全主题("ORG_ID.runs")保存所有的操作调用；
3. Port 的执行代理从 Kafka 主题中提取新的触发事件，并触发 Terraform 云运行。

## 先决条件

* [Helm](https://helm.sh) 必须安装才能使用海图。有关安装的更多详情，请参阅 Helm 的[documentation](https://helm.sh/docs) ；
* 与 Kafka 的连接凭证由 Port 提供给您；
* 如果要触发 Terraform Cloud 运行，需要有 Terraform Cloud[User token](https://developer.hashicorp.com/terraform/cloud-docs/users-teams-organizations/users#api-tokens) 或[Team token](https://developer.hashicorp.com/terraform/cloud-docs/users-teams-organizations/api-tokens#team-api-tokens) 。

:::warning 
**注意**: 不能使用组织令牌访问 Terraform 云运行端点。 必须使用**用户令牌**或**团队令牌**访问。

:::

## 进一步的步骤

* 请参阅[Triggering example](#Triggering-example) for Terraform Cloud。
* 通过 Intercom 联系我们，为您的组织设置 Kafka 主题。
* [Install the Port execution agent to trigger the Terraform Cloud Run](#Installation).

## 触发示例

创建以下蓝图、操作和映射，以触发 Terraform 云运行。

<details>
<summary>Blueprint</summary>

```json showLineNumbers
{
  "identifier": "terraform_cloud_workspace",
  "title": "Terraform Cloud Workspace",
  "icon": "Terraform",
  "schema": {
    "properties": {
      "workspace_id": {
        "title": "Workspace Id",
        "type": "string"
      }
    },
    "required": ["workspace_id"]
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</details>

<details>
<summary>Action</summary>

```json showLineNumbers
[
  {
    "identifier": "trigger_tf_run",
    "title": "Trigger TF Cloud run",
    "icon": "Terraform",
    "userInputs": {
      "properties": {},
      "required": [],
      "order": []
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "agent": true,
      "synchronized": false,
      "method": "POST",
      "url": "https://app.terraform.io/api/v2/runs/"
    },
    "trigger": "DAY-2",
    "requiredApproval": false
  }
]
```

</details>

<details>
<summary>Mapping - (Should be saved as a file named `invocations.json`)</summary>

:::info 要了解有关 `controlThePayload` 配置的更多信息，请参阅[Control the payload](/create-self-service-experiences/setup-backend/webhook/port-execution-agent/control-the-payload.md) 文档。

:::

```json
[
  {
    "enabled": ".action == \"trigger_tf_run\"",
    "headers": {
      "Authorization": "\"Bearer \" + env.TF_TOKEN",
      "Content-Type": "\"application/vnd.api+json\""
    },
    "body": {
      "data": {
        "attributes": {
          "is-destroy": false,
          "message": "\"Triggered via Port\"",
          "variables": ".payload.properties | to_entries | map({key: .key, value: .value})"
        },
        "type": "\"runs\"",
        "relationships": {
          "workspace": {
            "data": {
              "type": "\"workspaces\"",
              "id": ".payload.entity.properties.workspace_id"
            }
          }
        }
      }
    },
    "report": {
      "status": "if .response.statusCode == 201 then \"SUCCESS\" else \"FAILURE\" end",
      "link": "\"https://app.terraform.io/app/\" + .body.payload.entity.properties.organization_name + \"/workspaces/\" + .body.payload.entity.properties.workspace_name + \"/runs/\" + .response.json.data.id",
      "externalRunId": ".response.json.data.id"
    }
  }
]
```

</details>

## 安装

1. 使用以下命令添加 Port 的 Helm 软件源: 

```sh showLineNumbers
helm repo add port-labs https://port-labs.github.io/helm-charts
```

:::note 如果之前已经添加了此 repo，请运行 `helm repo update` 获取最新版本的图表。 然后运行 `helm search repo port-labs` 查看图表。

:::

2.用上述映射创建一个名为 `invocations.json` 的 JSON 文件。
3.填写所需数值后，使用以下命令安装 `port-agent` 图表: 

```sh showLineNumbers
helm install my-port-agent port-labs/port-agent \
    --create-namespace --namespace port-agent \
    --set-file controlThePayloadConfig=./invocations.json \
    --set env.normal.PORT_ORG_ID=YOUR_ORG_ID \
    --set env.normal.KAFKA_CONSUMER_GROUP_ID=YOUR_KAFKA_CONSUMER_GROUP \
    --set env.secret.PORT_CLIENT_ID=YOUR_PORT_CLIENT_ID \
    --set env.secret.PORT_CLIENT_SECRET=YOUR_PORT_CLIENT_SECRET \
    --set env.secret.TF_TOKEN=YOUR_TERRAFORM_CLOUD_TOKEN
```