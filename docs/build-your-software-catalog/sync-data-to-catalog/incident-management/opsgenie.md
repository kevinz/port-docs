---

sidebar_position: 2

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import DockerParameters from "./_opsgenie_docker_params.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'
import OpsGenieAlertBlueprint from "../webhook/examples/resources/opsgenie/_example_opsgenie_alert_blueprint.mdx";
import OpsGenieAlertConfiguration from "../webhook/examples/resources/opsgenie/_example_opsgenie_alert_configuration.mdx";

# Opsgenie

我们的 Opsgenie 集成允许您根据映射和定义，将 Opsgenie 账户中的 "警报"、"服务 "和 "事件 "导入 Port。

## 常见被引用情况

* 在您的 Opsgenie 账户中映射 "警报"、"服务 "和 "事件"。
* 实时观察对象更改(创建/更新/删除)，并自动将更改应用到您的 Port 实体中。

## 先决条件

<Prerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                        | Description                                                                                                   | Required |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| `port.clientId`                  | Your port client id                                                                                           | ✅       |
| `port.clientSecret`              | Your port client secret                                                                                       | ✅       |
| `port.baseUrl`                   | Your port base url, relevant only if not using the default port app                                           | ❌       |
| `integration.identifier`         | Change the identifier to describe your integration                                                            | ✅       |
| `integration.type`               | The integration type                                                                                          | ✅       |
| `integration.eventListener.type` | The event listener type                                                                                       | ✅       |
| `integration.secrets.apiToken`   | The Opsgenie API token                                                                                        | ✅       |
| `integration.config.apiUrl`      | The Opsgenie API URL. If not specified, the default will be https://api.opsgenie.com                          | ✅       |
| `scheduledResyncInterval`        | The number of minutes between each resync                                                                     | ❌       |
| `initializePortResources`        | Default true, When set to true the integration will create default blueprints and the port App config Mapping | ❌       |


<br/>
<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-opsgenie-integration port-labs/port-ocean \
  --set port.clientId="CLIENT_ID"  \
  --set port.clientSecret="CLIENT_SECRET"  \
  --set initializePortResources=true  \
  --set integration.identifier="my-opsgenie-integration"  \
  --set integration.type="opsgenie"  \
  --set integration.eventListener.type="POLLING"  \
  --set integration.secrets.apiToken="API_TOKEN"  \
  --set integration.config.apiUrl="https://api.opsgenie.com"
```

</TabItem>

<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在 git 仓库的 `argocd/my-ocean-opsgenie-integration` 中创建一个内容为 `values.yaml` 的文件: 

:::note 请记住替换 `OPSGENIE_API_URL` 和 `OPSGENIE_API_TOKEN` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-opsgenie-integration
  type: opsgenie
  eventListener:
    type: POLLING
  config:
  // highlight-next-line
    apiUrl: OPSGENIE_API_URL
  secrets:
  // highlight-next-line
    apiToken: OPSGENIE_API_TOKEN
```

<br/>

2.创建以下 "my-ocean-opsgenie-integration.yaml "配置清单，安装 "my-ocean-opsgenie-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-opsgenie-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-opsgenie-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-opsgenie-integration/values.yaml
      // highlight-start
      parameters:
        - name: port.clientId
          value: YOUR_PORT_CLIENT_ID
        - name: port.clientSecret
          value: YOUR_PORT_CLIENT_SECRET
  - repoURL: YOUR_GIT_REPO_URL
  // highlight-end
    targetRevision: main
    ref: values
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

</details>
<br/>

3.使用 `kubectl` 配置应用程序清单: 

```bash
kubectl apply -f my-ocean-opsgenie-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Opsgenie integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />

<br/>

以下是 `opsgenie-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Opsgenie Exporter Workflow

# This workflow responsible for running Opsgenie exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Opsgenie Integration
        run: |
          # Set Docker image and run the container
          integration_type="opsgenie"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__API_TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__API_TOKEN }} \
          -e OCEAN__INTEGRATION__CONFIG__API_URL=${{ secrets.OCEAN__INTEGRATION__CONFIG__API_URL }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Opsgenie integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/)的 "Secret Text "类型: 

<DockerParameters />

<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

