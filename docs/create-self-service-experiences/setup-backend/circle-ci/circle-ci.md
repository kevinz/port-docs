# Circle CI Action 

Port's Circle CI Action 可以使用客户提供的输入和[`port_payload`](../../self-service-actions-deep-dive/self-service-actions-deep-dive.md#action-message-structure) ，触发[new pipeline](https://circleci.com/docs/api/v2/index.html?utm_source=google&amp;utm_medium=sem&amp;utm_campaign=sem-google-dg--emea-en-dsa-tCPA-auth-nb&amp;utm_term=g_-_c__dsa_&amp;utm_content=&amp;gclid=Cj0KCQiAmNeqBhD4ARIsADsYfTeWb6EyzetX9OD0cFdfR--Tt8oOEf8CXnhyoRT46HMCGgbbVytPWG0aAloJEALw_wcB#operation/triggerPipeline)。

![Port Kafka Architecture](../../../../static/img/self-service-actions/setup-backend/circleci/circle-ci-agent-architecture.png)

上图所示步骤如下: 

1. Port 向主题发布调用的包含管道详细信息的 "Action "消息；
2. 安全主题("ORG_ID.runs")保存所有的操作调用；
3. Port 的执行代理从 Kafka 主题中提取新的触发事件，并触发 Circle CI 管道。

## 先决条件

* [Helm](https://helm.sh) 必须安装才能使用海图。有关安装的更多详情，请参阅 Helm 的[documentation](https://helm.sh/docs) ；
* 与 Kafka 的连接凭证由 Port 提供给您；
* 如果要触发 Circle CI 管道，需要有一个 Circle CI[personal API token](https://app.circleci.com/settings/user/tokens) 。

## 进一步的步骤

* 请参阅[Triggering example](#Triggering-example) Circle CI。
* 通过 Intercom 联系我们，为您的组织设置 Kafka 主题。
* [Install the Port execution agent to trigger the Circle CI pipeline](#Installation).

## 触发示例

创建以下蓝图、操作和映射，以触发 Circle CI 管道。

<details>
<summary>Blueprint</summary>

```json showLineNumbers
{
  "identifier": "circle_ci_project",
  "title": "CircleCI Project",
  "icon": "CircleCI",
  "schema": {
    "properties": {
      "project_slug": {
        "title": "Slug",
        "type": "string"
      }
    },
    "required": ["project_slug"]
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
    "identifier": "trigger_circle_ci_pipeline",
    "title": "Trigger CircleCI pipeline",
    "icon": "CircleCI",
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
      "url": "https://circleci.com"
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
    "enabled": ".action == \"trigger_circle_ci_pipeline\"",
    "url": "(env.CIRCLE_CI_URL // \"https://circleci.com\") as $baseUrl | .payload.entity.properties.project_slug | @uri as $path | $baseUrl + \"/api/v2/project/\" + $path + \"/pipeline\"",
    "headers": {
      "Circle-Token": "env.CIRCLE_CI_TOKEN"
    },
    "body": {
      "branch": ".payload.properties.branch // \"main\"",
      "parameters": ".payload.action.invocationMethod as $invocationMethod | .payload.properties | to_entries | map({(.key): (.value | tostring)}) | add | if $invocationMethod.omitUserInputs then {} else . end"
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
    --set env.secret.CIRCLE_CI_TOKEN=YOUR_CIRCLE_CI_PERSONAL_TOKEN
```
