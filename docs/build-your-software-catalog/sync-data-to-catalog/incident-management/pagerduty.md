---
sidebar_position: 1
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import DockerParameters from "./_pagerduty_docker_params.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'
import PagerDutyServiceBlueprint from "../webhook/examples/resources/pagerduty/_example_pagerduty_service.mdx"
import PagerDutyIncidentBlueprint from "../webhook/examples/resources/pagerduty/_example_pagerduty_incident.mdx"
import PagerDutyWebhookConfig from "../webhook/examples/resources/pagerduty/_example_pagerduty_webhook_config.mdx"
import PagerDutyWebhookHistory from "../webhook/examples/resources/pagerduty/_example_pagerduty_webhook_history_config.mdx"
import PagerDutyScript from "../webhook/examples/resources/pagerduty/_example_pagerduty_shell_history_config.mdx"

# PagerDuty

我们的 PagerDuty 集成允许您根据映射和定义，将 PagerDuty 账户中的 "日程"、"服务 "和 "事件 "导入 Port。

## 常见被用于情况

* 映射 PagerDuty 组织环境中的 "日程"、"服务 "和 "事件"。
* 实时观察对象更改(创建/更新/删除)，并自动将更改应用到您的 Port 实体中。

## 先决条件

<Prerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                        | Description                                                                                                             | Required |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | -------- |
| `port.clientId`                  | Your port client id                                                                                                     | ✅       |
| `port.clientSecret`              | Your port client secret                                                                                                 | ✅       |
| `port.baseUrl`                   | Your port base url, relevant only if not using the default port app                                                     | ❌       |
| `integration.identifier`         | Change the identifier to describe your integration                                                                      | ✅       |
| `integration.type`               | The integration type                                                                                                    | ✅       |
| `integration.eventListener.type` | The event listener type                                                                                                 | ✅       |
| `integration.secrets.token`      | PagerDuty API token                                                                                                | ✅       |
| `integration.config.apiUrl`      | Pagerduty api url. If not specified, the default will be https://api.pagerduty.com                                      | ✅       |
| `integration.config.appHost`     | The host of the Port Ocean app. Used to set up the integration endpoint as the target for Webhooks created in PagerDuty | ❌       |
| `scheduledResyncInterval`        | The number of minutes between each resync                                                                               | ❌       |
| `initializePortResources`        | Default true, When set to true the integration will create default blueprints and the port App config Mapping           | ❌       |


<br/>

<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-pagerduty-integration port-labs/port-ocean \
  --set port.clientId="PORT_CLIENT_ID"  \
  --set port.clientSecret="PORT_CLIENT_SECRET"  \
  --set port.baseUrl="https://api.getport.io"  \
  --set initializePortResources=true  \
  --set scheduledResyncInterval=120  \
  --set integration.identifier="my-pagerduty-integration"  \
  --set integration.type="pagerduty"  \
  --set integration.eventListener.type="POLLING"  \
  --set integration.secrets.token="string"  \
  --set integration.config.apiUrl="string"
```

</TabItem>

<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在 git 仓库的`argocd/my-ocean-pagerduty-integration`中创建一个`values.yaml`文件，内容如下: 

:::note 请记住替换 `PAGERDUTY_API_URL` 和 `PAGERDUTY_API_TOKEN` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-pagerduty-integration
  type: pagerduty
  eventListener:
    type: POLLING
  config:
  // highlight-next-line
    apiUrl: PAGERDUTY_API_URL
  secrets:
  // highlight-next-line
    token: PAGERDUTY_API_TOKEN
```

<br/>

2.创建下面的 "my-ocean-pagerduty-integration.yaml "配置清单，安装 "my-ocean-pagerduty-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-pagerduty-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-pagerduty-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-pagerduty-integration/values.yaml
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
kubectl apply -f my-ocean-pagerduty-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the PagerDuty integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />

<br/>

下面是 "pagerduty-integration.yml "工作流文件的示例: 

