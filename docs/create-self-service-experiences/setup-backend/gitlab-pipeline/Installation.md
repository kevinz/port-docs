---
sidebar_position: 1
---

# 安装

GitLab 管道可通过[Port's execution agent](/create-self-service-experiences/setup-backend/webhook/port-execution-agent/port-execution-agent.md) 被用于。

让我们安装代理并配置它以触发 GitLab Pipelines。

## 先决条件

* [Helm](https://helm.sh) 必须安装才能使用海图。有关安装的更多详情，请参阅 Helm 的[documentation](https://helm.sh/docs) ；
* 与 Kafka 的连接凭证由 Port 提供给您；
* 如果想触发 GitLab Pipelines，需要有[GitLab trigger token](https://docs.gitlab.com/ee/ci/triggers/) 。

:::important  触发令牌

要触发 GitLab Pipeline，需要提供一个 GitLab[trigger token](https://docs.gitlab.com/ee/ci/triggers/#create-a-trigger-token) 作为环境变量。

要向代理提供触发令牌，可将 helm chart 传递给一个环境变量，变量名为 "GitLab 组 "和 "GitLab 项目 "的组合，以下划线 (`_`)分隔，名称区分大小写。

例如: "group_project=token

您可以为 GitLab 环境中的不同组和项目加载多个触发令牌。

:::

## 安装代理

1. 使用以下命令添加 Port 的 Helm 软件源: 

```bash showLineNumbers
helm repo add port-labs https://port-labs.github.io/helm-charts
```

:::note 如果之前已经添加了此 repo，请运行 `helm repo update` 获取最新版本的图表。 然后运行 `helm search repo port-labs` 查看图表。

:::

2.使用以下命令安装 `port-agent` 图表: 

```bash showLineNumbers
helm install my-port-agent port-labs/port-agent \
    --create-namespace --namespace port-agent \
    --set env.normal.PORT_ORG_ID=YOUR_ORG_ID \
    --set env.normal.KAFKA_CONSUMER_GROUP_ID=YOUR_KAFKA_CONSUMER_GROUP \
    --set env.secret.PORT_CLIENT_ID=YOUR_PORT_CLIENT_ID \
    --set env.secret.PORT_CLIENT_SECRET=YOUR_PORT_CLIENT_SECRET \
    --set env.secret.<YOUR GITLAB GROUP>_<YOUR GITLAB PROJECT>=YOUR_GITLAB_TOKEN
```

#### 自主托管 GitLab

如果使用私人 GitLab 环境，请将 `GITLAB_URL` 环境变量引用到 Port 代理安装中: 

```bash showLineNumbers
--set env.normal.GITLAB_URL
```

完成！**Port 的执行代理**已在您的环境中运行，并将触发您配置的任何 GitLab 管道。

## 控制有效载荷

通过 Port 代理，您可以控制触发管道时发送到 gitlab api 的有效载荷。

通过自定义 Pipeline 的有效载荷，您可以控制发送到 gitlab 的数据以及触发 Pipeline 的方式。

默认情况下，Port 代理会被用于此默认映射来触发 gitlab 管道: 

<details>
<summary>Default Gitlab port agent mapping</summary>

```json showLineNumbers
[
  {
    "enabled": ".payload.action.invocationMethod.type == \"GITLAB\"",
    "url": "(env.GITLAB_URL // \"https://gitlab.com/\") as $baseUrl | (.payload.action.invocationMethod.groupName + \"/\" +.payload.action.invocationMethod.projectName) | @uri as $path | $baseUrl + \"api/v4/projects/\" + $path + \"/trigger/pipeline\"",
    "body": {
      "ref": ".payload.properties.ref // .payload.action.invocationMethod.defaultRef // \"main\"",
      "token": ".payload.action.invocationMethod.groupName as $gitlab_group | .payload.action.invocationMethod.projectName as $gitlab_project | env[($gitlab_group | gsub(\"/\"; \"_\")) + \"_\" + $gitlab_project]",
      "variables": ".payload.action.invocationMethod as $invocationMethod | .payload.properties | to_entries | map({(.key): (.value | tostring)}) | add | if $invocationMethod.omitUserInputs then {} else . end",
      "port_payload": "if .payload.action.invocationMethod.omitPayload then {} else . end"
    }
  }
]
```

</details>

有关有效载荷映射的更多信息，请访问[Control the payload mapping](/create-self-service-experiences/setup-backend/webhook/port-execution-agent/control-the-payload.md) 页面。