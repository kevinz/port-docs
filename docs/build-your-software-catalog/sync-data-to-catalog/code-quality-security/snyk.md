import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'
import SnykBlueprint from "../webhook/examples/resources/snyk/_example_snyk_vulnerability_blueprint.mdx";
import SnykConfiguration from "../webhook/examples/resources/snyk/_example_snyk_vulnerability_webhook_configuration.mdx";

# Snyk

通过我们的 Snyk 集成，您可以根据您的映射和定义，将 Snyk 账户中的 "组织"、"目标"、"项目 "和 "问题 "导入 Port。

## 常见被用于情况

* 在您的 Snyk 环境中映射 "组织"、"目标"、"项目 "和 "问题"。
* 实时观察对象更改(创建/更新/删除)，并自动将更改应用到 Port 中的实体。
* 使用自助操作创建/删除 Snyk 对象。

## 先决条件

<Prerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                           | Description                                                                                                        | Required |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------- |
| `port.clientId`                     | Your Port client id                                                                                                | ✅       |
| `port.clientSecret`                 | Your Port client secret                                                                                            | ✅       |
| `port.baseUrl`                      | Your Port base url, relevant only if not using the default Port app                                                | ❌       |
| `integration.identifier`            | Change the identifier to describe your integration                                                                 | ✅       |
| `integration.type`                  | The integration type                                                                                               | ✅       |
| `integration.eventListener.type`    | The event listener type                                                                                            | ✅       |
| `integration.secrets.token`         | The Snyk API token                                                                                                 | ✅       |
| `integration.config.organizationId` | The Snyk organization ID. Fetches data for this organization when provided                                                                     |❌       |
| `integration.config.groups` | A comma-separated list of Snyk group ids to filter data for. Fetches data for organizations within the specified groups                                                          |❌       |
| `integration.config.apiUrl`         | The Snyk API URL. If not specified, the default will be https://api.snyk.io                                        | ❌       |
| `integration.config.appHost`        | The host of the Port Ocean app. Used to set up the integration endpoint as the target for Webhooks created in Snyk | ❌       |
| `integration.secret.webhookSecret`  | This is a password you create, that Snyk uses to sign webhook events to Port                                       | ❌       |
| `scheduledResyncInterval`           | The number of minutes between each resync                                                                          | ❌       |
| `initializePortResources`           | Default true, When set to true the integration will create default blueprints and the port App config Mapping      | ❌       |


<br/>

<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm">

默认情况下，集成会获取与所提供的 Snyk 令牌相关联的所有组织。 如果希望自定义访问权限，可使用以下参数: 

integration.config.organizationId`: 使用此参数可限制对特定组织的访问。 如果指定，集成将只获取 Provider 所提供组织的数据。

integration.config.groups": 当您想限制对特定 Snyk 组内所有组织的访问时，请使用此参数。 提供以逗号分隔的 Snyk 组 ID 列表，集成将据此过滤数据。

:::note  默认行为 如果两个参数都未提供，集成将按照默认行为运行，即获取与提供的 Snyk 令牌相关联的所有组织。

:::

要使用 Helm 安装集成，请运行以下命令: 

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-snyk-integration port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set port.baseUrl="https://api.getport.io"  \
    --set initializePortResources=true  \
    --set scheduledResyncInterval=120 \
    --set integration.identifier="my-snyk-integration"  \
    --set integration.type="snyk"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.secrets.token="SNYK_TOKEN"
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在 git 仓库的`argocd/my-ocean-snyk-integration`中创建内容为`values.yaml`的文件: 

:::note  默认行为 默认情况下，集成会获取与 Provider Snyk 令牌相关联的所有组织。

记住要替换 `SNYK_TOKEN` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-snyk-integration
  type: snyk
  eventListener:
    type: POLLING
  secrets:
  // highlight-next-line
    token: SNYK_TOKEN
```

<br/>

如果您希望自定义访问权限，可以使用以下配置: 

* `organizationId` 键被用于来限制对特定组织的访问。如果在 `values.yaml` 文件中指定，集成将只获取 Providers 组织的数据。

:::note  配置变量替换 请记住替换 `SNYK_TOKEN` 和 `SNYK_ORGANIZATION_ID` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-snyk-integration
  type: snyk
  eventListener:
    type: POLLING
  config:
  // highlight-next-line
    organizationId: SNYK_ORGANIZATION_ID
  secrets:
    token: SNYK_TOKEN
