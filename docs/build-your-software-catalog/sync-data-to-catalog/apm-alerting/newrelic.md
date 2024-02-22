import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import DockerParameters from "./_newrelic-docker-parameters.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'

# 新 Relic

通过我们的 New Relic 集成，您可以根据您的映射和定义，将 New Relic 云账户中的 "实体 "和 "问题 "导入 Port。

实体 "可以是主机、应用程序、服务、数据库或任何其他向 New Relic 发送数据的组件。 问题 "是一组描述症状的相应问题的事件。

## 常见被引用情况

* 将 New Relic 中受监控的应用程序和服务与其当前打开的警报进行映射。
* 关注受监控应用程序上发出的新警报和更新，并自动将它们同步到 Port 中。

## 先决条件

<Prerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

被引用此安装选项意味着集成将能实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                               | Description                                                                                                   | Required |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| `port.clientId`                         | Your port client id                                                                                           | ✅       |
| `port.clientSecret`                     | Your port client secret                                                                                       | ✅       |
| `port.baseUrl`                          | Your port base url, relevant only if not using the default port app                                           | ❌       |
| `integration.identifier`                | Change the identifier to describe your integration                                                            | ✅       |
| `integration.type`                      | The integration type                                                                                          | ✅       |
| `integration.eventListener.type`        | The event listener type                                                                                       | ✅       |
| `integration.secrets.newRelicAPIKey`    | The New Relic API key                                                                                         | ✅       |
| `integration.secrets.newRelicAccountID` | The New Relic account ID                                                                                      | ✅       |
| `scheduledResyncInterval`               | The number of minutes between each resync                                                                     | ❌       |
| `initializePortResources`               | Default true, When set to true the integration will create default blueprints and the port App config Mapping | ❌       |


<br/>
<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

:::note 如果您正在引用 New Relic 的欧盟地区，请在命令中添加以下 flag: 

`-set integration.config.newRelicGraphqlURL="https://api.eu.newrelic.com/graphql"`

:::

```bash showLineNumbers
# The following script will install an Ocean integration at your K8s cluster using helm
# initializePortResources: When set to true the integration will create default blueprints + JQ Mappings
# scheduledResyncInterval: the number of minutes between each resync
# integration.identifier: Change the identifier to describe your integration

helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-newrelic-integration port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set port.baseUrl="https://api.getport.io"  \
    --set initializePortResources=true  \
    --set scheduledResyncInterval=120 \
    --set integration.identifier="my-newrelic-integration"  \
    --set integration.type="newrelic"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.secrets.newRelicAPIKey="<NR_API_KEY>"  \
    --set integration.secrets.newRelicAccountID="<NR_ACCOUNT_ID>"
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在你的 git 仓库的 `argocd/my-ocean-newrelic-integration` 中创建一个 `values.yaml` 文件，内容如下: 

:::note 请记住替换 `NEW_RELIC_API_KEY` 和 `NEW_RELIC_ACCOUNT_ID` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-newrelic-integration
  type: newrelic
  eventListener:
    type: POLLING
  secrets:
  // highlight-start
    newRelicAPIKey: NEW_RELIC_API_KEY
    newRelicAccountID: NEW_RELIC_ACCOUNT_ID
  // highlight-end
```

<br/>

:::note 如果您正在使用 New Relic 的欧盟地区，请将突出显示的代码(GraphQL 配置值)添加到 `values.yaml` 中: 

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-newrelic-integration
  type: newrelic
  eventListener:
    type: POLLING
  // highlight-start
  config:
    newRelicGraphqlURL: https://api.eu.newrelic.com/graphql
  // highlight-end
  secrets:
    newRelicAPIKey: NEW_RELIC_API_KEY
    newRelicAccountID: NEW_RELIC_ACCOUNT_ID
```

:::

2.创建以下 "my-ocean-newrelic-integration.yaml "配置清单，安装 "my-ocean-newrelic-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-newrelic-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-newrelic-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-newrelic-integration/values.yaml
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
kubectl apply -f my-ocean-newrelic-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">
 <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the New Relic integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成能实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />
<br/>

下面是 `newrelic-integration.yml` 工作流程文件的示例: 

:::note 如果您正在使用 New Relic 的欧盟地区，请在 docker 命令中添加以下 flag: 

`-e OCEAN__INTEGRATION__CONFIG__NEW_RELIC_URL=https://api.eu.newrelic.com/graphql`。

