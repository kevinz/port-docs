import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# Dynatrace

我们的 Dyantrace 集成允许您根据映射和定义，将 Dynatrace 实例中的 "问题"、"slo "和 "实体 "资源导入 Port。

## 常见被引用情况

* 在 Dynatrace 中映射受监控的实体、问题和 SLO。
* 实时关注对象变更(创建/更新)，并自动将变更应用到 Port 中的实体。

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                             | Description                                                                                                   | Required |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| `port.clientId`                       | Your port [client id](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)                                                                                           | ✅       |
| `port.clientSecret`                   | Your port [client secret](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)                                                                                       | ✅       |
| `port.baseUrl`                        | Your port base url, relevant only if not using the default port app                                           | ❌       |
| `integration.identifier`              | Change the identifier to describe your integration                                                            | ✅       |
| `integration.type`                    | The integration type                                                                                          | ✅       |
| `integration.eventListener.type`      | The event listener type                                                                                       | ✅       |
| `integration.secrets.dynatraceApiKey` | API Key for Dynatrace instance                                                                          | ✅       |
| `integration.config.dynatraceHostUrl` | The URL to the Dynatrace instance                                                                          | ✅       |
| `scheduledResyncInterval`             | The number of minutes between each resync                                                                     | ❌       |
| `initializePortResources`             | Default true, When set to true the integration will create default blueprints and the port App config Mapping | ❌       |


<br/>

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-dynatrace-integration port-labs/port-ocean \
  --set port.clientId="CLIENT_ID"  \
  --set port.clientSecret="CLIENT_SECRET"  \
  --set initializePortResources=true  \
  --set scheduledResyncInterval=60  \
  --set integration.identifier="my-dynatrace-integration"  \
  --set integration.type="dynatrace"  \
  --set integration.eventListener.type="POLLING"  \
  --set integration.secrets.dynatraceApiKey="<your-api-key>"  \
  --set integration.config.dynatraceHostUrl="<your-isntance-url>"
```

</TabItem>

<TabItem value="one-time" label="Scheduled">
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Dynatrace integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 


| Parameter                                        | Description                                                                                                        | Required |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ | -------- |
| `OCEAN__INTEGRATION__CONFIG__DYNATRACE_API_KEY`  | The Dynatrace API key                                                                                              | ✅       |
| `OCEAN__INTEGRATION__CONFIG__DYNATRACE_HOST_URL` | The Dynatrace host URL                                                                                             | ✅       |
| `OCEAN__INITIALIZE_PORT_RESOURCES`               | Default true, When set to false the integration will not create default blueprints and the port App config Mapping | ❌       |
| `OCEAN__INTEGRATION__IDENTIFIER`                 | Change the identifier to describe your integration, if not set will use the default one                            | ❌       |
| `OCEAN__PORT__CLIENT_ID`                         | Your port client id                                                                                                | ✅       |
| `OCEAN__PORT__CLIENT_SECRET`                     | Your port client secret                                                                                            | ✅       |
| `OCEAN__PORT__BASE_URL`                          | Your port base url, relevant only if not using the default port app                                                | ❌       |


<br/>

以下是 `dynatrace-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Dynatrace Exporter Workflow

# This workflow responsible for running Dynatrace exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Dynatrace Integration
        run: |
          # Set Docker image and run the container
          integration_type="dynatrace"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__DYNATRACE_API_KEY=${{ secrets.OCEAN__INTEGRATION__CONFIG__DYNATRACE_API_KEY }} \
          -e OCEAN__INTEGRATION__CONFIG__DYNATRACE_HOST_URL=${{ secrets.OCEAN__INTEGRATION__CONFIG__DYNATRACE_HOST_URL }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Dynatrace integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/)的 "Secret Text "类型: 


| Parameter                                        | Description                                                                                                        | Required |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ | -------- |
| `OCEAN__INTEGRATION__CONFIG__DYNATRACE_API_KEY`  | The Dynatrace API key                                                                                              | ✅       |
| `OCEAN__INTEGRATION__CONFIG__DYNATRACE_HOST_URL` | The Dynatrace host URL                                                                                             | ✅       |
| `OCEAN__INITIALIZE_PORT_RESOURCES`               | Default true, When set to false the integration will not create default blueprints and the port App config Mapping | ❌       |
| `OCEAN__INTEGRATION__IDENTIFIER`                 | Change the identifier to describe your integration, if not set will use the default one                            | ❌       |
| `OCEAN__PORT__CLIENT_ID`                         | Your port client id                                                                                                | ✅       |
| `OCEAN__PORT__CLIENT_SECRET`                     | Your port client secret                                                                                            | ✅       |
| `OCEAN__PORT__BASE_URL`                          | Your port base url, relevant only if not using the default port app                                                | ❌       |