```

<br/>

* groups "键被用于来限制对特定 Snyk 组内所有组织的访问。在 `values.yaml` 文件中，为 `groups` 键提供一个以逗号分隔的 Snyk 组 ID 列表，集成将过滤组内所有组织的数据。

:::note  配置变量替换 请记住替换 `SNYK_TOKEN` 和 `SNYK_GROUPS` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-snyk-integration
  type: snyk
  eventListener:
    type: POLLING
  config:
  // highlight-next-line
    groups: SNYK_GROUPS
  secrets:
    token: SNYK_TOKEN
```

<br/>

2.创建下面的 "my-ocean-snyk-integration.yaml "配置清单，安装 "my-ocean-snyk-integration "ArgoCD应用程序: 

:::note  配置变量替换 请记住替换 `YOUR_PORT_CLIENT_ID` `YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-snyk-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-snyk-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-snyk-integration/values.yaml
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
kubectl apply -f my-ocean-snyk-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">

默认情况下，集成会获取与所提供的 Snyk 令牌相关联的所有组织。 如果希望自定义访问权限，可使用以下参数: 

OCEAN__INTEGRATION__CONFIG__ORGANIZATION_ID`: 使用此参数可限制对特定组织的访问。 如果被用于，集成将只获取所提供组织的数据。

OCEAN__INTEGRATION__CONFIG__GROUPS": 当您想限制对特定 Snyk 组内所有组织的访问时，请使用此参数。 被用于的 Snyk 组 ID 是一个逗号分隔的列表，集成将据此过滤数据。

  :::note 默认行为 
  如果两个参数都未提供，集成将以默认行为运行，即获取与所提供的 Snyk 令牌相关的所有操作符: 
  :::
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Snyk integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 


| Parameter                                     | Description                                                                                                        | Required |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------- |
| `OCEAN__INTEGRATION__CONFIG__TOKEN`           | The Snyk API token                                                                                                 | ✅       |
| `OCEAN__INTEGRATION__CONFIG__ORGANIZATION_ID` | The Snyk organization ID. Provide this parameter to limit access to a specific organization.                                                                                       | ❌      |
| `OCEAN__INTEGRATION__CONFIG__GROUPS` | A comma-separated list of Snyk group ids to filter data for. Provide this parameter to limit access to all organizations within specific group(s)                                                   | ❌      |
| `OCEAN__INTEGRATION__CONFIG__API_URL`         | The Snyk API URL. If not specified, the default will be https://api.snyk.io                                        | ❌       |
| `OCEAN__INITIALIZE_PORT_RESOURCES`            | Default true, When set to false the integration will not create default blueprints and the port App config Mapping | ❌       |
| `OCEAN__INTEGRATION__IDENTIFIER`              | Change the identifier to describe your integration, if not set will use the default one                            | ❌       |
| `OCEAN__PORT__CLIENT_ID`                      | Your port client id                                                                                                | ✅       |
| `OCEAN__PORT__CLIENT_SECRET`                  | Your port client secret                                                                                            | ✅       |
| `OCEAN__PORT__BASE_URL`                       | Your port base url, relevant only if not using the default port app                                                | ❌       |


<br/>

下面是 `snyk-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Snyk Exporter Workflow

# This workflow responsible for running Snyk exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Snyk Integration
        run: |
          # Set Docker image and run the container
          integration_type="snyk"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__TOKEN }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

</TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Snyk integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/)的 "Secret Text "类型: 


| Parameter                                     | Description                                                                                                                                                      | Required |
| --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| `OCEAN__INTEGRATION__CONFIG__TOKEN`           | The Snyk API token                                                                                                                                               | ✅       |
| `OCEAN__INTEGRATION__CONFIG__ORGANIZATION_ID` | The Snyk organization ID. Provide this parameter to limit access to a specific organization | ❌  |
| `OCEAN__INTEGRATION__CONFIG__GROUPS` | A comma-separated list of Snyk group ids to filter data for. Provide this parameter to limit access to all organizations within specific group(s)                                       | ❌      |
| `OCEAN__INTEGRATION__CONFIG__API_URL`         | The Snyk API URL. If not specified, the default will be https://api.snyk.io                                                                                      | ❌       |
| `OCEAN__INITIALIZE_PORT_RESOURCES`            | Default true, When set to false the integration will not create default blueprints and the port App config Mapping                                               | ❌       |
| `OCEAN__INTEGRATION__IDENTIFIER`              | Change the identifier to describe your integration, if not set will use the default one                                                                          | ❌       |
| `OCEAN__PORT__CLIENT_ID`                      | Your port client id ([How to get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials))     | ✅       |
| `OCEAN__PORT__CLIENT_SECRET`                  | Your port client ([How to get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)) secret | ✅       |
| `OCEAN__PORT__BASE_URL`                       | Your port base url, relevant only if not using the default port app                                                                                              | ❌       |


<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

```text showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run Snyk Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__TOKEN'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="snyk"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__TOKEN=$OCEAN__INTEGRATION__CONFIG__TOKEN \
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
This pipeline will run the Snyk integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Azure Devops 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