```text showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run Opsgenie Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__API_TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__API_TOKEN'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__API_URL', variable: 'OCEAN__INTEGRATION__CONFIG__API_URL'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="opsgenie"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__API_TOKEN=$OCEAN__INTEGRATION__CONFIG__API_TOKEN \
                                -e OCEAN__INTEGRATION__CONFIG__API_URL=$OCEAN__INTEGRATION__CONFIG__API_URL \
                                -e OCEAN__PORT__CLIENT_ID=$OCEAN__PORT__CLIENT_ID \
                                -e OCEAN__PORT__CLIENT_SECRET=$OCEAN__PORT__CLIENT_SECRET \
                                $image_name

                            exit $?
                        ''')
                    }
                }
            }
        }
    }
}
```

  </TabItem>

  <TabItem value="azure" label="Azure Devops">
<AzurePremise name="Opsgenie" />

<DockerParameters />

<br/>

下面是 `opsgenie-integration.yml` Pipelines 文件的示例: 

```yaml showLineNumbers
trigger:
- main

pool:
  vmImage: "ubuntu-latest"

variables:
  - group: port-ocean-credentials

steps:
- script: |
    # Set Docker image and run the container
    integration_type="opsgenie"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm --platform=linux/amd64 \
      -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
      -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
      -e OCEAN__INTEGRATION__CONFIG__API_TOKEN=${OCEAN__INTEGRATION__CONFIG__API_TOKEN} \
      -e OCEAN__INTEGRATION__CONFIG__API_URL=${OCEAN__INTEGRATION__CONFIG__API_URL} \
      -e OCEAN__PORT__CLIENT_ID=${OCEAN__PORT__CLIENT_ID} \
      -e OCEAN__PORT__CLIENT_SECRET=${OCEAN__PORT__CLIENT_SECRET} \
      $image_name

    exit $?
  displayName: 'Ingest Data into Port'
```

  </TabItem>

  </Tabs>
</TabItem>

</Tabs>

<AdvancedConfig/>

## 摄取 Opsgenie 对象