<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

```text showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run Dynatrace Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__DYNATRACE_API_KEY', variable: 'OCEAN__INTEGRATION__CONFIG__DYNATRACE_API_KEY'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__DYNATRACE_HOST_URL', variable: 'OCEAN__INTEGRATION__CONFIG__DYNATRACE_HOST_URL'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="dynatrace"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__DYNATRACE_API_KEY=$OCEAN__INTEGRATION__CONFIG__DYNATRACE_API_KEY \
                                -e OCEAN__INTEGRATION__CONFIG__DYNATRACE_HOST_URL=$OCEAN__INTEGRATION__CONFIG__DYNATRACE_HOST_URL \
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
  </Tabs>
</TabItem>

</Tabs>

### 生成 Dynatrace API 密钥

1. 导航至 `<instanceURL>/ui/apps/dynatrace.classic.token/ui/access-tokens`。例如，如果您在 `https://npm82883.apps.dynatrace.com` 访问 Dynatrace 实例，则应导航至 `https://npm82883.apps.dynatrace.com/ui/apps/dynatrace.classic.tokens/ui/access-tokens`。
2. 单击 **Generate new token** 创建新令牌。确保权限: 读取实体"、"读取问题 "和 "读取 SLO "已分配给令牌。

## 接收 Dynatrace 对象

Dynatrace 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 Dynatrace 获取 "实体 "数据的过程: 

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: entity
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .entityId
          title: .displayName
          blueprint: '"dynatraceEntity"'
          properties:
            firstSeen: ".firstSeenTms / 1000 | todate"
            lastSeen: ".lastSeenTms / 1000 | todate"
            type: .type
            tags: .tags[].stringRepresentation
            managementZones: .managementZones[].name
            properties: .properties
            fromRelationships: .fromRelationships
            toRelationships: .toRelationships
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 Dynatrace API 事件的现有字段和值进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了从 Dynatrace 查询哪些资源，以及在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可用于映射来自 Dynatrace 的数据，可以引用出现在下面链接的 API 响应中的任何字段进行映射配置。