```yaml showLineNumbers
name: PagerDuty Exporter Workflow

# This workflow responsible for running PagerDuty exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run PagerDuty Integration
        run: |
          # Set Docker image and run the container
          integration_type="pagerduty"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__TOKEN }} \
          -e OCEAN__INTEGRATION__CONFIG__API_URL=${{ secrets.OCEAN__INTEGRATION__CONFIG__API_URL }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the PagerDuty integration once and then exit, this is useful for **scheduled** ingestion of data.

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
        stage('Run PagerDuty Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__TOKEN'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__API_URL', variable: 'OCEAN__INTEGRATION__CONFIG__API_URL'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="pagerduty"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__TOKEN=$OCEAN__INTEGRATION__CONFIG__TOKEN \
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
<AzurePremise name="PagerDuty" />

<DockerParameters />

<br/>

下面是 "pagerduty-integration.yml "管道文件的示例: 

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
    integration_type="pagerduty"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm --platform=linux/amd64 \
        -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
        -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
        -e OCEAN__INTEGRATION__CONFIG__TOKEN=${OCEAN__INTEGRATION__CONFIG__TOKEN} \
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

## 接收 PagerDuty 对象

PagerDuty 集成使用 YAML 配置来描述将数据加载到开发人员门户的过程。 请参阅下面的[examples](#examples) 。

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 PagerDuty API 事件的现有字段和值进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定从 PagerDuty 查询哪些资源，以及在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被用于来映射来自 PagerDuty 的数据，可以引用下面链接的 API 响应中出现的任何字段来进行映射配置。

* * [`Schedule`](https://developer.pagerduty.com/api-reference/846ecf84402bb-list-schedules)
* [`Service`](https://developer.pagerduty.com/api-reference/e960cca205c0f-list-services)
* [`Incident`](https://developer.pagerduty.com/api-reference/9d0b4b12e36f9-list-incidents)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: services
      selector:
      ...
  ```


* 类型 "键是 PagerDuty 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: services
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: services
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
      # highlight-end
      port:
  ```


* Port"、"实体 "和 "映射 "键被用来将 PagerDuty 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: services
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one PagerDuty object to a Port entity. Each value is a JQ query.
            identifier: .id
            title: .name
            blueprint: '"pagerdutyService"'
            properties:
              status: .status
        # highlight-end
    - kind: service # In this instance service is mapped again with a different filter
      selector:
        query: '.name == "MyProjectName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`) 
::: 

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 PagerDuty 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 PagerDuty 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在事件管理类别下选择 PagerDuty。
5. 根据需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

### 时间表

<details>
<summary>Schedule blueprint</summary>

```json showLineNumbers
{
  "identifier": "pagerdutySchedule",
  "description": "This blueprint represents a PagerDuty schedule in our software catalog",
  "title": "PagerDuty Schedule",
  "icon": "pagerduty",
  "schema": {
    "properties": {
      "url": {
        "title": "Schedule URL",
        "type": "string",
        "format": "url"
      },
      "timezone": {
        "title": "Timezone",
        "type": "string"
      },
      "description": {
        "title": "Description",
        "type": "string"
      },
      "users": {
        "title": "Users",
        "type": "array"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {}
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: schedules
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id
          title: .name
          blueprint: '"pagerdutySchedule"'
          properties:
            url: .html_url
            timezone: .time_zone
            description: .description
            users: "[.users[].summary]"
```

</details>

### 服务

<details>
<summary>Service blueprint</summary>

```json showLineNumbers
{
  "identifier": "pagerdutyService",
  "description": "This blueprint represents a PagerDuty service in our software catalog",
  "title": "PagerDuty Service",
  "icon": "pagerduty",
  "schema": {
    "properties": {
      "status": {
        "title": "Status",
        "type": "string",
        "enum": [
          "active",
          "warning",
          "critical",
          "maintenance",
          "disabled"
        ],
        "enumColors": {
          "active": "green",
          "warning": "yellow",
          "critical": "red",
          "maintenance": "lightGray",
          "disabled": "darkGray"
        }
      },
      "url": {
        "title": "URL",
        "type": "string",
        "format": "url"
      },
      "oncall": {
        "title": "On Call",
        "type": "array",
        "items": {
          "type": "string",
          "format": "user"
        }
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: services
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id
          title: .name
          blueprint: '"pagerdutyService"'
          properties:
            status: .status
            url: .html_url
            oncall: "[.__oncall_user[].user.email]"
```

</details>

###事件

<details>
<summary>Incident blueprint</summary>

