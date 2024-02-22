import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import DockerParameters from "./_wiz-docker-parameters.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'
import WizBlueprint from "../webhook/examples/resources/wiz/_example_wiz_issue_blueprint.mdx";
import WizConfiguration from "../webhook/examples/resources/wiz/_example_wiz_issue_webhook_configuration.mdx";

# Wiz

通过 Wiz 集成，您可以根据您的映射和定义，将 Wiz 账户中的 "项目"、"问题"、"控件 "和 "服务单 "导入 Port。

## 常见被用于情况

* 映射 Wiz 组织环境中的 "项目"、"问题"、"控件 "和 "服务单"。
* 实时观察对象更改(创建/更新/删除)，并自动将更改应用到您的 Port 实体中。

## 先决条件

<Prerequisites />

您的 Wiz 凭据应具有 "read:projects "和 "read:issues "权限范围。请访问 Wiz[documentation](https://integrate.wiz.io/reference/prerequisites) ，了解如何获取凭据和设置权限。

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                           | Description                                                                                                        | Required |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------- |
| `port.clientId`                     | Your port client id ([Get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials))                                                                                               | ✅       |
| `port.clientSecret`                 | Your port client secret ([Get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials))                                                                                           | ✅       |
| `integration.identifier`            | Change the identifier to describe your integration                                                                 | ✅       |
| `integration.type`                  | The integration type                                                                                               | ✅       |
| `integration.eventListener.type`    | The event listener type                                                                                            | ✅       |
| `integration.secrets.wizClientId`   | The Wiz Client ID                                                                                                  | ✅       |
| `integration.secrets.wizClientSecret`| The Wiz Client Secret                                                                                             | ✅       |
| `integration.config.wizApiUrl`      | The Wiz API URL.                                                                                                   | ✅       |
| `integration.config.wizTokenUrl`    | The Wiz Token Authentication URL                                                                                   | ✅       |
| `integration.config.appHost`        | The host of the Port Ocean app. Used to set up the integration endpoint as the target for Webhooks created in Wiz  | ❌       |
| `integration.secret.wizWebhookVerificationToken`  | This is a password you create, that is used to verify webhook events to Port                                       | ❌       |
| `scheduledResyncInterval`           | The number of minutes between each resync                                                                          | ❌       |
| `initializePortResources`           | Default true, When set to true the integration will create default blueprints and the port App config Mapping      | ❌       |


<br/>

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-wiz-integration port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set initializePortResources=true  \
    --set scheduledResyncInterval=120 \
    --set integration.identifier="my-wiz-integration"  \
    --set integration.type="wiz"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.secrets.wizClientId="WIZ_CLIENT_ID"  \
    --set integration.secrets.wizClientSecret="WIZ_CLIENT_SECRET" \
        --set integration.secrets.wizApiUrl="WIZ_API_URL"  \
    --set integration.config.wizTokenUrl="WIZ_TOKEN_URL"
```

</TabItem>

<TabItem value="one-time" label="Scheduled">
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Wiz integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />

<br/>

下面是 `wiz-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Wiz Exporter Workflow

# This workflow responsible for running Wiz exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Wiz Integration
        run: |
          # Set Docker image and run the container
          integration_type="wiz"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_ID=${{ secrets.OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_ID }} \
          -e OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_SECRET=${{ secrets.OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_SECRET }} \
          -e OCEAN__INTEGRATION__CONFIG__WIZ_API_URL=${{ secrets.OCEAN__INTEGRATION__CONFIG__WIZ_API_URL }} \
          -e OCEAN__INTEGRATION__CONFIG__WIZ_TOKEN_URL=${{ secrets.OCEAN__INTEGRATION__CONFIG__WIZ_TOKEN_URL }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

</TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Wiz integration once and then exit, this is useful for **scheduled** ingestion of data.

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
        stage('Run Wiz Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_ID', variable: 'OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_ID'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_SECRET', variable: 'OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_SECRET'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__WIZ_API_URL', variable: 'OCEAN__INTEGRATION__CONFIG__WIZ_API_URL'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__WIZ_TOKEN_URL', variable: 'OCEAN__INTEGRATION__CONFIG__WIZ_TOKEN_URL'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="wiz"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_ID=$OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_ID \
                                -e OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_SECRET=$OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_SECRET \
                                -e OCEAN__INTEGRATION__CONFIG__WIZ_API_URL=$OCEAN__INTEGRATION__CONFIG__WIZ_API_URL \
                                -e OCEAN__INTEGRATION__CONFIG__WIZ_TOKEN_URL=$OCEAN__INTEGRATION__CONFIG__WIZ_TOKEN_URL \
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

<AzurePremise name="Wiz" />

<DockerParameters />

<br/>

下面是 `wiz-integration.yml` Pipelines 文件的示例: 

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
    integration_type="wiz"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
        -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
        -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
        -e OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_ID=${OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_ID} \
        -e OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_SECRET=${OCEAN__INTEGRATION__CONFIG__WIZ_CLIENT_SECRET} \
        -e OCEAN__INTEGRATION__CONFIG__WIZ_API_URL=${OCEAN__INTEGRATION__CONFIG__WIZ_API_URL} \
        -e OCEAN__INTEGRATION__CONFIG__WIZ_TOKEN_URL=${OCEAN__INTEGRATION__CONFIG__WIZ_TOKEN_URL} \
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

## Ingesting Wiz objects

Wiz 集成使用 YAML 配置来描述将数据加载到开发人员门户的过程。

下面是配置中的一个示例片段，演示了从 Wiz 获取 "项目 "数据的过程: 

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: project
    selector:
      query: 'true'
    port:
      entity:
        mappings:
          blueprint: '"wizProject"'
          identifier: .id
          title: .name
          properties:
            archived: .archived
            businessUnit: .businessUnit
            description: .description
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 从 Wiz 的 API 事件中对现有字段和 Values 进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了从 Wiz 中查询哪些资源，以及在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被用于来映射 Wiz 中的数据，可以引用下面链接的 API 响应中出现的任何字段来进行映射配置。

* * [`Project`](https://integrate.wiz.io/reference/pull-projects)
* [`Issue`](https://integrate.wiz.io/reference/issues-tutorial)
* [`Control`](https://integrate.wiz.io/docs/welcome#controls)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: project
      selector:
      ...
  ```


* 类型 "键是 Wiz 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: project
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: project
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
      # highlight-end
      port:
  ```


* Port"、"实体 "和 "映射 "键被用来将 Wiz 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: project
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one Wiz object to a Port entity. Each value is a JQ query.
            identifier: .id
            title: .attributes.name
            blueprint: '"wizProject"'
            identifier: .id
            title: .name
            properties:
              archived: .archived
              businessUnit: .businessUnit
              description: .description
        # highlight-end
    - kind: project # In this instance project is mapped again with a different filter
      selector:
        query: '.name == "MyProjectName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): ::: 

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Wiz 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 Wiz 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在代码质量和安全 Provider 类别下选择 Wiz。
5. 根据您的需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

### 项目

<details>
<summary>Project blueprint</summary>

```json showLineNumbers
{
  "identifier": "wizProject",
  "description": "This blueprint represents a wiz project",
  "title": "Wiz Project",
  "icon": "Box",
  "schema": {
    "properties": {
      "archived": {
        "type": "boolean",
        "title": "Archived?",
        "description": "Is the project archived?"
      },
      "businessUnit": {
        "type": "string",
        "title": "Business Unit",
        "description": "the business unit of the project"
      },
      "description": {
        "type": "string",
        "title": "Description",
        "description": "the project description"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "issues": {
      "target": "wizIssue",
      "title": "Issues",
      "description": "The issues affecting this project",
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
  - kind: project
    selector:
      query: 'true'
    port:
      entity:
        mappings:
          blueprint: '"wizProject"'
          identifier: .id
          title: .name
          properties:
            archived: .archived
            businessUnit: .businessUnit
            description: .description
```

</details>

### 控制

<details>
<summary>Control blueprint</summary>

```json showLineNumbers
{
  "identifier": "wizControl",
  "description": "This blueprint represents a wiz source rule",
  "title": "Wiz Control",
  "icon": "Flag",
  "schema": {
    "properties": {
      "controlDescription": {
        "type": "string",
        "title": "Control Description",
        "description": "the control description"
      },
      "resolutionRecommendation": {
        "type": "string",
        "title": "Control Recommendation",
        "description": "the control recommendation on resolving issues"
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
resources:
  - kind: control
    selector:
      query: 'true'
    port:
      entity:
        mappings:
          blueprint: '"wizControl"'
          identifier: .id
          title: .name
          properties:
            controlDescription: .controlDescription
            resolutionRecommendation: .resolutionRecommendation
```

</details>

### 问题

<details>
<summary>Issue blueprint</summary>

```yaml showLineNumbers
{
  "identifier": "wizIssue",
  "description": "This blueprint represents a wiz issue",
  "title": "Wiz Issue",
  "icon": "Alert",
  "schema": {
    "properties": {
      "url": {
        "type": "string",
        "title": "Issue URL",
        "format": "url",
        "description": "the link to the issue"
      },
      "status": {
        "title": "Status",
        "type": "string",
        "enum": [
          "OPEN",
          "IN_PROGRESS",
          "RESOLVED",
          "REJECTED"
        ],
        "enumColors": {
          "OPEN": "blue",
          "IN_PROGRESS": "orange",
          "RESOLVED": "green",
          "REJECTED": "darkGray"
        }
      },
      "severity": {
        "title": "Severity",
        "type": "string",
        "enum": [
          "INFORMATIONAL",
          "LOW",
          "MEDIUM",
          "HIGH",
          "CRITICAL"
        ],
        "enumColors": {
          "INFORMATIONAL": "blue",
          "LOW": "yellow",
          "MEDIUM": "orange",
          "HIGH": "red",
          "CRITICAL": "red"
        }
      },
      "type": {
        "title": "Type",
        "type": "string"
      },
      "vulnerability": {
        "title": "Vulnerability",
        "type": "object",
        "description": "The identified security risk"
      },
      "notes": {
        "title": "Notes",
        "type": "array"
      },
      "createdAt": {
        "title": "Created At",
        "type": "string",
        "format": "date-time"
      },
      "updatedAt": {
        "title": "Updated At",
        "type": "string",
        "format": "date-time"
      },
      "dueAt": {
        "title": "Due At",
        "type": "string",
        "format": "date-time"
      },
      "resolvedAt": {
        "title": "Resolved At",
        "type": "string",
        "format": "date-time"
      },
      "statusChangedAt": {
        "title": "Status ChangedAt",
        "type": "string",
        "format": "date-time"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "projects": {
      "target": "wizProject",
      "title": "Affected Projects",
      "description": "The projects affected by this issue",
      "required": false,
      "many": true
    },
    "serviceTickets": {
      "target": "wizServiceTicket",
      "title": "Service Tickets",
      "description": "The service tickets belonging to this issue",
      "required": false,
      "many": true
    },
    "control": {
      "target": "wizControl",
      "title": "Control",
      "description": "The control that flagged this issue",
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
  - kind: issue
    selector:
      query: 'true'
    port:
      entity:
        mappings:
          blueprint: '"wizIssue"'
          identifier: .id
          title: .entitySnapshot.name + " | " + .entitySnapshot.type
          properties:
            url: .id as $id | "https://app.wiz.io/issues#~(issue~'" + $id + ")"
            status: .status
            severity: .severity
            type: .type
            notes: .notes
            vulnerability: .entitySnapshot
            createdAt: .createdAt
            updatedAt: .updatedAt
            statusChangedAt: .statusChangedAt
            resolvedAt: .resolvedAt
          relations:
            projects: .projects[].id
            serviceTickets: .serviceTickets[].externalId
            control: .sourceRule.id
```

</details>

### 服务票据

<details>
<summary>Service Ticket blueprint</summary>

```yaml showLineNumbers
{
  "identifier": "wizServiceTicket",
  "description": "This blueprint represents a wiz service ticket",
  "title": "Wiz Service Ticket",
  "icon": "Book",
  "schema": {
    "properties": {
      "url": {
        "type": "string",
        "title": "Ticket URL",
        "format": "url",
        "description": "the service ticket URL"
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
resources:
  - kind: serviceTicket
  selector:
    query: 'true'
  port:
    entity:
      mappings:
        blueprint: '"wizServiceTicket"'
        identifier: .externalId
        title: .name
        properties:
          url: .url
```

</details>

## 通过 webhook 进行替代安装

虽然上述 Ocean 集成是推荐的安装方法，但您可能更喜欢使用 webhook 从 Wiz 引用数据。 如果是这样，请使用以下说明: 

<details>

<summary><b>Webhook installation (click to expand)</b></summary>

在本示例中，您将在[Wiz](https://wiz.io/) 和 Port 之间创建 webhook 集成，将 Wiz 问题实体导入 Port。

<h2>Port configuration</h2>

创建以下蓝图定义: 

<details>
<summary>Wiz issue blueprint</summary>

<WizBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>Wiz issue webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Wiz Mapper`；
    2.标识符 : `wiz_mapper`；
    3.Description : `将 Wiz 问题映射到 Port` 的 webhook 配置；
    4.图标 : `Box`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <WizConfiguration/>

</details>

<h2>Create a webhook in Wiz</h2>

1. 向 win@wiz.io 发送电子邮件，请求访问开发人员文档，或联系您的 Wiz 客户经理。
2. 按照文档中的[guide](https://integrate.wiz.io/reference/webhook-tutorial#create-a-custom-webhook) 创建 webhook。

完成！在 Wiz 中创建的任何问题都将触发一个 webhook 事件，该事件将发送到 Provider 提供的 webhook URL。 Port 将根据映射解析事件，并相应地更新目录实体。

</details>