Opsgenie 集成使用 YAML 配置来描述将数据加载到开发人员门户的过程。 请参阅下面的[examples](#examples) 。

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 Opsgenie API 事件的现有字段和 Values 进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了从 Opsgenie 查询哪些资源，以及在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被引用来映射来自 Opsgenie 的数据，可以引用下面链接的 API 响应中出现的任何字段来进行映射配置。

* * [`Alert`](https://docs.opsgenie.com/docs/alert-api#list-alerts)
* [`Service`](https://docs.opsgenie.com/docs/service-api#list-services)
* [`Incident`](https://docs.opsgenie.com/docs/incident-api#list-incidents)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: service
      selector:
      ...
  ```


* 类型 "键是 Opsgenie 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: service
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: service
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
      # highlight-end
      port:
  ```


* Port"、"实体 "和 "映射 "键被用来将 Opsgenie 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: service
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one Opsgenie object to a Port entity. Each value is a JQ query.
            identifier: .id
            title: .name
            blueprint: '"opsGenieService"'
            properties:
              description: .description
        # highlight-end
    - kind: service # In this instance service is mapped again with a different filter
      selector:
        query: '.name == "MyServiceName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`) 
::: 

## 配置实时更新

目前，OpsGenie API 不支持编程式 webhook 创建。 要在 OpsGenie 中设置 webhook 配置，以便向 Ocean 集成发送警报通知，请按照以下步骤操作: 

### 前提条件

使用以下格式准备 webhook `URL`: `{app_host}/integration/webhook`。 `app_host` 参数应与部署集成的 ingress 或外部负载平衡器匹配。例如，如果您的 ingress 或负载平衡器在 `https://myservice.domain.com` 公开 OpsGenie Ocean 集成，则您的 webhook `URL` 应为 `https://myservice.domain.com/integration/webhook`。

#### 在 OpsGenie 中创建 webhook

1. 转到 OpsGenie；
2. 选择 **设置**；
3. 点击侧边栏**综合**部分下的**综合**；
4. 点击**添加集成**；
5. 在搜索框中输入 _Webhook_ 并选择 webhook 选项；
6. 输入以下详细信息: 
    1. 名称"--请被引用一个有意义的名称，如 Port Ocean Webhook；
    2.确保选中 "已启用 "复选框；
    3.选中 "向有效负载添加警报描述 "复选框；
    4.选中 "在有效载荷中添加警报详细信息 "复选框；
    5.点击**添加新操作**，将以下操作触发器添加到 webhook: 
        1.如果在 Opsgenie 中_警报被忽略_，则在 Webhook 中_发布到 url_；
        2.如果 _alert 的 description 在 Opsgenie 中被更新_, _post to url_ 在 Webhook 中；
        3.如果在 Opsgenie 中更新了_警报的信息_，则在 Webhook 中_发布到 url_；
        4.如果在 Opsgenie 中更新了_警报的优先级_，则在 Webhook 中_发布到 url_；
        5.如果_在 Opsgenie 中为警报添加了应答器_，_在 Webhook 中发布到 url_；
        6. 如果_一个用户在 Opsgenie 中执行 "分配所有权_，_发布到 Webhook 中的 url_；
        7. 如果_在 Opsgenie 中为警报添加了标签_，_在 Webhook 中发布到 url_；
        8. .如果在 Opsgenie 中从警报中删除_标签，则在 Webhook 中_发布到 url_；
    6.Webhook URL` - 输入上文创建的 `URL` 的值。
7.点击 **保存集成**

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Opsgenie 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 Opsgenie 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在事件管理类别下选择 Opsgenie。
5. 根据需要修改[configuration](#configuration-structure) 。
6. 单击 "Resync"。

## 示例

蓝图和相关集成配置示例: 

### 服务

<details>
<summary>Service blueprint</summary>

```json showLineNumbers
{
  "identifier": "opsGenieService",
  "description": "This blueprint represents an OpsGenie service in our software catalog",
  "title": "OpsGenie Service",
  "icon": "OpsGenie",
  "schema": {
    "properties": {
      "description": {
        "type": "string",
        "title": "Description",
        "icon": "DefaultProperty"
      },
      "url": {
        "title": "URL",
        "type": "string",
        "description": "URL to the service",
        "format": "url",
        "icon": "DefaultProperty"
      },
      "tags": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "title": "Tags",
        "icon": "DefaultProperty"
      },
      "oncallTeam": {
        "type": "string",
        "title": "OnCall Team",
        "description": "Name of the team responsible for this service",
        "icon": "DefaultProperty"
      },
      "teamMembers": {
        "icon": "TwoUsers",
        "type": "array",
        "items": {
          "type": "string",
          "format": "user"
        },
        "title": "Team Members",
        "description": "Members of team responsible for this service"
      },
      "oncallUsers": {
        "icon": "TwoUsers",
        "type": "array",
        "items": {
          "type": "string",
          "format": "user"
        },
        "title": "Oncall Users",
        "description": "Who is on call for this service"
      },
      "numOpenIncidents": {
        "title": "Number of Open Incidents",
        "type": "number"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {
    "teamSize": {
      "title": "Team Size",
      "icon": "DefaultProperty",
      "description": "Size of the team",
      "calculation": ".properties.teamMembers | length",
      "type": "number"
    }
  },
  "relations": {}
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
resources:
  - kind: service
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .name | gsub("[^a-zA-Z0-9@_.:/=-]"; "-") | tostring
          title: .name
          blueprint: '"opsGenieService"'
          properties:
            description: .description
            url: .links.web
            tags: .tags
            oncallTeam: .__team.name
            teamMembers: "[.__team.members[].user.username]"
            oncallUsers: .__oncalls.onCallRecipients
            numOpenIncidents: '[ .__incidents[] | select(.status == "open")] | length'
```

</details>

###事件

<details>
<summary>Incident blueprint</summary>

```json showLineNumbers
{
  "identifier": "opsGenieIncident",
  "description": "This blueprint represents an OpsGenie incident in our software catalog",
  "title": "OpsGenie Incident",
  "icon": "OpsGenie",
  "schema": {
    "properties": {
      "description": {
        "title": "Description",
        "type": "string"
      },
      "status": {
        "type": "string",
        "title": "Status",
        "enum": ["closed", "open", "resolved"],
        "enumColors": {
          "closed": "blue",
          "open": "red",
          "resolved": "green"
        },
        "description": "The status of the incident"
      },
      "url": {
        "type": "string",
        "format": "url",
        "title": "URL"
      },
      "tags": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "title": "Tags"
      },
      "responders": {
        "type": "array",
        "title": "Responders",
        "description": "Responders to the alert"
      },
      "priority": {
        "type": "string",
        "title": "Priority"
      },
      "createdAt": {
        "title": "Create At",
        "type": "string",
        "format": "date-time"
      },
      "updatedAt": {
        "title": "Updated At",
        "type": "string",
        "format": "date-time"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "services": {
      "title": "Impacted Services",
      "target": "opsGenieService",
      "many": true,
      "required": false
    }
  }
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
resources:
  - kind: incident
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id
          title: .message
          blueprint: '"opsGenieIncident"'
          properties:
            status: .status
            responders: .responders
            priority: .priority
            tags: .tags
            url: .links.web
            createdAt: .createdAt
            updatedAt: .updatedAt
            description: .description
          relations:
            services: '[.__impactedServices[] | .name | gsub("[^a-zA-Z0-9@_.:/=-]"; "-") | tostring]'
```

</details>

#### 警报

<details>
<summary>Alert blueprint</summary>

```json showLineNumbers
{
  "identifier": "opsGenieAlert",
  "description": "This blueprint represents an OpsGenie alert in our software catalog",
  "title": "OpsGenie Alert",
  "icon": "OpsGenie",
  "schema": {
    "properties": {
      "description": {
        "title": "Description",
        "type": "string"
      },
      "status": {
        "type": "string",
        "title": "Status",
        "enum": ["closed", "open"],
        "enumColors": {
          "closed": "green",
          "open": "red"
        },
        "description": "The status of the alert"
      },
      "acknowledged": {
        "type": "boolean",
        "title": "Acknowledged"
      },
      "tags": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "title": "Tags"
      },
      "responders": {
        "type": "array",
        "title": "Responders",
        "description": "Responders to the alert"
      },
      "integration": {
        "type": "string",
        "title": "Integration",
        "description": "The name of the Integration"
      },
      "priority": {
        "type": "string",
        "title": "Priority"
      },
      "sourceName": {
        "type": "string",
        "title": "Source Name",
        "description": "Alert source name"
      },
      "createdBy": {
        "title": "Created By",
        "type": "string",
        "format": "user"
      },
      "createdAt": {
        "title": "Create At",
        "type": "string",
        "format": "date-time"
      },
      "updatedAt": {
        "title": "Updated At",
        "type": "string",
        "format": "date-time"
      },
      "count": {
        "title": "Count",
        "type": "number"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "relatedIncident": {
      "title": "Related Incident",
      "target": "opsGenieIncident",
      "required": false,
      "many": false
    }
  }
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
resources:
  - kind: alert
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id
          title: .message
          blueprint: '"opsGenieAlert"'
          properties:
            status: .status
            acknowledged: .acknowledged
            responders: .responders
            priority: .priority
            sourceName: .source
            tags: .tags
            count: .count
            createdBy: .owner
            createdAt: .createdAt
            updatedAt: .updatedAt
            description: .description
            integration: .integration.name
          relations:
            relatedIncident: .__relatedIncident.id
```

</details>

## 让我们来测试一下

本节包括来自 Opsgenie 的响应数据示例。 此外，还包括根据上一节提供的 Ocean 配置从重新同步事件中创建的实体。

### 有效载荷

下面是 Opsgenie 提供的有效载荷结构示例: 

<details>
<summary> Service response data</summary>

```json showLineNumbers
{
  "id": "daa0d66f-ad35-4396-b30d-70f0314c697a",
  "name": "Port Outbound Service",
  "description": "For outbound communications and integrations",
  "teamId": "63374eee-0b03-42d4-bb8c-50d1fa64827c",
  "tags": ["communication", "channel"],
  "links": {
    "web": "https://mytestaccount.app.opsgenie.com/service/daa0d66f-ad35-4396-b30d-70f0314c697a/status",
    "api": "https://api.opsgenie.com/v1/services/daa0d66f-ad35-4396-b30d-70f0314c697a"
  },
  "isExternal": false,
  "__team": {
    "id": "63374eee-0b03-42d4-bb8c-50d1fa64827c",
    "name": "Data Science Team",
    "description": "",
    "members": [
      {
        "user": {
          "id": "ce544b61-7b35-43ea-89ee-a8750638d3a4",
          "username": "testuser@gmail.com"
        },
        "role": "admin"
      },
      {
        "user": {
          "id": "9ea8f86a-6648-46d6-a2fc-1a6eb5f739c8",
          "username": "devtester@gmail.com"
        },
        "role": "user"
      }
    ],
    "links": {
      "web": "https://app.opsgenie.com/teams/dashboard/63374eee-0b03-42d4-bb8c-50d1fa64827c/main",
      "api": "https://api.opsgenie.com/v2/teams/63374eee-0b03-42d4-bb8c-50d1fa64827c"
    }
  },
  "__incidents": [
    {
      "id": "652f14b3-019a-4d4c-8b83-e4da527a416c",
      "description": "summary",
      "impactedServices": ["daa0d66f-ad35-4396-b30d-70f0314c697a"],
      "tinyId": "3",
      "message": "OpenAI Incident",
      "status": "open",
      "tags": ["tags"],
      "createdAt": "2023-09-26T17:06:16.824Z",
      "updatedAt": "2023-09-26T17:49:10.17Z",
      "priority": "P3",
      "ownerTeam": "",
      "responders": [
        {
          "type": "team",
          "id": "63374eee-0b03-42d4-bb8c-50d1fa64827c"
        },
        {
          "type": "user",
          "id": "ce544b61-7b35-43ea-89ee-a8750638d3a4"
        }
      ],
      "extraProperties": {},
      "links": {
        "web": "https://mytestaccount.app.opsgenie.com/incident/detail/652f14b3-019a-4d4c-8b83-e4da527a416c",
        "api": "https://api.opsgenie.com/v1/incidents/652f14b3-019a-4d4c-8b83-e4da527a416c"
      },
      "impactStartDate": "2023-09-26T17:06:16.824Z",
      "impactEndDate": "2023-09-26T17:45:25.719Z",
      "actions": []
    },
    {
      "id": "4a0c5e6d-b239-4cd9-a7e8-a5f880e99473",
      "description": "descirption",
      "impactedServices": [
        "59591948-d418-4bb9-af16-170d6b232b7d",
        "daa0d66f-ad35-4396-b30d-70f0314c697a"
      ],
      "tinyId": "2",
      "message": "My Incident",
      "status": "open",
      "tags": ["hello"],
      "createdAt": "2023-09-20T13:33:00.941Z",
      "updatedAt": "2023-09-26T17:48:54.48Z",
      "priority": "P3",
      "ownerTeam": "",
      "responders": [
        {
          "type": "team",
          "id": "63374eee-0b03-42d4-bb8c-50d1fa64827c"
        },
        {
          "type": "user",
          "id": "ce544b61-7b35-43ea-89ee-a8750638d3a4"
        },
        {
          "type": "user",
          "id": "9ea8f86a-6648-46d6-a2fc-1a6eb5f739c8"
        }
      ],
      "extraProperties": {},
      "links": {
        "web": "https://mytestaccount.app.opsgenie.com/incident/detail/4a0c5e6d-b239-4cd9-a7e8-a5f880e99473",
        "api": "https://api.opsgenie.com/v1/incidents/4a0c5e6d-b239-4cd9-a7e8-a5f880e99473"
      },
      "impactStartDate": "2023-09-20T13:33:00.941Z",
      "actions": []
    }
  ],
  "__oncalls": {
    "_parent": {
      "id": "d55e148e-d320-4766-9dcf-4fc2ce9daef1",
      "name": "Data Science Team_schedule",
      "enabled": true
    },
    "onCallRecipients": ["testuser@gmail.com"]
  }
}
```

</details>

<details>
<summary> Incident response data</summary>

```json showLineNumbers
{
  "id": "4a0c5e6d-b239-4cd9-a7e8-a5f880e99473",
  "description": "descirption",
  "impactedServices": [
    "59591948-d418-4bb9-af16-170d6b232b7d",
    "daa0d66f-ad35-4396-b30d-70f0314c697a"
  ],
  "tinyId": "2",
  "message": "My Incident",
  "status": "open",
  "tags": ["hello"],
  "createdAt": "2023-09-20T13:33:00.941Z",
  "updatedAt": "2023-09-26T17:48:54.48Z",
  "priority": "P3",
  "ownerTeam": "",
  "responders": [
    {
      "type": "team",
      "id": "63374eee-0b03-42d4-bb8c-50d1fa64827c"
    },
    {
      "type": "user",
      "id": "ce544b61-7b35-43ea-89ee-a8750638d3a4"
    },
    {
      "type": "user",
      "id": "9ea8f86a-6648-46d6-a2fc-1a6eb5f739c8"
    }
  ],
  "extraProperties": {},
  "links": {
    "web": "https://mytestaccount.app.opsgenie.com/incident/detail/4a0c5e6d-b239-4cd9-a7e8-a5f880e99473",
    "api": "https://api.opsgenie.com/v1/incidents/4a0c5e6d-b239-4cd9-a7e8-a5f880e99473"
  },
  "impactStartDate": "2023-09-20T13:33:00.941Z",
  "actions": [],
  "__impactedServices": [
    {
      "id": "59591948-d418-4bb9-af16-170d6b232b7d",
      "name": "My Test Service",
      "description": "This is for Opsgenie testing",
      "teamId": "63374eee-0b03-42d4-bb8c-50d1fa64827c",
      "tags": ["port", "devops", "ai"],
      "links": {
        "web": "https://mytestaccount.app.opsgenie.com/service/59591948-d418-4bb9-af16-170d6b232b7d/status",
        "api": "https://api.opsgenie.com/v1/services/59591948-d418-4bb9-af16-170d6b232b7d"
      },
      "isExternal": false
    },
    {
      "id": "daa0d66f-ad35-4396-b30d-70f0314c697a",
      "name": "Port Outbound Service",
      "description": "For outbound communications and integrations",
      "teamId": "63374eee-0b03-42d4-bb8c-50d1fa64827c",
      "tags": ["ui", "comment"],
      "links": {
        "web": "https://mytestaccount.app.opsgenie.com/service/daa0d66f-ad35-4396-b30d-70f0314c697a/status",
        "api": "https://api.opsgenie.com/v1/services/daa0d66f-ad35-4396-b30d-70f0314c697a"
      },
      "isExternal": false
    }
  ]
}
```

</details>

<details>
<summary> Alert response data</summary>

```json showLineNumbers
{
  "seen": true,
  "id": "580e9625-ecc7-42b0-8836-3d3637438bc4-169486700232",
  "tinyId": "2",
  "alias": "355f1681-f6e3-4355-a927-897cd8edaf8f_4d37bc8a-030b-4e8f-a230-aa429947253c",
  "message": "Login Auth not working",
  "status": "open",
  "integration": "Default API",
  "acknowledged": false,
  "isSeen": true,
  "tags": ["auth", "login"],
  "snoozed": false,
  "count": 1,
  "lastOccurredAt": "2023-04-21T08:36:18.46Z",
  "createdAt": "2023-04-21T08:36:18.46Z",
  "updatedAt": "2023-08-11T10:41:16.533Z",
  "source": "testuser@gmail.com",
  "owner": "testuser@gmail.com",
  "priority": "P3",
  "teams": [],
  "responders": [
    {
      "type": "user",
      "id": "ce544b61-7b35-43ea-89ee-a8750638d3a4"
    }
  ],
  "report": {
    "ackTime": 4852912663
  },
  "ownerTeamId": "",
  "__relatedIncident": {
    "id": "4a0c5e6d-b239-4cd9-a7e8-a5f880e99473",
    "description": "descirption",
    "impactedServices": [
      "59591948-d418-4bb9-af16-170d6b232b7d",
      "daa0d66f-ad35-4396-b30d-70f0314c697a"
    ],
    "tinyId": "2",
    "message": "My Incident",
    "status": "open",
    "tags": ["hello"],
    "createdAt": "2023-09-20T13:33:00.941Z",
    "updatedAt": "2023-09-26T17:48:54.48Z",
    "priority": "P3",
    "ownerTeam": "",
    "responders": [
      {
        "type": "team",
        "id": "63374eee-0b03-42d4-bb8c-50d1fa64827c"
      },
      {
        "type": "user",
        "id": "ce544b61-7b35-43ea-89ee-a8750638d3a4"
      },
      {
        "type": "user",
        "id": "9ea8f86a-6648-46d6-a2fc-1a6eb5f739c8"
      }
    ],
    "extraProperties": {},
    "links": {
      "web": "https://mytestaccount.app.opsgenie.com/incident/detail/4a0c5e6d-b239-4cd9-a7e8-a5f880e99473",
      "api": "https://api.opsgenie.com/v1/incidents/4a0c5e6d-b239-4cd9-a7e8-a5f880e99473"
    }
  }
}
```

</details>

#### 映射结果

结合样本有效载荷和 Ocean 配置，可生成以下 Port 实体: 

<details>
<summary> Service entity in Port</summary>

```json showLineNumbers
{
  "identifier": "Port-Outbound-Service",
  "title": "Port Outbound Service",
  "team": [],
  "properties": {
    "description": "For outbound communications and integrations",
    "url": "https://mytestaccount.app.opsgenie.com/service/daa0d66f-ad35-4396-b30d-70f0314c697a/status",
    "tags": ["communication", "channel"],
    "oncallTeam": "Data Science Team",
    "teamMembers": ["testuser@gmail.com", "devtester@gmail.com"],
    "oncallUsers": ["testuser@gmail.com"],
    "numOpenIncidents": 2
  },
  "relations": {},
  "icon": "OpsGenie"
}
```

</details>

<details>
<summary> Incident entity in Port</summary>

```json showLineNumbers
{
  "identifier": "4a0c5e6d-b239-4cd9-a7e8-a5f880e99473",
  "blueprint": "opsGenieIncident",
  "title": "My Incident",
  "properties": {
    "status": "open",
    "responders": [
      {
        "type": "team",
        "id": "63374eee-0b03-42d4-bb8c-50d1fa64827c"
      },
      {
        "type": "user",
        "id": "ce544b61-7b35-43ea-89ee-a8750638d3a4"
      },
      {
        "type": "user",
        "id": "9ea8f86a-6648-46d6-a2fc-1a6eb5f739c8"
      }
    ],
    "priority": "P3",
    "tags": ["hello"],
    "url": "https://mytestaccount.app.opsgenie.com/incident/detail/4a0c5e6d-b239-4cd9-a7e8-a5f880e99473",
    "createdAt": "2023-09-20T13:33:00.941Z",
    "updatedAt": "2023-09-26T17:48:54.48Z",
    "description": "descirption"
  },
  "relations": {
    "services": ["Port-Outbound-Service", "My-Test-Service"]
  },
  "icon": "OpsGenie"
}
```

</details>

<details>
<summary> Alert entity in Port</summary>

```json showLineNumbers
{
  "identifier": "580e9625-ecc7-42b0-8836-3d3637438bc4-1694867002321",
  "title": "Login Auth not working",
  "team": [],
  "properties": {
    "status": "open",
    "acknowledged": true,
    "tags": ["auth", "login"],
    "responders": [
      {
        "type": "user",
        "id": "ce544b61-7b35-43ea-89ee-a8750638d3a4"
      }
    ],
    "integration": "Default API",
    "priority": "P3",
    "sourceName": "testuser@gmail.com",
    "createdBy": "testuser@gmail.com",
    "createdAt": "2023-09-16T12:23:22.321Z",
    "updatedAt": "2023-09-26T17:51:00.346Z",
    "count": 1
  },
  "relations": {
    "relatedIncident": "4a0c5e6d-b239-4cd9-a7e8-a5f880e99473"
  },
  "icon": "OpsGenie"
}
```

</details>

## 通过 webhook 进行替代安装

虽然上述 Ocean 集成是推荐的安装方法，但您可能更喜欢使用 webhook 从 Opsgenie 引用数据。 如果是这样，请使用以下说明: 

<details>

<summary><b>Webhook installation (click to expand)</b></summary>

在本示例中，您将在[OpsGenie](https://www.atlassian.com/software/opsgenie) 和 Port 之间创建一个 webhook 集成，用于接收警报实体。

###Port配置

创建以下蓝图定义: 

<details>
<summary>OpsGenie alert blueprint</summary>

<OpsGenieAlertBlueprint/>

</details>

创建以下 webhook 配置[using Port UI](../../?operation=ui#configuring-webhook-endpoints) : 

<details>
<summary>OpsGenie alert webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `OpsGenie mapper`；
    2.标识符 : `opsgenie_mapper`；
    3.Description : `将 OpsGenie 警报映射到 Port` 的 webhook 配置；
    4.图标 : `OpsGenie`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <OpsGenieAlertConfiguration/>
3.单击页面底部的**保存**。

</details>

<h2>Create a webhook in OpsGenie</h2>

1. 转到 OpsGenie；
2. 选择 **设置**；
3. 点击侧边栏**综合**部分下的**综合**；
4. 点击**添加集成**；
5. 在搜索框中输入 _Webhook_ 并选择 webhook 选项；
6. 输入以下详细信息: 
    1. 名称"- 请被引用一个有意义的名称，如 Port Webhook；
    2.确保选中 "已启用 "复选框；
    3.选中 "向有效负载添加警报描述 "复选框；
    4.选中 "在有效载荷中添加警报详细信息 "复选框；
    5.点击**添加新操作**，将以下操作触发器添加到 webhook: 
        1.如果在 Opsgenie 中_警报被忽略_，则在 Webhook 中_发布到 url_；
        2.如果 _alert 的 description 在 Opsgenie 中被更新_, _post to url_ 在 Webhook 中；
        3.如果在 Opsgenie 中更新了_警报的信息_，则在 Webhook 中_发布到 url_；
        4.如果在 Opsgenie 中更新了_警报的优先级_，则在 Webhook 中_发布到 url_；
        5.如果_在 Opsgenie 中为警报添加了应答器_，_在 Webhook 中发布到 url_；
        6. 如果_一个用户在 Opsgenie 中执行 "分配所有权_，_发布到 Webhook 中的 url_；
        7. 如果_在 Opsgenie 中为警报添加了标签_，_在 Webhook 中发布到 url_；
        8. .如果在 Opsgenie 中从警报中删除_标签，则在 Webhook 中_发布到 url_；
    6.Webhook URL` - 输入创建 Webhook 配置后收到的 `url` 键值；
7.点击**保存集成**

:::tip 为了查看 Opsgenie webhooks 中可用的不同有效载荷和事件、[look here](https://support.atlassian.com/opsgenie/docs/opsgenie-edge-connector-alert-action-data/)

:::

完成！OpsGenie 警报发生的任何变化(创建、确认等)都会触发 webhook 事件，OpsGenie 会将该事件发送到 Provider 提供的 webhook URL。 Port 会根据映射解析事件并相应地更新目录实体。

<h2>Let's Test It</h2>

本节包括创建警报时从 OpsGenie 发送的 webhook 事件示例。 此外，还包括根据上一节提供的 webhook 配置从事件中创建的实体。

<h3>Payload</h3>

下面是创建 OpsGenie 警报时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Webhook event payload</summary>

```json showLineNumbers
{
  "source": {
    "name": "web",
    "type": "API"
  },
  "alert": {
    "tags": ["tag1", "tag2"],
    "teams": ["team1", "team2"],
    "responders": ["recipient1", "recipient2"],
    "message": "test alert",
    "username": "username",
    "alertId": "052652ac-5d1c-464a-812a-7dd18bbfba8c",
    "source": "user@domain.com",
    "alias": "aliastest",
    "tinyId": "10",
    "entity": "An example entity",
    "createdAt": 1686916265415,
    "updatedAt": 1686916266116,
    "userId": "daed1180-0ce8-438b-8f8e-57e1a5920a2d",
    "description": "Testing opsgenie alerts",
    "priority": "P1"
  },
  "action": "Create",
  "integrationId": "37c8f316-17c6-49d7-899b-9c7e540c048d",
  "integrationName": "Port-Integration"
}
```

</details>

<h3>Mapping Result</h3>

结合示例有效载荷和 webhook 配置可生成以下 Port 实体: 

```json showLineNumbers
{
  "identifier": "052652ac-5d1c-464a-812a-7dd18bbfba8c",
  "title": "10 - test alert",
  "blueprint": "opsGenieAlert",
  "properties": {
    "description": "Testing opsgenie alerts",
    "lastChangeType": "Create",
    "priority": "P1",
    "sourceName": "web",
    "sourceType": "API",
    "tags": ["tag1", "tag2"],
    "responders": ["recipient1", "recipient2"],
    "teams": ["team1", "team2"]
  },
  "relations": {}
}
```

<h2>Ingest who is on-call</h2>

在本示例中，我们将为具有 "on-call "属性的 "service "实体创建一个蓝图，该蓝图将直接从 OpsGenie 采集数据。 下面的示例将使用 GitLab Pipelines 或 GitHub Workflows 在定义的计划时间内从 OpsGenie REST Api 提取数据，并将数据作为 "service "蓝图的一个属性报告给 Port。

* * [Github Workflow](https://github.com/port-labs/opsgenie-oncall-example)
* [GitLab CI Pipeline](https://gitlab.com/getport-labs/opsgenie-oncall-example)

</details>