:::

```yaml showLineNumbers
name: New Relic Exporter Workflow

# This workflow responsible for running New Relic exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run New Relic Integration
        run: |
          # Set Docker image and run the container
          integration_type="newrelic"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__NEW_RELIC_API_KEY=${{ secrets.OCEAN__INTEGRATION__CONFIG__NEW_RELIC_API_KEY }} \
          -e OCEAN__INTEGRATION__CONFIG__NEW_RELIC_ACCOUNT_ID=${{ secrets.OCEAN__INTEGRATION__CONFIG__NEW_RELIC_ACCOUNT_ID }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the New Relic integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/) 的 "Secret Text "类型: 

<DockerParameters />
<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

:::note 如果您正在使用 New Relic 的欧盟地区，请在 docker 命令中添加以下 flag: 

`-e OCEAN__INTEGRATION__CONFIG__NEW_RELIC_URL=https://api.eu.newrelic.com/graphql`。

:::

```yml showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run New Relic Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__NEW_RELIC_API_KEY', variable: 'OCEAN__INTEGRATION__CONFIG__NEW_RELIC_API_KEY'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__NEW_RELIC_ACCOUNT_ID', variable: 'OCEAN__INTEGRATION__CONFIG__NEW_RELIC_ACCOUNT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="newrelic"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__NEW_RELIC_API_KEY=$OCEAN__INTEGRATION__CONFIG__NEW_RELIC_API_KEY \
                                -e OCEAN__INTEGRATION__CONFIG__NEW_RELIC_ACCOUNT_ID=$OCEAN__INTEGRATION__CONFIG__NEW_RELIC_ACCOUNT_ID \
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
<AzurePremise name="New Relic" />

<DockerParameters />

<br/>

下面是 `newrelic-integration.yml` Pipelines 文件的示例: 

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
    integration_type="newrelic"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
        -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
        -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
        -e OCEAN__INTEGRATION__CONFIG__NEW_RELIC_API_KEY=${OCEAN__INTEGRATION__CONFIG__NEW_RELIC_API_KEY} \
        -e OCEAN__INTEGRATION__CONFIG__NEW_RELIC_ACCOUNT_ID=${OCEAN__INTEGRATION__CONFIG__NEW_RELIC_ACCOUNT_ID} \
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

### 事件监听器

该集成使用轮询方式，每分钟从 Port 中提取一次配置，并检查配置是否有变化。 如果有变化，就会重新同步。

<AdvancedConfig/>

## Ingesting Newrelic objects

Newrelic 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 Newrelic 获取 "问题 "数据的过程: 

```yaml showLineNumbers
resources:
  - kind: newRelicAlert
    selector:
      query: "true"
      newRelicTypes: ["ISSUE"]
    port:
      entity:
        mappings:
          blueprint: '"newRelicAlert"'
          identifier: .issueId
          title: .title[0]
          properties:
            priority: .priority
            state: .state
            sources: .sources
            conditionName: .conditionName
            alertPolicyNames: .policyName
            activatedAt: .activatedAt
          relations:
            newRelicService: .__APPLICATION.entity_guids + .__SERVICE.entity_guids
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 Newrelic API 事件的现有字段和 Values 进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了从 Newrelic 查询哪些资源，以及在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被引用来映射来自 Newrelic 的数据，可以引用下面链接的 API 响应中出现的任何字段来进行映射配置。