确保使用[Azure Devops variable groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&amp;tabs=yaml) 配置以下变量。将它们添加到名为 `port-ocean-credentials` 的变量组中: 


| Parameter                                     | Description                                                                                                                                                      | Required |
| --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| `OCEAN__INTEGRATION__CONFIG__TOKEN`           | The Snyk API token                                                                                                                                               | ✅       |
| `OCEAN__INTEGRATION__CONFIG__ORGANIZATION_ID` | The Snyk organization ID. Provide this parameter to limit access to a specific organization | ❌  |
| `OCEAN__INTEGRATION__CONFIG__GROUPS` | A comma-separated list of Snyk group ids to filter data for. Provide this parameter to limit access to all organizations within specific group(s)                                       | ❌      |
| `OCEAN__INTEGRATION__CONFIG__API_URL`         | The Snyk API URL. If not specified, the default will be https://api.snyk.io                                                                                      | ❌       |
| `OCEAN__INITIALIZE_PORT_RESOURCES`            | Default true, When set to false the integration will not create default blueprints and the port App config Mapping                                               | ❌       |
| `OCEAN__INTEGRATION__IDENTIFIER`              | Change the identifier to describe your integration, if not set will use the default one                                                                          | ❌       |
| `OCEAN__PORT__CLIENT_ID`                      | Your port client id ([How to get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials))     | ✅       |
| `OCEAN__PORT__CLIENT_SECRET`                  | Your port client ([How to get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)) secret | ✅       |
| `OCEAN__PORT__BASE_URL`                       | Your port base url, relevant only if not using the default port app                                                                                              | ❌       |


<br/>

下面是 `snyk-integration.yml` Pipelines 文件的示例: 

```yaml showLineNumbers
trigger:
- main

pool:
  vmImage: "ubuntu-latest"

variables:
  - group: port-ocean-credentials # OCEAN__PORT__CLIENT_ID, OCEAN__PORT__CLIENT_SECRET, OCEAN__INTEGRATION__CONFIG__TOKEN

steps:
- script: |
    echo Add other tasks to build, test, and deploy your project.
    # Set Docker image and run the container
    integration_type="snyk"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
    -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
    -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
    -e OCEAN__INTEGRATION__CONFIG__TOKEN=${OCEAN__INTEGRATION__CONFIG__TOKEN} \
    -e OCEAN__PORT__CLIENT_ID=${OCEAN__PORT__CLIENT_ID} \
    -e OCEAN__PORT__CLIENT_SECRET=${OCEAN__PORT__CLIENT_SECRET} \
    $image_name

    exit $?
  displayName: 'Ingest Synk Data into Port'
```

  </TabItem>
  </Tabs>
</TabItem>

</Tabs>

<AdvancedConfig/>

## 接收 Snyk 对象

Snyk 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 Snyk 获取 "项目 "数据的过程: 

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: project
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id
          title: .attributes.name
          blueprint: '"snykProject"'
          properties:
            url: ("https://app.snyk.io/org/" + .relationships.organization.data.id + "/project/" + .id | tostring)
            owner: .__owner.email
            businessCriticality: .attributes.business_criticality
            environment: .attributes.environment
            lifeCycle: .attributes.lifecycle
            highOpenVulnerabilities: .meta.latest_issue_counts.high
            mediumOpenVulnerabilities: .meta.latest_issue_counts.medium
            lowOpenVulnerabilities: .meta.latest_issue_counts.low
            criticalOpenVulnerabilities: .meta.latest_issue_counts.critical
            importedBy: .__importer.email
            tags: .attributes.tags
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 Snyk API 事件的现有字段和 Values 进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了将从 Snyk 查询哪些资源，以及将在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被用于来映射来自 Snyk 的数据，可以引用出现在下面链接的 API 响应中的任何字段来进行映射配置。