* * [`problem`](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/problems-v2/problems/get-problems-list#definition--Problem)
* [`entity`](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/entity-v2/get-entities-list#definition--Entity)
* [`slo`](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/service-level-objectives/get-all#definition--SLO)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: entity
      selector:
      ...
  ```


* 类型 "键是 Dynatrace 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: entity
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: entity
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
      # highlight-end
      port:
  ```


* Port"、"实体 "和 "映射 "键被用来将 Dynatrace 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: entity
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one Dynatrace object to a Port entity. Each value is a JQ query.
            identifier: .entityId
            title: .displayName
            blueprint: '"dynatraceEntity"'
            properties:
              firstSeen: ".firstSeenTms / 1000 | todate"
              lastSeen: ".lastSeenTms / 1000 | todate"
              type: .type
              tags: .tags[].stringRepresentation
              managementZones: .managementZones[].name
              properties: .properties
              fromRelationships: .fromRelationships
              toRelationships: .toRelationships
        # highlight-end
    - kind: entity # In this instance entity is mapped again with a different filter
      selector:
        query: '.displayName == "MyEntityName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): ::: 

## 配置实时更新

目前，Dynatrace API 不支持编程式 webhook 创建。 要在 Dynatrace 中设置 webhook 配置，以便向 Ocean 集成发送警报通知，请按照以下步骤操作: 

### 前提条件

使用以下格式准备 webhook `URL`: `{app_host}/integration/webhook`。 `app_host` 参数应与部署集成的 ingress 或外部负载平衡器匹配。例如，如果您的 ingress 或负载平衡器在 `https://myservice.domain.com` 公开 Dynatrace Ocean 集成，则您的 webhook `URL` 应为 `https://myservice.domain.com/integration/webhook`。

#### 在 Dynatrace 中创建 webhook

1. 转到 Dynatrace；
2. 转到 **设置** > **集成** > **问题通知**；
3. 选择 **添加通知**；
4. 从可用的通知类型中选择 **自定义集成**；
5. 使用以下详细信息配置通知；
    1. 已启用"- 确保通知已启用；
    2. 显示名称"- 使用有意义的名称，如 Port Ocean Webhook；
    3. Webhook URL` - 输入您在上面创建的 `URL` 的值；
    4.启用 **调用 webhook 是新事件合并到现有问题**；
    5.自定义有效载荷"- 粘贴以下配置: 


    ```

    {
        "State":"{State}",
        "ProblemID":"{ProblemID}",
        "ProblemTitle":"{ProblemTitle}"
    }

    ```

    You can customize to your taste, the only important thing is the `ProblemID` key. The webhook integration will not work without it.
   6. `Alerting profile` - select the corresponding alerting profile
   7. Leave the rest of the fields as is;
6. Click **Save changes**

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Dynatrace 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要使用 Dynatrace 进行引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 选择事件管理类别下的 Dynatrace。
5. 根据需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

### 实体

<details>
<summary>Entity blueprint</summary>

```json showLineNumbers
{
  "identifier": "dynatraceEntity",
  "description": "This blueprint represents a Dynatrace Entity",
  "title": "Dynatrace Entity",
  "icon": "Dynatrace",
  "schema": {
    "properties": {
      "firstSeen": {
        "type": "string",
        "title": "First Seen",
        "description": "The timestamp at which the entity was first seen, in UTC milliseconds.",
        "format": "date-time"
      },
      "lastSeen": {
        "type": "string",
        "title": "Last Seen",
        "description": "The timestamp at which the entity was last seen, in UTC milliseconds.",
        "format": "date-time"
      },
      "type": {
        "type": "string",
        "title": "Type",
        "description": "The type of the entity."
      },
      "tags": {
        "type": "array",
        "title": "Tags",
        "description": "A list of tags of the entity."
      },
      "managementZones": {
        "type": "array",
        "title": "Management Zones",
        "description": "A list of all management zones that the entity belongs to."
      },
      "properties": {
        "type": "properties",
        "title": "Properties",
        "description": "A list of additional properties of the entity."
      },
      "fromRelationships": {
        "type": "object",
        "title": "From Relationships",
        "description": "A list of all relationships where the entity is the source."
      },
      "toRelationships": {
        "type": "object",
        "title": "To Relationships",
        "description": "A list of all relationships where the entity is the target."
      }
    }
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
  - kind: entity
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .entityId
          title: .displayName
          blueprint: '"dynatraceEntity"'
          properties:
            firstSeen: ".firstSeenTms / 1000 | todate"
            lastSeen: ".lastSeenTms / 1000 | todate"
            type: .type
            tags: .tags[].stringRepresentation
            managementZones: .managementZones[].name
            properties: .properties
            fromRelationships: .fromRelationships
            toRelationships: .toRelationships
```

</details>

#### 问题

<details>
<summary> Problem blueprint</summary>

```json showlineNumbers
{
  "identifier": "dynatraceProblem",
  "description": "This blueprint represents a Dynatrace Problem",
  "title": "Dynatrace Problem",
  "icon": "Dynatrace",
  "schema": {
    "properties": {
      "entityTags": {
        "type": "array",
        "title": "Entity Tags",
        "description": "A list of all entity tags of the problem."
      },
      "evidenceDetails": {
        "type": "array",
        "title": "Evidence Details",
        "description": "A list of all evidence details of the problem."
      },
      "managementZones": {
        "type": "array",
        "title": "Management Zones",
        "description": "A list of all management zones that the problem belongs to."
      },
      "problemFilters": {
        "type": "array",
        "title": "Problem Filters",
        "description": "A list of alerting profiles that match the problem."
      },
      "severityLevel": {
        "type": "string",
        "title": "Severity Level",
        "description": "The severity level of the problem.",
        "enum": [
          "AVAILABILITY",
          "CUSTOM_ALERT",
          "ERROR",
          "INFO",
          "MONITORING_UNAVAILABLE",
          "PERFORMANCE",
          "RESOURCE_CONTENTION"
        ],
        "enumColors": {
          "AVAILABILITY": "blue",
          "CUSTOM_ALERT": "turquoise",
          "ERROR": "red",
          "INFO": "green",
          "MONITORING_UNAVAILABLE": "gray",
          "PERFORMANCE": "orange",
          "RESOURCE_CONTENTION": "yellow"
        }
      },
      "status": {
        "type": "string",
        "title": "Status",
        "description": "The status of the problem.",
        "enum": ["CLOSED", "OPEN"],
        "enumColors": {
          "CLOSED": "green",
          "OPEN": "red"
        }
      },
      "startTime": {
        "type": "string",
        "title": "Start Time",
        "description": "The start time of the problem, in UTC milliseconds.",
        "format": "date-time"
      },
      "endTime": {
        "type": "string",
        "title": "End Time",
        "description": "The end time of the problem, in UTC milliseconds.",
        "format": "date-time"
      }
    },
    "required": []
  },
  "relations": {
    "impactedEntities": {
      "title": "Impacted Entities",
      "target": "dynatraceEntity",
      "required": false,
      "many": true
    },
    "linkedProblemInfo": {
      "title": "Linked Problem Info",
      "target": "dynatraceProblem",
      "required": false,
      "many": false
    },
    "rootCauseEntity": {
      "title": "Root Cause Entity",
      "target": "dynatraceEntity",
      "required": false,
      "many": false
    }
  },
  "mirrorProperties": {},
  "calculationProperties": {}
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: problem
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .problemId
          title: .title
          blueprint: '"dynatraceProblem"'
          properties:
            entityTags: .entityTags[].stringRepresentation
            evidenceDetails: .evidenceDetails.details[].displayName
            managementZones: .managementZones[].name
            problemFilters: .problemFilters[].name
            severityLevel: .severityLevel
            status: .status
            startTime: ".startTime / 1000 | todate"
            endTime: ".endTime | if . == -1 then null else (./1000 | todate) end"
          relations:
            impactedEntities: .impactedEntities[].entityId.id
            linkedProblemInfo: .linkedProblemInfo.problemId
            rootCauseEntity: .rootCauseEntity.name
```

</details>

### SLO

<details>
<summary> SLO blueprint</summary>

```json showlineNumbers
{
  "identifier": "dynatraceSlo",
  "description": "This blueprint represents a Dynatrace SLO",
  "title": "Dynatrace SLO",
  "icon": "Dynatrace",
  "schema": {
    "properties": {
      "status": {
        "type": "string",
        "title": "Status",
        "description": "The status of the SLO.",
        "enum": ["FAILURE", "WARNING", "SUCCESS"],
        "enumColors": {
          "FAILURE": "red",
          "WARNING": "yellow",
          "SUCCESS": "green"
        }
      },
      "target": {
        "type": "number",
        "title": "Target",
        "description": "The target value of the SLO."
      },
      "enabled": {
        "type": "boolean",
        "title": "Enabled",
        "description": "Whether the SLO is enabled."
      },
      "burnRateMetricKey": {
        "type": "string",
        "title": "Burn Rate Metric Key",
        "description": "The key for the SLO's error budget burn rate func metric."
      },
      "warning": {
        "type": "number",
        "title": "Warning",
        "description": "The warning value of the SLO. At warning state the SLO is still fulfilled but is getting close to failure."
      },
      "error": {
        "type": "number",
        "title": "Error",
        "description": "The error of the SLO calculation. If the value differs from NONE, there is something wrong with the SLO calculation."
      },
      "errorBudget": {
        "type": "number",
        "title": "Error Budget",
        "description": "The error budget of the calculated SLO."
      },
      "errorBudgetBurnRate": {
        "type": "number",
        "title": "Error Budget Burn Rate",
        "description": "The error budget burn rate of the SLO."
      },
      "errorBudgetMetricKey": {
        "type": "string",
        "title": "Error Budget Metric Key",
        "description": "The key for the SLO's error budget func metric."
      },
      "evaluatedPercentage": {
        "type": "number",
        "title": "Evaluated Percentage",
        "description": "The calculated status value of the SLO."
      },
      "evaluationType": {
        "type": "string",
        "title": "Evaluation Type",
        "description": "The type of the SLO evaluation."
      },
      "filter": {
        "type": "string",
        "title": "Filter",
        "description": "The filter for the SLO evaluation."
      },
      "metricExpression": {
        "type": "string",
        "title": "Metric Expression",
        "description": "The percentage-based metric expression for the calculation of the SLO."
      },
      "metricKey": {
        "type": "string",
        "title": "Metric Key",
        "description": "The key for the SLO's status func metric."
      },
      "normalizedErrorBudgetMetricKey": {
        "type": "string",
        "title": "Normalized Error Budget Metric Key",
        "description": "The key for the SLO's normalized error budget func metric."
      },
      "relatedOpenProblems": {
        "type": "integer",
        "title": "Related Open Problems",
        "description": "Number of open problems related to the SLO. Has the value of -1 if there's an error with fetching SLO related problems."
      },
      "relatedTotalProblems": {
        "type": "integer",
        "title": "Related Total Problems",
        "description": "Number of total problems related to the SLO. Has the value of -1 if there's an error with fetching SLO related problems."
      },
      "timeframe": {
        "type": "string",
        "title": "Timeframe",
        "description": "The timeframe for the SLO evaluation."
      }
    }
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
  - kind: slo
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id
          title: .name
          blueprint: '"dynatraceSlo"'
          properties:
            status: .status
            target: .target
            enabled: .enabled
            burnRateMetricKey: .burnRateMetricKey
            warning: .warning
            error: .error
            errorBudget: .errorBudget
            errorBudgetBurnRate: .errorBudgetBurnRate.burnRateValue
            errorBudgetMetricKey: .errorBudgetMetricKey
            evaluatedPercentage: .evaluatedPercentage
            evaluationType: .evaluationType
            filter: .filter
            metricExpression: .metricExpression
            metricKey: .metricKey
            normalizedErrorBudgetMetricKey: .normalizedErrorBudgetMetricKey
            relatedOpenProblems: .relatedOpenProblems
            relatedTotalProblems: .relatedTotalProblems
            timeframe: .timeframe
```

</details>