* * [`Entity`](https://docs.newrelic.com/docs/new-relic-solutions/new-relic-one/core-concepts/what-entity-new-relic/)
* [`Issue`](https://docs.newrelic.com/docs/alerts-applied-intelligence/new-relic-alerts/get-started/alerts-ai-overview-page/#issues)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: project
      selector:
      ...
  ```


* 类型 "键是 Newrelic 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: project
        selector:
        ...
  ```


* 通过 "选择器 "键，可以筛选出哪些指定 "类型 "的对象将被收录到软件目录中: 


  ```yaml showLineNumbers
  resources:
    - kind: newRelicService
      selector:
        query: "true"
        newRelicTypes: ["SERVICE", "APPLICATION"]
        calculateOpenIssueCount: true
        entityQueryFilter: "type in ('SERVICE','APPLICATION')"
        entityExtraPropertiesQuery: |
          ... on ApmApplicationEntityOutline {
            guid
            name
          }
  ```


* **newRelicTypes** - 将被获取的 Newrelic 实体类型数组。默认值为['服务'、'应用程序']。这与 Newrelic 实体中的类型字段有关。
* **calculateOpenIssueCount:**
    - 布尔值，表示集成是否应计算每个实体的开放问题数量。默认值为 "false"。
    - **注意** - 这可能会导致性能降级，因为集成必须计算每个实体的开放问题数，但不幸的是，New Relic API 并不支持这一点。
* 实体查询过滤器: ** **
    - 将应用于 New Relic API 查询的过滤器。它将放在 New Relic GraphQL API 中`entitySearch`查询的`query`字段内。查询过滤器示例[click here](https://docs.newrelic.com/docs/apis/nerdgraph/examples/nerdgraph-entities-api-tutorial/#search-query) 。
    - 不指定此字段将导致集成获取所有实体并将它们映射到`kind`中定义的蓝图。
    - 经验法则--大多数情况下，"EntityQueryFilter "与 "NewRelicTypes "相同。例如，如果我们想获取所有服务和应用程序，我们将把 `EntityQueryFilter` 设置为 `type in ('SERVICE','APPLICATION')` ，把 `NewRelicTypes` 设置为 `['SERVICE','APPLICATION']`。
* **实体额外属性查询: **
    - 这是一个可选属性，允许为每个 Newrelic 实体定义要获取的额外属性。这将与我们在 Newrelic GraphQL API 中的 "实体搜索 "查询中的 "实体 "部分下请求的默认查询属性连接起来。有关其他查询属性的示例，请访问[click here](https://docs.newrelic.com/docs/apis/nerdgraph/examples/nerdgraph-entities-api-tutorial/#apm-summary) 。
* Port"、"实体 "和 "映射 "键被用来将 Newrelic 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: newRelicAlert
      selector:
        query: "true"
        newRelicTypes: ["ISSUE"]
      port:
        # highlight-start
        entity:
          mappings:
            blueprint: '"newRelicAlert"'
            identifier: .issueId
            title: .title[0]
            properties:
              priority: .priority
              state: .state
              sources: .sources
              conditionName: .conditionName
              alertPolicyNames: .policyName
              activatedAt: .activatedAt
            relations:
              newRelicService: .__APPLICATION.entity_guids + .__SERVICE.entity_guids
        # highlight-end
    - kind: newRelicAlert # In this instance project is mapped again with a different filter
      selector:
        query: '.name == "MyIssuetName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): ::: 

### 标签

一些 Newrelic "实体 "有一个名为 "标签 "的属性，其中包含机器信息、主机名、代理名称和版本等可能有用的信息。 例如: 

```json showLineNumbers
"tags": [
  {
    "key": "coreCount",
    "values": [
      "10"
    ]
  },
  {
    "key": "hostStatus",
    "values": [
      "running"
    ]
  },
]
```

在映射之前，该集成会对每个 "标签 "进行转换，转换之后，上面的示例就会变成这样: 

```json showLineNumbers
tags = ["coreCount":"10","hostStatus":"running"]
```

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Newrelic 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 Newrelic 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在 APM 和警报类别下选择 Newrelic。
5. 将[integration configuration](#configuration-structure) 的内容添加到编辑器中。
6. 单击 "Resync"。

## 示例

蓝图和相关集成配置示例: 

#### 服务(实体)

<details>
<summary>Service blueprint</summary>

```json showLineNumbers
{
  "identifier": "newRelicService",
  "description": "This blueprint represents a New Relic service or application in our software catalog",
  "title": "New Relic Service",
  "icon": "NewRelic",
  "schema": {
    "properties": {
      "has_apm": {
        "title": "Has APM",
        "type": "boolean"
      },
      "open_issues_count": {
        "title": "Open Issues Count",
        "type": "number",
        "default": 0
      },
      "link": {
        "title": "Link",
        "type": "string",
        "format": "url"
      },
      "reporting": {
        "title": "Reporting",
        "type": "boolean"
      },
      "tags": {
        "title": "Tags",
        "type": "object"
      },
      "account_id": {
        "title": "Account ID",
        "type": "string"
      },
      "type": {
        "title": "Type",
        "type": "string"
      },
      "domain": {
        "title": "Domain",
        "type": "string"
      },
      "throughput": {
        "title": "Throughput",
        "type": "number"
      },
      "response_time_avg": {
        "title": "Response Time AVG",
        "type": "number"
      },
      "error_rate": {
        "title": "Error Rate",
        "type": "number"
      },
      "instance_count": {
        "title": "Instance Count",
        "type": "number"
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
- kind: newRelicService
  selector:
    query: "true"
    newRelicTypes: ["SERVICE", "APPLICATION"]
    calculateOpenIssueCount: true
    entityQueryFilter: "type in ('SERVICE','APPLICATION')"
    entityExtraPropertiesQuery: |
      ... on ApmApplicationEntityOutline {
        guid
        name
        alertSeverity
        applicationId
        apmBrowserSummary {
          ajaxRequestThroughput
          ajaxResponseTimeAverage
          jsErrorRate
          pageLoadThroughput
          pageLoadTimeAverage
        }
        apmSummary {
          apdexScore
          errorRate
          hostCount
          instanceCount
          nonWebResponseTimeAverage
          nonWebThroughput
          responseTimeAverage
          throughput
          webResponseTimeAverage
          webThroughput
        }
      }
  port:
    entity:
      mappings:
        blueprint: '"newRelicService"'
        identifier: .guid
        title: .name
        properties:
          has_apm: 'if .domain | contains("APM") then "true" else "false" end'
          link: .permalink
          open_issues_count: .__open_issues_count
          reporting: .reporting
          tags: .tags
          domain: .domain
          type: .type
```

</details>

### 问题

<details>
<summary>Issue blueprint</summary>

```json showLineNumbers
{
  "identifier": "newRelicAlert",
  "description": "This blueprint represents a New Relic alert in our software catalog",
  "title": "New Relic Alert",
  "icon": "NewRelic",
  "schema": {
    "properties": {
      "priority": {
        "type": "string",
        "title": "Priority",
        "enum": ["CRITICAL", "HIGH", "MEDIUM", "LOW"],
        "enumColors": {
          "CRITICAL": "red",
          "HIGH": "red",
          "MEDIUM": "yellow",
          "LOW": "green"
        }
      },
      "state": {
        "type": "string",
        "title": "State",
        "enum": ["ACTIVATED", "CLOSED", "CREATED"],
        "enumColors": {
          "ACTIVATED": "yellow",
          "CLOSED": "green",
          "CREATED": "lightGray"
        }
      },
      "trigger": {
        "type": "string",
        "title": "Trigger"
      },
      "sources": {
        "type": "array",
        "title": "Sources"
      },
      "alertPolicyNames": {
        "type": "array",
        "title": "Alert Policy Names"
      },
      "conditionName": {
        "type": "array",
        "title": "Condition Name"
      },
      "activatedAt": {
        "type": "string",
        "title": "Time Issue was activated"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "newRelicService": {
      "title": "New Relic Service",
      "target": "newRelicService",
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
- kind: newRelicAlert
  selector:
    query: "true"
    newRelicTypes: ["ISSUE"]
  port:
    entity:
      mappings:
        blueprint: '"newRelicAlert"'
        identifier: .issueId
        title: .title[0]
        properties:
          priority: .priority
          state: .state
          sources: .sources
          conditionName: .conditionName
          alertPolicyNames: .policyName
          activatedAt: .activatedAt
        relations:
          newRelicService: .__APPLICATION.entity_guids + .__SERVICE.entity_guids
```

</details>