```json showLineNumbers
{
  "identifier": "pagerdutyIncident",
  "description": "This blueprint represents a PagerDuty incident in our software catalog",
  "title": "PagerDuty Incident",
  "icon": "pagerduty",
  "schema": {
    "properties": {
      "status": {
        "type": "string",
        "title": "Incident Status",
        "enum": [
          "triggered",
          "annotated",
          "acknowledged",
          "reassigned",
          "escalated",
          "reopened",
          "resolved"
        ]
      },
      "url": {
        "type": "string",
        "format": "url",
        "title": "Incident URL"
      },
      "urgency": {
        "type": "string",
        "title": "Incident Urgency",
        "enum": ["high", "low"]
      },
      "responder": {
        "type": "string",
        "title": "Assignee"
      },
      "escalation_policy": {
        "type": "string",
        "title": "Escalation Policy"
      },
      "created_at": {
        "title": "Create At",
        "type": "string",
        "format": "date-time"
      },
      "updated_at": {
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
    "pagerdutyService": {
      "title": "PagerDuty Service",
      "target": "pagerdutyService",
      "required": false,
      "many": true
    }
  }
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
resources:
  - kind: incidents
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id | tostring
          title: .title
          blueprint: '"pagerdutyIncident"'
          properties:
            status: .status
            url: .self
            urgency: .urgency
            responder: .assignments[0].assignee.summary
            escalation_policy: .escalation_policy.summary
            created_at: .created_at
            updated_at: .updated_at
          relations:
            pagerdutyService: .service.id
```

</details>

## 收录事件分析

要利用分析数据丰富 PagerDuty 事件实体，请按照以下步骤操作: 

1. 更新事件蓝图以包含 "分析 "属性。
    <details>

    <summary>Updated incident blueprint</summary>

    ```json showLineNumbers
    {
      "identifier": "pagerdutyIncident",
      "description": "This blueprint represents a PagerDuty incident in our software catalog",
      "title": "PagerDuty Incident",
      "icon": "pagerduty",
      "schema": {
        "properties": {
          "status": {
            "type": "string",
            "title": "Incident Status",
            "enum": [
              "triggered",
              "annotated",
              "acknowledged",
              "reassigned",
              "escalated",
              "reopened",
              "resolved"
            ]
          },
          "url": {
            "type": "string",
            "format": "url",
            "title": "Incident URL"
          },
          "urgency": {
            "type": "string",
            "title": "Incident Urgency",
            "enum": ["high", "low"]
          },
          "responder": {
            "type": "string",
            "title": "Assignee"
          },
          "escalation_policy": {
            "type": "string",
            "title": "Escalation Policy"
          },
          "created_at": {
            "title": "Create At",
            "type": "string",
            "format": "date-time"
          },
          "updated_at": {
            "title": "Updated At",
            "type": "string",
            "format": "date-time"
          },
          # highlight-start
          "analytics": {
            "title": "Analytics",
            "type": "object"
          }
          # highlight-end
        },
        "required": []
      },
      "mirrorProperties": {},
      "calculationProperties": {},
      "relations": {
        "pagerdutyService": {
          "title": "PagerDuty Service",
          "target": "pagerdutyService",
          "required": false,
          "many": true
        }
      }
    }
    ```

   </details>