* * [`Organization`](https://snyk.docs.apiary.io/#reference/organizations/the-snyk-organization-for-a-request/list-all-the-organizations-a-user-belongs-to)
* [`Target`](https://apidocs.snyk.io/?version=2023-08-21%7Ebeta#get-/orgs/-org_id-/targets)
* [`Project`](https://apidocs.snyk.io/?version=2023-08-21#get-/orgs/-org_id-/projects)
* [`Issue`](https://snyk.docs.apiary.io/#reference/projects/aggregated-project-issues/list-all-aggregated-issues)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: project
      selector:
      ...
  ```


* 类型 "键是 Snyk 对象的指定符: 


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


* Port"、"实体 "和 "映射 "键被用来将 Snyk 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: project
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one Snyk object to a Port entity. Each value is a JQ query.
            identifier: .id
            title: .attributes.name
            blueprint: '"snykProject"'
            properties:
              url: ("https://app.snyk.io/org/" + .relationships.organization.data.id + "/project/" + .id | tostring)
              owner: .__owner.email
              businessCriticality: .attributes.business_criticality
              environment: .attributes.environment
              lifeCycle: .attributes.lifecycle
              highOpenVulnerabilities: .meta.latest_issue_counts.high
              mediumOpenVulnerabilities: .meta.latest_issue_counts.medium
              lowOpenVulnerabilities: .meta.latest_issue_counts.low
              criticalOpenVulnerabilities: .meta.latest_issue_counts.critical
              importedBy: .__importer.email
              tags: .attributes.tags
        # highlight-end
    - kind: project # In this instance project is mapped again with a different filter
      selector:
        query: '.name == "MyProjectName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): 
::: 

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Snyk 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 Snyk 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在 "代码质量和安全 Provider "类别下选择 Snyk。
5. 根据您的需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

#### 组织

<details>
<summary>Organization blueprint</summary>

```json showLineNumbers
{
  "identifier": "snykOrganization",
  "title": "Snyk Organization",
  "icon": "Snyk",
  "schema": {
    "properties": {
      "url": {
        "type": "string",
        "title": "URL",
        "format": "url",
        "icon": "Snyk"
      },
      "slug": {
        "type": "string",
        "title": "Slug"
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
- kind: organization
  selector:
    query: 'true'
  port:
    entity:
      mappings:
        identifier: .id
        title: .name
        blueprint: '"snykOrganization"'
        properties:
          slug: .slug
          url: ("https://app.snyk.io/org/" + .slug | tostring)
```

</details>

### 目标

<details>
<summary>Target blueprint</summary>

```json showLineNumbers
{
  "identifier": "snykTarget",
  "title": "Snyk Target",
  "icon": "Snyk",
  "schema": {
    "properties": {
      "criticalOpenVulnerabilities": {
        "icon": "Vulnerability",
        "type": "number",
        "title": "Open Critical Vulnerabilities"
      },
      "highOpenVulnerabilities": {
        "icon": "Vulnerability",
        "type": "number",
        "title": "Open High Vulnerabilities"
      },
      "mediumOpenVulnerabilities": {
        "icon": "Vulnerability",
        "type": "number",
        "title": "Open Medium Vulnerabilities"
      },
      "lowOpenVulnerabilities": {
        "icon": "Vulnerability",
        "type": "number",
        "title": "Open Low Vulnerabilities"
      },
      "origin": {
        "title": "Target Origin",
        "type": "string",
        "enum": [
          "artifactory-cr",
          "aws-config",
          "aws-lambda",
          "azure-functions",
          "azure-repos",
          "bitbucket-cloud",
          "bitbucket-server",
          "cli",
          "cloud-foundry",
          "digitalocean-cr",
          "docker-hub",
          "ecr",
          "gcr",
          "github",
          "github-cr",
          "github-enterprise",
          "gitlab",
          "gitlab-cr",
          "google-artifact-cr",
          "harbor-cr",
          "heroku",
          "ibm-cloud",
          "kubernetes",
          "nexus-cr",
          "pivotal",
          "quay-cr",
          "terraform-cloud"
        ]
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
- kind: target
  selector:
    query: "true"
  port:
    entity:
      mappings:
        identifier: .attributes.displayName
        title: .attributes.displayName
        blueprint: '"snykTarget"'
        properties:
          origin: .attributes.origin
          highOpenVulnerabilities: "[.__projects[].meta.latest_issue_counts.high] | add"
          mediumOpenVulnerabilities: "[.__projects[].meta.latest_issue_counts.medium] | add"
          lowOpenVulnerabilities: "[.__projects[].meta.latest_issue_counts.low] | add"
          criticalOpenVulnerabilities: "[.__projects[].meta.latest_issue_counts.critical] | add"
```

</details>

### 项目

<details>
<summary>Project blueprint</summary>

```json showLineNumbers
{
  "identifier": "snykProject",
  "title": "Snyk Project",
  "icon": "Snyk",
  "schema": {
    "properties": {
      "url": {
        "type": "string",
        "title": "URL",
        "format": "url",
        "icon": "Snyk"
      },
      "owner": {
        "type": "string",
        "title": "Owner",
        "format": "user",
        "icon": "TwoUsers"
      },
      "businessCriticality": {
        "title": "Business Criticality",
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "critical",
            "high",
            "medium",
            "low"
          ]
        },
        "icon": "DefaultProperty"
      },
      "environment": {
        "items": {
          "type": "string",
          "enum": [
            "frontend",
            "backend",
            "internal",
            "external",
            "mobile",
            "saas",
            "onprem",
            "hosted",
            "distributed"
          ]
        },
        "icon": "Environment",
        "title": "Environment",
        "type": "array"
      },
      "lifeCycle": {
        "title": "Life Cycle",
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "development",
            "sandbox",
            "production"
          ]
        },
        "icon": "DefaultProperty"
      },
      "highOpenVulnerabilities": {
        "icon": "Vulnerability",
        "type": "number",
        "title": "Open High Vulnerabilities"
      },
      "mediumOpenVulnerabilities": {
        "icon": "Vulnerability",
        "type": "number",
        "title": "Open Medium Vulnerabilities"
      },
      "lowOpenVulnerabilities": {
        "icon": "Vulnerability",
        "type": "number",
        "title": "Open Low Vulnerabilities"
      },
      "importedBy": {
        "icon": "TwoUsers",
        "type": "string",
        "title": "Imported By",
        "format": "user"
      },
      "tags": {
        "type": "array",
        "title": "Tags",
        "icon": "DefaultProperty"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "snykVulnerabilities": {
      "title": "Snyk Vulnerabilities",
      "target": "snykVulnerability",
      "required": false,
      "many": true
    },
    "snykOrganization": {
      "title": "Snyk Organization",
      "target": "snykOrganization",
      "required": true,
      "many": false
    }
  }
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
- kind: project
  selector:
    query: 'true'
  port:
    entity:
      mappings:
        identifier: .id
        title: .attributes.name
        blueprint: '"snykProject"'
        properties:
          url: ("https://app.snyk.io/org/" + .relationships.organization.data.id + "/project/" + .id | tostring)
          owner: .__owner.email
          businessCriticality: .attributes.business_criticality
          environment: .attributes.environment
          lifeCycle: .attributes.lifecycle
          highOpenVulnerabilities: .meta.latest_issue_counts.high
          mediumOpenVulnerabilities: .meta.latest_issue_counts.medium
          lowOpenVulnerabilities: .meta.latest_issue_counts.low
          criticalOpenVulnerabilities: .meta.latest_issue_counts.critical
          importedBy: .__importer.email
          tags: .attributes.tags
        relations:
          snykVulnerabilities: '[.__issues[] | select(.issueType == "vuln").issueData.id]'
          snykOrganization: .relationships.organization.data.id
```

</details>

### 漏洞

<details>
<summary>Vulnerability blueprint</summary>

```yaml showLineNumbers
{
  "identifier": "snykVulnerability",
  "title": "Snyk Vulnerability",
  "icon": "Snyk",
  "schema": {
    "properties": {
      "score": {
        "icon": "Star",
        "type": "number",
        "title": "Score"
      },
      "packageName": {
        "type": "string",
        "title": "Package Name",
        "icon": "DefaultProperty"
      },
      "packageVersions": {
        "icon": "Package",
        "title": "Package Versions",
        "type": "array"
      },
      "type": {
        "type": "string",
        "title": "Type",
        "enum": [
          "vuln",
          "license",
          "configuration"
        ],
        "icon": "DefaultProperty"
      },
      "severity": {
        "icon": "Alert",
        "title": "Issue Severity",
        "type": "string",
        "enum": [
          "low",
          "medium",
          "high",
          "critical"
        ],
        "enumColors": {
          "low": "green",
          "medium": "yellow",
          "high": "red",
          "critical": "red"
        }
      },
      "url": {
        "icon": "Link",
        "type": "string",
        "title": "Issue URL",
        "format": "url"
      },
      "language": {
        "type": "string",
        "title": "Language",
        "icon": "DefaultProperty"
      },
      "publicationTime": {
        "type": "string",
        "format": "date-time",
        "title": "Publication Time",
        "icon": "DefaultProperty"
      },
      "isPatched": {
        "type": "boolean",
        "title": "Is Patched",
        "icon": "DefaultProperty"
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
- kind: issue
  selector:
    query: '.issueType == "vuln"'
  port:
    entity:
      mappings:
        identifier: .issueData.id
        title: .issueData.title
        blueprint: '"snykVulnerability"'
        properties:
          score: .priorityScore
          packageName: .pkgName
          packageVersions: .pkgVersions
          type: .issueType
          severity: .issueData.severity
          url: .issueData.url
          language: .issueData.language // .issueType
          publicationTime: .issueData.publicationTime
          isPatched: .isPatched
```

</details>

## 通过 webhook 进行替代安装

虽然上述 Ocean 集成是推荐的安装方法，但您可能更喜欢使用 webhook 从 Snyk 引用数据。 如果是这样，请使用以下说明: 

<details>

<summary><b>Webhook installation (click to expand)</b></summary>

在本示例中，您将在[Snyk](https://snyk.io/) 和 Port 之间创建 Webhook 集成，将 Snyk 代码和基础架构漏洞实体导入 Port。

<h3>Port configuration</h3>

创建以下蓝图定义: 

<details>
<summary>Snyk vulnerability blueprint</summary>

<SnykBlueprint/>

</details>

创建以下 webhook 配置[using Port UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>Snyk vulnerability webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Snyk Mapper`；
    2.标识符 : `snyk_mapper`；
    3.Description : `将 Snyk 漏洞映射到 Port` 的 webhook 配置；
    4.图标 : `Snyk`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <SnykConfiguration/>
3.向下滚动到 **高级设置**，输入以下详细信息: 
    1. secret: `WEBHOOK_SECRET`；
    2.签名头名称:  `x-hub-signature`；
    3.签名算法: 从下拉选项中选择 `sha256`；
    4.签名前缀 : `sha256=`.
    5.点击页面底部的**保存**。

切记将 `WEBHOOK_SECRET` 替换为在 Snyk 中创建 webhook 时指定的真实secret。

</details>

<h3>Create a webhook in Snyk</h3>

1. 进入[Snyk](https://snyk.io/) ，选择要配置 webhook 的账户；
2. 点击页面左侧的**设置**，在**组织 ID** 部分下复制您的组织 ID；
3. 导航至[Snyk accounts page](https://snyk.io/account/) 并复制您的 API 令牌。您将使用此值授权 REST API；
4. 打开任何 REST API 客户端(如 POSTMAN)，进行以下 API 调用以创建 webhook: 
    1. `API URL` - 被用于 https://api.snyk.io/v1/org/`YOUR_ORG_ID`/webhooks；
    2. `Method` - 选择 POST
    3. `Authorization` - API 标记应在授权标头中以 `Authorization: token YOUR_API_KEY` 的形式提供；
    4. `Request Body` - 请求正文应为 JSON 格式。在正文中填写以下信息


   ```json
   {
     "url": "https://ingest.getport.io/<YOUR_PORT_WEBHOOK_KEY>",
     "secret": "WEBHOOK_SECRET"
   }
   ```

5. Click **Send** to create your Snyk webhook;

:::note 您也可以使用下面的 `curl` 命令创建 Snyk webhook: 

```curl showLineNumbers
curl -X POST \
     -H "Authorization: token YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"url": "https://ingest.getport.io/<YOUR_PORT_WEBHOOK_KEY>", "secret": "WEBHOOK_SECRET"}' \
     https://api.snyk.io/v1/org/<YOUR_ORG_ID>/webhooks
```

:::

完成！在源代码中检测到的任何漏洞都会触发一个 webhook 事件，并将其发送到 Port 提供的 webhook URL。 Port 将根据映射解析这些事件，并相应地更新目录实体。

</details>