2. 在集成的 `selector` 键中添加 `incidentAnalytics` 属性。设置为 `true`时，集成将从[PagerDuty Analytics API](https://developer.pagerduty.com/api-reference/328d94baeaa0e-get-raw-data-single-incident) 获取数据并摄取到 Port。默认情况下，此属性设置为 `false`。


    ```yaml showLineNumbers
    resources:
      - kind: incidents
        selector:
          query: "true"
          # highlight-next-line
          incidentAnalytics: "true"
        port:
          entity:
            mappings:
              identifier: .id | tostring
              title: .title
              blueprint: '"pagerdutyIncident"'
              properties:
                status: .status
                url: .self
    ```


3. 在 "分析 "蓝图属性和分析数据响应之间建立映射。


    ```yaml showLineNumbers
    resources:
      - kind: incidents
        selector:
          query: "true"
          incidentAnalytics: "true"
        port:
          entity:
            mappings:
              identifier: .id | tostring
              title: .title
              blueprint: '"pagerdutyIncident"'
              properties:
                status: .status
                url: .self
                urgency: .urgency
                responder: .assignments[0].assignee.summary
                escalation_policy: .escalation_policy.summary
                created_at: .created_at
                updated_at: .updated_at
                # highlight-next-line
                analytics: .__analytics
              relations:
                pagerdutyService: .service.id
    ```

4. Below is the complete integration configuration for enriching the incident blueprint with analytics data.

<details>
<summary>Incident analytics integration configuration</summary>


    ```yaml showLineNumbers
    resources:
    - kind: incidents
      selector:
        query: "true"
        incidentAnalytics: "true"
      port:
        entity:
          mappings:
            identifier: .id | tostring
            title: .title
            blueprint: '"pagerdutyIncident"'
            properties:
              status: .status
              url: .self
              urgency: .urgency
              responder: .assignments[0].assignee.summary
              escalation_policy: .escalation_policy.summary
              created_at: .created_at
              updated_at: .updated_at
              analytics: .__analytics
            relations:
              pagerdutyService: .service.id
    ```

</details>

## 让我们来测试一下

本节包括来自 Providerduty 的响应数据示例，此外还包括根据上一节提供的 Ocean 配置从重新同步事件中创建的实体。

### 有效载荷

下面是 Pagerduty 的有效载荷结构示例: 

<details>
<summary> Schedule response data</summary>

```json showLineNumbers
{
  "id": "PWAXLIH",
  "type": "schedule",
  "summary": "Port Test Service - Weekly Rotation",
  "self": "https://api.pagerduty.com/schedules/PWAXLIH",
  "html_url": "https://getport-io.pagerduty.com/schedules/PWAXLIH",
  "name": "Port Test Service - Weekly Rotation",
  "time_zone": "Asia/Jerusalem",
  "description": "This is the weekly on call schedule for Port Test Service associated with your first escalation policy.",
  "users": [
    {
      "id": "PJCRRLH",
      "type": "user_reference",
      "summary": "Adam",
      "self": "https://api.pagerduty.com/users/PJCRRLH",
      "html_url": "https://getport-io.pagerduty.com/users/PJCRRLH"
    },
    {
      "id": "P4K4DLP",
      "type": "user_reference",
      "summary": "Alice",
      "self": "https://api.pagerduty.com/users/P4K4DLP",
      "html_url": "https://getport-io.pagerduty.com/users/P4K4DLP"
    },
    {
      "id": "PFZ63E2",
      "type": "user_reference",
      "summary": "Doe",
      "self": "https://api.pagerduty.com/users/PFZ63E2",
      "html_url": "https://getport-io.pagerduty.com/users/PFZ63E2"
    },
    {
      "id": "PRGAUI4",
      "type": "user_reference",
      "summary": "Pages",
      "self": null,
      "html_url": "https://getport-io.pagerduty.com/users/PRGAUI4",
      "deleted_at": "2023-10-17T18:58:07+03:00"
    },
    {
      "id": "PYIEKLY",
      "type": "user_reference",
      "summary": "Demo",
      "self": "https://api.pagerduty.com/users/PYIEKLY",
      "html_url": "https://getport-io.pagerduty.com/users/PYIEKLY"
    }
  ],
  "escalation_policies": [
    {
      "id": "P7LVMYP",
      "type": "escalation_policy_reference",
      "summary": "Test Escalation Policy",
      "self": "https://api.pagerduty.com/escalation_policies/P7LVMYP",
      "html_url": "https://getport-io.pagerduty.com/escalation_policies/P7LVMYP"
    }
  ],
  "teams": []
}
```

</details>

<details>
<summary> Service response data</summary>

```json showLineNumbers
{
  "id": "PGAAJBE",
  "name": "My Test Service",
  "description": "For testing",
  "created_at": "2023-08-03T16:53:48+03:00",
  "updated_at": "2023-08-03T16:53:48+03:00",
  "status": "active",
  "teams": [],
  "alert_creation": "create_alerts_and_incidents",
  "addons": [],
  "scheduled_actions": [],
  "support_hours": "None",
  "last_incident_timestamp": "None",
  "escalation_policy": {
    "id": "P7LVMYP",
    "type": "escalation_policy_reference",
    "summary": "Test Escalation Policy",
    "self": "https://api.pagerduty.com/escalation_policies/P7LVMYP",
    "html_url": "https://getport-io.pagerduty.com/escalation_policies/P7LVMYP"
  },
  "incident_urgency_rule": {
    "type": "constant",
    "urgency": "high"
  },
  "acknowledgement_timeout": "None",
  "auto_resolve_timeout": "None",
  "integrations": [],
  "type": "service",
  "summary": "My Test Service",
  "self": "https://api.pagerduty.com/services/PGAAJBE",
  "html_url": "https://getport-io.pagerduty.com/service-directory/PGAAJBE",
  "__oncall_user": [
    {
      "escalation_policy": {
        "id": "P7LVMYP",
        "type": "escalation_policy_reference",
        "summary": "Test Escalation Policy",
        "self": "https://api.pagerduty.com/escalation_policies/P7LVMYP",
        "html_url": "https://getport-io.pagerduty.com/escalation_policies/P7LVMYP"
      },
      "escalation_level": 1,
      "schedule": {
        "id": "PWAXLIH",
        "type": "schedule_reference",
        "summary": "Port Test Service - Weekly Rotation",
        "self": "https://api.pagerduty.com/schedules/PWAXLIH",
        "html_url": "https://getport-io.pagerduty.com/schedules/PWAXLIH"
      },
      "user": {
        "name": "demo",
        "email": "devops-port@pager-demo.com",
        "time_zone": "Asia/Jerusalem",
        "color": "teal",
        "avatar_url": "https://secure.gravatar.com/avatar/5cc831a4e778f54460efc4cd20d13acd.png?d=mm&r=PG",
        "billed": true,
        "role": "admin",
        "description": "None",
        "invitation_sent": true,
        "job_title": "None",
        "teams": [],
        "contact_methods": [
          {
            "id": "POKPUFD",
            "type": "email_contact_method_reference",
            "summary": "Default",
            "self": "https://api.pagerduty.com/users/PYIEKLY/contact_methods/POKPUFD",
            "html_url": "None"
          }
        ],
        "notification_rules": [
          {
            "id": "P9NWEKF",
            "type": "assignment_notification_rule_reference",
            "summary": "0 minutes: channel POKPUFD",
            "self": "https://api.pagerduty.com/users/PYIEKLY/notification_rules/P9NWEKF",
            "html_url": "None"
          },
          {
            "id": "PPJHFA5",
            "type": "assignment_notification_rule_reference",
            "summary": "0 minutes: channel POKPUFD",
            "self": "https://api.pagerduty.com/users/PYIEKLY/notification_rules/PPJHFA5",
            "html_url": "None"
          }
        ],
        "id": "PYIEKLY",
        "type": "user",
        "summary": "demo",
        "self": "https://api.pagerduty.com/users/PYIEKLY",
        "html_url": "https://getport-io.pagerduty.com/users/PYIEKLY"
      },
      "start": "2023-10-17T15:57:50Z",
      "end": "2024-02-13T22:16:48Z"
    }
  ]
}
```

</details>

<details>
<summary> Incident response data</summary>

```json showLineNumbers
{
  "incident_number": 2,
  "title": "Example Incident",
  "description": "Example Incident",
  "created_at": "2023-05-15T13:59:45Z",
  "updated_at": "2023-05-15T13:59:45Z",
  "status": "triggered",
  "incident_key": "89809d37f4344d36a90c0a192c20c617",
  "service": {
    "id": "PWJAGSD",
    "type": "service_reference",
    "summary": "Port Test Service",
    "self": "https://api.pagerduty.com/services/PWJAGSD",
    "html_url": "https://getport-io.pagerduty.com/service-directory/PWJAGSD"
  },
  "assignments": [
    {
      "at": "2023-05-15T13:59:45Z",
      "assignee": {
        "id": "PJCRRLH",
        "type": "user_reference",
        "summary": "Username",
        "self": "https://api.pagerduty.com/users/PJCRRLH",
        "html_url": "https://getport-io.pagerduty.com/users/PJCRRLH"
      }
    }
  ],
  "assigned_via": "escalation_policy",
  "last_status_change_at": "2023-05-15T13:59:45Z",
  "resolved_at": null,
  "first_trigger_log_entry": {
    "id": "R5S5T07QR1SZRQFYB7SXEO2EKZ",
    "type": "trigger_log_entry_reference",
    "summary": "Triggered through the website.",
    "self": "https://api.pagerduty.com/log_entries/R5S5T07QR1SZRQFYB7SXEO2EKZ",
    "html_url": "https://getport-io.pagerduty.com/incidents/Q1P3AHC3KLGVAS/log_entries/R5S5T07QR1SZRQFYB7SXEO2EKZ"
  },
  "alert_counts": {
    "all": 0,
    "triggered": 0,
    "resolved": 0
  },
  "is_mergeable": true,
  "escalation_policy": {
    "id": "P7LVMYP",
    "type": "escalation_policy_reference",
    "summary": "Test Escalation Policy",
    "self": "https://api.pagerduty.com/escalation_policies/P7LVMYP",
    "html_url": "https://getport-io.pagerduty.com/escalation_policies/P7LVMYP"
  },
  "teams": [],
  "pending_actions": [],
  "acknowledgements": [],
  "basic_alert_grouping": null,
  "alert_grouping": null,
  "last_status_change_by": {
    "id": "PWJAGSD",
    "type": "service_reference",
    "summary": "Port Test Service",
    "self": "https://api.pagerduty.com/services/PWJAGSD",
    "html_url": "https://getport-io.pagerduty.com/service-directory/PWJAGSD"
  },
  "urgency": "high",
  "id": "Q1P3AHC3KLGVAS",
  "type": "incident",
  "summary": "[#2] Example Incident",
  "self": "https://api.pagerduty.com/incidents/Q1P3AHC3KLGVAS",
  "html_url": "https://getport-io.pagerduty.com/incidents/Q1P3AHC3KLGVAS"
}
```

</details>

#### 映射结果

结合样本有效载荷和 Ocean 配置，可生成以下 Port 实体: 

<details>
<summary> Schedule entity in Port</summary>

```json showLineNumbers
{
  "identifier": "PWAXLIH",
  "title": "Port Test Service - Weekly Rotation",
  "icon": null,
  "blueprint": "pagerdutySchedule",
  "team": [],
  "properties": {
    "url": "https://getport-io.pagerduty.com/schedules/PWAXLIH",
    "timezone": "Asia/Jerusalem",
    "description": "Asia/Jerusalem",
    "users": ["Adam", "Alice", "Doe", "Demo", "Pages"]
  },
  "relations": {},
  "createdAt": "2023-12-01T13:18:02.215Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-12-01T13:18:02.215Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>

<details>
<summary> Service entity in Port</summary>

```json showLineNumbers
{
  "identifier": "PGAAJBE",
  "title": "My Test Service",
  "icon": null,
  "blueprint": "pagerdutyService",
  "team": [],
  "properties": {
    "status": "active",
    "url": "https://getport-io.pagerduty.com/service-directory/PGAAJBE",
    "oncall": "devops-port@pager-demo.com"
  },
  "relations": {},
  "createdAt": "2023-11-01T13:18:02.215Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-11-01T13:18:02.215Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>

<details>
<summary> Incident entity in Port</summary>

```json showLineNumbers
{
  "identifier": "Q1P3AHC3KLGVAS",
  "title": "Example Incident",
  "icon": null,
  "blueprint": "pagerdutyIncident",
  "team": [],
  "properties": {
    "status": "triggered",
    "url": "https://api.pagerduty.com/incidents/Q1P3AHC3KLGVAS",
    "urgency": "high",
    "responder": "Username",
    "escalation_policy": "Test Escalation Policy",
    "created_at": "2023-07-30T11:29:21.000Z",
    "updated_at": "2023-07-30T11:29:21.000Z"
  },
  "relations": {
    "pagerdutyService": "PWJAGSD"
  },
  "createdAt": "2023-08-07T07:56:04.384Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-08-07T07:56:04.384Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>

## 通过 webhook 进行替代安装

虽然上述 Ocean 集成是推荐的安装方法，但您可能更喜欢使用 webhook 从 PagerDuty 引用数据。 如果是这样，请使用以下说明: 

<details>

<summary><b>Webhook installation (click to expand)</b></summary>

在本示例中，您将在[PagerDuty](https://www.pagerduty.com/) 和 Port 之间创建 webhook 集成，将 PagerDuty 服务及其相关事件导入 Port。该集成将涉及设置 webhook，以便在创建或更新事件时接收来自 PagerDuty 的通知，使 Port 能够相应地导入和处理事件实体。

<h2>Import PagerDuty services and incidents</h2>

<h3>Port configuration</h3>

创建以下蓝图定义: 

<details>
<summary>PagerDuty service blueprint</summary>

<PagerDutyServiceBlueprint/>

</details>

<details>
<summary>PagerDuty incident blueprint</summary>

<PagerDutyIncidentBlueprint/>

</details>

创建以下 webhook 配置[using Port UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>PagerDuty webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `PagerDuty Mapper`；
    2.标识符 : `pagerduty_mapper`；
    3.Description : `将 PagerDuty 服务及其相关事件映射到 Port` 的 webhook 配置；
    4.图标 : `Pagerduty`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <PagerDutyWebhookConfig/>
3.向下滚动到**高级设置**，输入以下详细信息: 
    1. secret: `WEBHOOK_SECRET`；
    2.签名头名称:  `X-Pagerduty-Signature`；
    3.签名算法: 从下拉选项中选择 `sha256`；
    4.签名前缀 : `v1=`.
    5.点击页面底部的**保存**。
    切记用在 PagerDuty 中订阅 webhook 后收到的真实secret更新 `WEBHOOK_SECRET`。

</details>

<h3>Create a webhook in PagerDuty</h3>

1. 转到[PagerDuty](https://www.pagerduty.com/) ，选择要配置 webhook 的账户。
2. 导航至导航栏中的**集成**，然后点击**通用 Webhook (v3)**。
3. 点击 ** 新建 Webhook**，并提供以下信息: 
    1. Webhook URL` - 输入[creating the webhook configuration](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints) 后收到的 `url` 键的值。
    2. 范围类型"- 选择是接收特定服务的 webhook 事件(如果适用，请选择 "服务")，还是接收账户中所有服务的 webhook 事件(如果适用，请选择 "账户")。
    3. `Description` - 为您的 webhook 提供可选描述。
    4. 事件订阅"- 选择要订阅的事件类型。
    5.自定义标头"- 输入要添加到 webhook 有效负载的任何可选 HTTP 标头。
4.单击 ** 添加 webhook** 创建您的 webhook。
5.或者，你也可以使用 `curl` 方法来创建 webhook。复制下面的代码并在终端运行: 

```curl showLineNumbers
curl --request POST \
  --url \
 https://api.pagerduty.com/webhook_subscriptions
  --header 'Accept: application/vnd.pagerduty+json;version=2' \
  --header 'Authorization: Token token=<YOUR_PAGERDUTY_API_TOKEN>' \
  --header 'Content-Type: application/json' \
  --data \
 '{
  "webhook_subscription": {
  "delivery_method": {
    "type": "http_delivery_method",
    "url": "https://ingest.getport.io/<YOUR_PORT_WEBHOOK_KEY>",
    "custom_headers": [
      {
        "name": "your-header-name",
        "value": "your-header-value"
      }
    ]
  },
  "description": "Sends PagerDuty v3 webhook events to Port.",
  "events": [
      "service.created",
      "service.updated",
      "incident.triggered",
      "incident.responder.added",
      "incident.acknowledged",
      "incident.annotated",
      "incident.delegated",
      "incident.escalated",
      "incident.priority_updated",
      "incident.reassigned",
      "incident.reopened",
      "incident.resolved",
      "incident.responder.replied",
      "incident.status_update_published",
      "incident.unacknowledged"
  ],
  "filter": {
    "type": "account_reference"
  },
  "type": "webhook_subscription"
  }
  }'
```

:::tip 为了查看 PagerDuty webhooks 中可用的不同事件、[look here](https://developer.pagerduty.com/docs/db0fa8c8984fc-overview#event-types)

:::

完成！您在 PagerDuty 中的服务或事件发生的任何变化都将触发指向 Port 提供的 webhook URL 的 webhook 事件。 Port 将根据映射解析事件并相应地更新目录实体。

<h2>Let's Test It</h2>

本节包括当创建或更新事件时从 PagerDuty 发送的 webhook 事件示例。 此外，本节还包括根据上一节提供的 webhook 配置从事件中创建的实体。

<h3>Payload</h3>

下面是创建 PagerDuty 事件时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary>Webhook event payload</summary>

```json showLineNumbers
{
  "event": {
    "id": "01DVUHO6P4XQDFJ9AHOADT3UQ4",
    "event_type": "incident.triggered",
    "resource_type": "incident",
    "occurred_at": "2023-06-12T11:56:08.355Z",
    "agent": {
      "html_url": "https://your_account.pagerduty.com/users/PJCRRLH",
      "id": "PJCRRLH",
      "self": "https://api.pagerduty.com/users/PJCRRLH",
      "summary": "username",
      "type": "user_reference"
    },
    "client": "None",
    "data": {
      "id": "Q01J2OS7YBWLNY",
      "type": "incident",
      "self": "https://api.pagerduty.com/incidents/Q01J2OS7YBWLNY",
      "html_url": "https://your_account.pagerduty.com/incidents/Q01J2OS7YBWLNY",
      "number": 7,
      "status": "triggered",
      "incident_key": "acda20953f7446248f90260db65144f8",
      "created_at": "2023-06-12T11:56:08Z",
      "title": "Test PagerDuty Incident",
      "service": {
        "html_url": "https://your_account.pagerduty.com/services/PWJAGSD",
        "id": "PWJAGSD",
        "self": "https://api.pagerduty.com/services/PWJAGSD",
        "summary": "Port Internal Service",
        "type": "service_reference"
      },
      "assignees": [
        {
          "html_url": "https://your_account.pagerduty.com/users/PRGAUI4",
          "id": "PRGAUI4",
          "self": "https://api.pagerduty.com/users/PRGAUI4",
          "summary": "username",
          "type": "user_reference"
        }
      ],
      "escalation_policy": {
        "html_url": "https://your_account.pagerduty.com/escalation_policies/P7LVMYP",
        "id": "P7LVMYP",
        "self": "https://api.pagerduty.com/escalation_policies/P7LVMYP",
        "summary": "Test Escalation Policy",
        "type": "escalation_policy_reference"
      },
      "teams": [],
      "priority": "None",
      "urgency": "high",
      "conference_bridge": "None",
      "resolve_reason": "None"
    }
  }
}
```

</details>

<h3>Mapping Result</h3>

结合示例有效载荷和 webhook 配置可生成以下 Port 实体: 

```json showLineNumbers
{
  "identifier": "Q01J2OS7YBWLNY",
  "title": "Test PagerDuty Incident",
  "blueprint": "pagerdutyIncident",
  "team": [],
  "properties": {
    "status": "triggered",
    "url": "https://your_account.pagerduty.com/incidents/Q01J2OS7YBWLNY",
    "details": "Test PagerDuty Incident",
    "urgency": "high",
    "responder": "Username",
    "escalation_policy": "Test Escalation Policy"
  },
  "relations": {
    "microservice": "PWJAGSD"
  }
}
```

<h2>Import PagerDuty historical data</h2>

在本例中，您将使用 Provider 提供的 Bash 脚本从 PagerDuty API 获取数据并将其引用到 Port。

脚本从 PagerDuty 中提取服务和事件，并分别作为微服务和事件实体发送到 Port。

<h3>Port configuration</h3>

本示例使用了上一节中相同的[blueprint](#prerequisites) 定义以及新的 webhook 配置: 

创建以下 webhook 配置[using Port UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>PagerDuty webhook configuration for historical data</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `PagerDuty History Mapper`；
    2.标识符 : `pagerduty_history_mapper`；
    3.Description : `将 PagerDuty 历史服务及其相关事件映射到 Port` 的 webhook 配置；
    4.图标 : `Pagerduty`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <PagerDutyWebhookHistory/>
3.向下滚动到**高级设置**，输入以下详细信息: 
    1. secret: `WEBHOOK_SECRET`；
    2.签名头名称:  `X-Pagerduty-Signature`；
    3.签名算法: 从下拉选项中选择 `sha256`；
    4.签名前缀 : `v1=`.
    5.点击页面底部的**保存**。

切记用在 PagerDuty 中订阅 webhook 后收到的真实secret更新`WEBHOOK_SECRET`。

</details>

<details>
<summary> PagerDuty Bash script for historical data </summary>

<PagerDutyScript/>

</details>

<h3>How to Run the script</h3>

该脚本需要两个配置值: 

1. PD_TOKEN": 您的 PagerDuty API 标记；
2. PORT_URL: 您的 Port webhook URL。

然后运行脚本触发: 

```bash showLineNumbers
bash pagerduty_to_port.sh
```

该脚本从 PagerDuty 获取服务和事件，并将其发送到 Port。

:::tip 脚本会将每个服务和事件的 JSON 有效载荷写入名为 `output.json` 的文件。 如果遇到任何问题，这将有助于调试。

:::

完成！现在您可以将历史数据从 PagerDuty 导入 Port。 Port 将根据映射解析事件并相应更新目录实体。

</details>
