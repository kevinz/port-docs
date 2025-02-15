import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import HelmParameters from "../templates/_ocean-advanced-parameters-helm.mdx"
import DockerParameters from "./_jira_one_time_docker_parameters.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'
import JiraIssueBlueprint from "../webhook/examples/resources/jira/_example_jira_issue_blueprint.mdx"
import JiraIssueConfiguration from "../webhook/examples/resources/jira/_example_jira_issue_configuration.mdx"
import JiraProjectBlueprint from "../webhook/examples/resources/jira-server/_example_jira_project_blueprint.mdx";
import JiraWebhookConfiguration from "../webhook/examples/resources/jira-server/_example_jira_webhook_configuration.mdx";
import JiraIssueConfigurationPython from "../webhook/examples/resources/jira/_example_jira_issue_configuration_python.mdx"
import JiraServerConfigurationPython from "../webhook/examples/resources/jira-server/_example_jira_server_configuration_python.mdx";

# Jira

通过我们的 Jira 集成，您可以根据您的映射和定义，将 Jira 云账户中的 "问题 "和 "项目 "导入 Port。

:::info  仅支持 Jira cloud 此集成目前支持 `Jira Cloud`. 要将 Port 与 `Jira Server` 集成，请使用[Port's webhook integration](/build-your-software-catalog/sync-data-to-catalog/webhook/examples/jira-server) 。

:::

## 常见被用于情况

* 映射 Jira 组织环境中的问题和项目。
* 实时关注对象变更(创建/更新/删除)，并自动将变更应用到 Port 中的实体。
* 使用自助操作创建/删除 Jira 对象。

## 先决条件

<Prerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                                | Description                                                                                                                                | Example                          | Required |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------- | -------- |
| `port.clientId`                          | Your port [client id](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)            |                                  | ✅       |
| `port.clientSecret`                      | Your port [client secret](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)        |                                  | ✅       |
| `integration.secrets.atlassianUserEmail` | The email of the user used to query Jira                                                                                                   | user@example.com                 | ✅       |
| `integration.secrets.atlassianUserToken` | [Jira API token](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/) generated by the user |                                  | ✅       |
| `integration.config.jiraHost`            | The URL of your Jira                                                                                                                       | https://example.atlassian.net    | ✅       |
| `integration.config.appHost`             | The host of the Port Ocean app. Used to set up the integration endpoint as the target for webhooks created in Jira                         | https://my-ocean-integration.com | ❌       |


<HelmParameters/>

<br/>
<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-jira-integration port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set port.baseUrl="https://api.getport.io"  \
    --set initializePortResources=true  \
    --set scheduledResyncInterval=120 \
    --set integration.identifier="my-jira-integration"  \
    --set integration.type="jira"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.config.jiraHost="string"  \
    --set integration.secrets.atlassianUserEmail="string"  \
    --set integration.secrets.atlassianUserToken="string"
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在你的 git 仓库的 `argocd/my-ocean-jira-integration` 中创建一个内容为 `values.yaml` 的文件: 

:::note 请记住替换 `ATLASSIAN_JIRA_HOST``ATLASSIAN_USER_EMAIL` 和 `ATLASSIAN_USER_TOKEN` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-jira-integration
  type: jira
  eventListener:
    type: POLLING
  config:
  // highlight-next-line
    jiraHost: ATLASSIAN_JIRA_HOST
  secrets:
  // highlight-start
    atlassianUserEmail: ATLASSIAN_USER_EMAIL
    atlassianUserToken: ATLASSIAN_USER_TOKEN
  // highlight-end
```

<br/>

2.创建以下 "my-ocean-jira-integration.yaml "配置清单，安装 "my-ocean-jira-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-jira-integration
  namespace: argocd
spec:
  destination:
    namespace: mmy-ocean-jira-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-jira-integration/values.yaml
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
kubectl apply -f my-ocean-jira-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">

  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Jira integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项。

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters/>

<br/>

下面是 `jira-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Jira Exporter Workflow

# This workflow responsible for running Jira exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Jira Integration
        run: |
          # Set Docker image and run the container
          integration_type="jira"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__JIRA_HOST=${{ secrets.OCEAN__INTEGRATION__CONFIG__JIRA_HOST }} \
          -e OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_EMAIL=${{ secrets.OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_EMAIL }} \
          -e OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_TOKEN }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Jira integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/) 的 "Secret Text "类型: 

<DockerParameters/>

<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

```yml showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run Jira Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__JIRA_HOST', variable: 'OCEAN__INTEGRATION__CONFIG__JIRA_HOST'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_EMAIL', variable: 'OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_EMAIL'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_TOKEN'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="jira"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__JIRA_HOST=$OCEAN__INTEGRATION__CONFIG__JIRA_HOST \
                                -e OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_EMAIL=$OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_EMAIL \
                                -e OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_TOKEN=$OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_TOKEN \
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
<AzurePremise name="Jira" />

<DockerParameters />

<br/>

下面是 `jira-integration.yml` Pipelines 文件的示例: 

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
    integration_type="jira"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
      -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
      -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
      -e OCEAN__INTEGRATION__CONFIG__JIRA_HOST=${OCEAN__INTEGRATION__CONFIG__JIRA_HOST} \
      -e OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_EMAIL=${OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_EMAIL} \
      -e OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_TOKEN=${OCEAN__INTEGRATION__CONFIG__ATLASSIAN_USER_TOKEN} \
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

## 接收 Jira 对象

Jira 集成使用 YAML 配置来描述将数据加载到开发人员门户的过程。

下面是配置中的一个示例片段，演示了从 Jira 获取 "项目 "数据的过程: 

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
          identifier: .key
          title: .name
          blueprint: '"jiraProject"'
          properties:
            url: (.self | split("/") | .[:3] | join("/")) + "/projects/" + .key
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 Jira API 事件的现有字段和 Values 进行选择、修改、连接、转换和其他操作。

:::info  附加参数 在上面的示例中，使用了两个附加参数: `createMissingRelatedEntities`(创建缺失的相关实体)- 用于启用在 Port 中创建缺失的相关实体。 当您想在一次调用中创建一个实体及其相关实体，或者想创建一个其相关实体尚不存在的实体时，这个参数非常有用。

`deleteDependentEntities` - 用于启用从属 Port 实体的删除。 当您有两个具有必填关系的蓝图，且关系中的目标实体应被删除时，此参数非常有用。 在这种情况下，如果此参数设置为 `false`，删除操作将失败。 如果设置为 `true`，源实体也将被删除。

:::

### 配置结构

集成配置决定了将从 Jira 查询哪些资源，以及将在 Port 中创建哪些实体和属性。

:::tip  支持的资源 (`Kind`) 以下资源可被用于来映射来自 Jira 的数据，可以引用下面链接的 API 响应中出现的任何字段来进行映射配置。

* * [`Project`](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-projects/#api-rest-api-3-project-search-get)
* [`Issue`](https://developer.atlassian.com/cloud/jira/platform/rest/v3/api-group-issue-search/#api-rest-api-3-search-get)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: project
      selector:
      ...
  ```


* 类型 "键是 Jira 对象的指定符: 


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


:::tip  支持 JQL Ocean Jira 集成支持使用[JQL](https://support.atlassian.com/jira-service-management-cloud/docs/use-advanced-search-with-jira-query-language-jql/) 从 "问题 "类型中查询对象，从而可以专门过滤从 Jira 查询并被导入 Port 的问题。

要使用 JQL 过滤，请在 `selector` 对象中添加一个 `jql` 键，并将所需的 JQL 查询作为值。 例如: 

```yaml showLineNumbers
resources:
      # highlight-next-line
    - kind: issue # JQL filtering can only be used with the "issue" kind
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
      # highlight-next-line
        jql: "status != Done" # JQL query, will only ingest issues whose status is not "Done"
      port:
```

:::

* Port"、"实体 "和 "映射 "键被用来将 Jira 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: project
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one Jira object to a Port entity. Each value is a JQ query.
            identifier: .key
            title: .name
            blueprint: '"jiraProject"'
            properties:
              url: (.self | split("/") | .[:3] | join("/")) + "/projects/" + .key
        # highlight-end
    - kind: project # In this instance project is mapped again with a different filter
      selector:
        query: '.name == "MyProjectName"'
      port:
        entity:
          mappings: ...
  ```


:::tip  Blueprint key 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，则需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再使用一对双引号 (`"`)。

:::

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Jira 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 Jira 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在项目管理 Provider 类别下选择 Jira。
5. 根据需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

### 问题

<details>
<summary>Issue blueprint</summary>

```json showLineNumbers
{
  "identifier": "jiraIssue",
  "title": "Jira Issue",
  "icon": "Jira",
  "schema": {
    "properties": {
      "url": {
        "title": "Issue URL",
        "type": "string",
        "format": "url",
        "description": "URL to the issue in Jira"
      },
      "status": {
        "title": "Status",
        "type": "string",
        "description": "The status of the issue"
      },
      "issueType": {
        "title": "Type",
        "type": "string",
        "description": "The type of the issue"
      },
      "components": {
        "title": "Components",
        "type": "array",
        "description": "The components related to this issue"
      },
      "assignee": {
        "title": "Assignee",
        "type": "string",
        "format": "user",
        "description": "The user assigned to the issue"
      },
      "reporter": {
        "title": "Reporter",
        "type": "string",
        "description": "The user that reported to the issue",
        "format": "user"
      },
      "creator": {
        "title": "Creator",
        "type": "string",
        "description": "The user that created to the issue",
        "format": "user"
      },
      "priority": {
        "title": "Priority",
        "type": "string",
        "description": "The priority of the issue"
      },
      "created": {
        "title": "Created At",
        "type": "string",
        "description": "The created datetime of the issue",
        "format": "date-time"
      },
      "updated": {
        "title": "Updated At",
        "type": "string",
        "description": "The updated datetime of the issue",
        "format": "date-time"
      }
    }
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {
    "project": {
      "target": "jiraProject",
      "title": "Project",
      "description": "The Jira project that contains this issue",
      "required": false,
      "many": false
    },
    "parentIssue": {
      "target": "jiraIssue",
      "title": "Parent Issue",
      "required": false,
      "many": false
    },
    "subtasks": {
      "target": "jiraIssue",
      "title": "Subtasks",
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
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: issue
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .key
          title: .fields.summary
          blueprint: '"jiraIssue"'
          properties:
            url: (.self | split("/") | .[:3] | join("/")) + "/browse/" + .key
            status: .fields.status.name
            issueType: .fields.issuetype.name
            components: .fields.components
            assignee: .fields.assignee.displayName
            reporter: .fields.reporter.displayName
            creator: .fields.creator.displayName
            priority: .fields.priority.id
            created: .fields.created
            updated: .fields.updated
          relations:
            project: .fields.project.key
            parentIssue: .fields.parent.key
            subtasks: .fields.subtasks | map(.key)
```

</details>

### 项目

<details>
<summary>Project blueprint</summary>

```json showLineNumbers
{
  "identifier": "jiraProject",
  "title": "Jira Project",
  "icon": "Jira",
  "description": "A Jira project",
  "schema": {
    "properties": {
      "url": {
        "title": "Project URL",
        "type": "string",
        "format": "url",
        "description": "URL to the project in Jira"
      }
    }
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
  - kind: project
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .key
          title: .name
          blueprint: '"jiraProject"'
          properties:
            url: (.self | split("/") | .[:3] | join("/")) + "/projects/" + .key
```

</details>

## 让我们来测试一下

本节包括来自 Jira 的响应数据示例。 此外，还包括根据上一节提供的 Ocean 配置从重新同步事件中创建的实体。

### 有效载荷

下面是 Jira 提供的有效载荷结构示例: 

<details>
<summary> Project response data</summary>

```json showLineNumbers
{
  "expand": "description,lead,issueTypes,url,projectKeys,permissions,insight",
  "self": "https://myaccount.atlassian.net/rest/api/3/project/10000",
  "id": "10000",
  "key": "PA",
  "name": "Port-AI",
  "avatarUrls": {
    "48x48": "https://myaccount.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10413",
    "24x24": "https://myaccount.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10413?size=small",
    "16x16": "https://myaccount.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10413?size=xsmall",
    "32x32": "https://myaccount.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10413?size=medium"
  },
  "projectTypeKey": "software",
  "simplified": true,
  "style": "next-gen",
  "isPrivate": false,
  "properties": {},
  "entityId": "7f4f8d6f-705b-4074-84be-46f0d012cd8e",
  "uuid": "7f4f8d6f-705b-4074-84be-46f0d012cd8e"
}
```

</details>

<details>
<summary> Issue response data</summary>

```json showLineNumbers
{
  "expand": "operations,versionedRepresentations,editmeta,changelog,customfield_10010.requestTypePractice,renderedFields",
  "id": "10000",
  "self": "https://myaccount.atlassian.net/rest/api/3/issue/10000",
  "key": "PA-1",
  "fields": {
    "statuscategorychangedate": "2023-11-06T11:02:59.341+0000",
    "issuetype": {
      "self": "https://myaccount.atlassian.net/rest/api/3/issuetype/10001",
      "id": "10001",
      "description": "Tasks track small, distinct pieces of work.",
      "iconUrl": "https://myaccount.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10318?size=medium",
      "name": "Task",
      "subtask": false,
      "avatarId": 10318,
      "entityId": "a7309bf9-70c5-4237-bdaf-0261037b6ecc",
      "hierarchyLevel": 0
    },
    "timespent": "None",
    "customfield_10030": "None",
    "project": {
      "self": "https://myaccount.atlassian.net/rest/api/3/project/10000",
      "id": "10000",
      "key": "PA",
      "name": "Port-AI",
      "projectTypeKey": "software",
      "simplified": true,
      "avatarUrls": {
        "48x48": "https://myaccount.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10413",
        "24x24": "https://myaccount.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10413?size=small",
        "16x16": "https://myaccount.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10413?size=xsmall",
        "32x32": "https://myaccount.atlassian.net/rest/api/3/universal_avatar/view/type/project/avatar/10413?size=medium"
      }
    },
    "customfield_10031": "None",
    "customfield_10032": "None",
    "fixVersions": [],
    "aggregatetimespent": "None",
    "resolution": "None",
    "customfield_10027": "None",
    "customfield_10028": "None",
    "customfield_10029": "None",
    "resolutiondate": "None",
    "workratio": -1,
    "watches": {
      "self": "https://myaccount.atlassian.net/rest/api/3/issue/PA-1/watchers",
      "watchCount": 1,
      "isWatching": true
    },
    "lastViewed": "None",
    "created": "2023-11-06T11:02:59.000+0000",
    "customfield_10020": "None",
    "customfield_10021": "None",
    "customfield_10022": "None",
    "priority": {
      "self": "https://myaccount.atlassian.net/rest/api/3/priority/3",
      "iconUrl": "https://myaccount.atlassian.net/images/icons/priorities/medium.svg",
      "name": "Medium",
      "id": "3"
    },
    "customfield_10023": "None",
    "customfield_10024": "None",
    "customfield_10025": "None",
    "labels": ["infra"],
    "customfield_10026": "None",
    "customfield_10016": "None",
    "customfield_10017": "None",
    "customfield_10018": {
      "hasEpicLinkFieldDependency": false,
      "showField": false,
      "nonEditableReason": {
        "reason": "PLUGIN_LICENSE_ERROR",
        "message": "The Parent Link is only available to Jira Premium users."
      }
    },
    "customfield_10019": "0|hzzzzz:",
    "timeestimate": "None",
    "aggregatetimeoriginalestimate": "None",
    "versions": [],
    "issuelinks": [],
    "assignee": {
      "self": "https://myaccount.atlassian.net/rest/api/3/user?accountId=712020%3A05acda87-42da-44d8-b21e-f71a508e5d11",
      "accountId": "712020:05acda87-42da-44d8-b21e-f71a508e5d11",
      "emailAddress": "username@example.com.io",
      "avatarUrls": {
        "48x48": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "24x24": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "16x16": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "32x32": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png"
      },
      "displayName": "User Name",
      "active": true,
      "timeZone": "UTC",
      "accountType": "atlassian"
    },
    "updated": "2023-11-06T11:03:18.244+0000",
    "status": {
      "self": "https://myaccount.atlassian.net/rest/api/3/status/10000",
      "description": "",
      "iconUrl": "https://myaccount.atlassian.net/",
      "name": "To Do",
      "id": "10000",
      "statusCategory": {
        "self": "https://myaccount.atlassian.net/rest/api/3/statuscategory/2",
        "id": 2,
        "key": "new",
        "colorName": "blue-gray",
        "name": "To Do"
      }
    },
    "components": [],
    "timeoriginalestimate": "None",
    "description": "None",
    "customfield_10010": "None",
    "customfield_10014": "None",
    "customfield_10015": "None",
    "customfield_10005": "None",
    "customfield_10006": "None",
    "security": "None",
    "customfield_10007": "None",
    "customfield_10008": "None",
    "aggregatetimeestimate": "None",
    "customfield_10009": "None",
    "summary": "Setup infra",
    "creator": {
      "self": "https://myaccount.atlassian.net/rest/api/3/user?accountId=712020%3A05acda87-42da-44d8-b21e-f71a508e5d11",
      "accountId": "712020:05acda87-42da-44d8-b21e-f71a508e5d11",
      "emailAddress": "username@example.com.io",
      "avatarUrls": {
        "48x48": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "24x24": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "16x16": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "32x32": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png"
      },
      "displayName": "User Name",
      "active": true,
      "timeZone": "UTC",
      "accountType": "atlassian"
    },
    "subtasks": [],
    "reporter": {
      "self": "https://myaccount.atlassian.net/rest/api/3/user?accountId=712020%3A05acda87-42da-44d8-b21e-f71a508e5d11",
      "accountId": "712020:05acda87-42da-44d8-b21e-f71a508e5d11",
      "emailAddress": "username@example.com.io",
      "avatarUrls": {
        "48x48": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "24x24": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "16x16": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png",
        "32x32": "https://secure.gravatar.com/avatar/0d5d34ceb820d324d69046a1b2f51dc0?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-3.png"
      },
      "displayName": "User Name",
      "active": true,
      "timeZone": "UTC",
      "accountType": "atlassian"
    },
    "aggregateprogress": {
      "progress": 0,
      "total": 0
    },
    "customfield_10001": "None",
    "customfield_10002": "None",
    "customfield_10003": "None",
    "customfield_10004": "None",
    "environment": "None",
    "duedate": "None",
    "progress": {
      "progress": 0,
      "total": 0
    },
    "votes": {
      "self": "https://myaccount.atlassian.net/rest/api/3/issue/PA-1/votes",
      "votes": 0,
      "hasVoted": false
    }
  }
}
```

</details>

#### 映射结果

结合样本有效载荷和 Ocean 配置，可生成以下 Port 实体: 

<details>
<summary> Project entity in Port</summary>

```json showLineNumbers
{
  "identifier": "PA",
  "title": "Port-AI",
  "icon": null,
  "blueprint": "jiraProject",
  "team": [],
  "properties": {
    "url": "https://myaccount.atlassian.net/projects/PA"
  },
  "relations": {},
  "createdAt": "2023-11-06T11:22:05.433Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-11-06T11:22:05.433Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>

<details>
<summary> Issue entity in Port</summary>

```json showLineNumbers
{
  "identifier": "PA-1",
  "title": "Setup infra",
  "icon": null,
  "blueprint": "jiraIssue",
  "team": [],
  "properties": {
    "url": "https://myaccount.atlassian.net/browse/PA-1",
    "status": "To Do",
    "issueType": "Task",
    "components": [],
    "assignee": "User Name",
    "reporter": "User Name",
    "creator": "User Name",
    "priority": "3",
    "created": "2023-11-06T11:02:59.000+0000",
    "updated": "2023-11-06T11:03:18.244+0000"
  },
  "relations": {
    "parentIssue": null,
    "project": "PA",
    "subtasks": []
  },
  "createdAt": "2023-11-06T11:22:07.550Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-11-06T11:22:07.550Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>

## 通过 webhook 进行替代安装

虽然上述 Ocean 集成是推荐的安装方法，但您可能更喜欢使用 webhook 从 Jira 引用数据。 如果是这样，请使用以下说明: 

<details>

<summary><b>Webhook installation (click to expand)</b></summary>

在本示例中，您将在[Jira](https://www.atlassian.com/software/jira) 和 Port 之间创建一个 webhook 集成，用于接收 Jira 问题实体。

<h2> Port configuration </h2>

创建以下蓝图定义: 

<details>
<summary>Jira issue blueprint</summary>

<JiraIssueBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>Jira issue webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Jira mapper`；
    2.标识符 : `jira_mapper`；
    3.Description : `将 Jira 问题映射到 Port` 的 webhook 配置；
    4.图标 : `Jira`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <JiraIssueConfiguration/>
3.单击页面底部的**保存**。

</details>

<h2> Create a webhook in Jira </h2>

1. 以具有管理 Jira 全局权限的用户身份登录 Jira；
2. 单击右上角的齿轮图标；
3. 选择 **系统**；
4. 在左侧边栏底部的**高级**下，选择**Webhooks**；
5. 点击**创建 Webhook**
6. 输入以下详细信息: 
    1. 名称"- 被用于一个有意义的名称，如 Port Webhook；
    2. 状态"--请确保网络钩子已**启用**；
    3. Webhook URL` - 输入创建 Webhook 配置后收到的 `url` 键的值；
    4. `Description` - 输入 webhook 的描述；
    5.问题相关事件"- 在此部分输入 JQL 查询，以筛选发送到 webhook 的问题(如果此字段为空，则所有问题都将触发 webhook 事件)；
    6.问题 "下 - 标记创建、更新和删除；
7.单击页面底部的**创建**。

:::tip 为了查看 Jira webhooks 中可用的不同有效载荷和事件、[look here](https://developer.atlassian.com/server/jira/platform/webhooks/)

:::

完成！您对问题所做的任何更改(打开、关闭、编辑等)都会触发 webhook 事件，Jira 会将该事件发送到 Port 提供的 webhook URL。 Port 会根据映射解析事件，并相应地更新目录实体。

<h2> Let's Test It </h2>

本节包括创建或更新问题时从 Jira 发送的 webhook 事件示例。 此外，还包括根据上一节提供的 webhook 配置从事件中创建的实体。

<h3> Payload </h3>

下面是创建 Jira 问题时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Webhook event payload</summary>

```json showLineNumbers
{
  "timestamp": 1686916266116,
  "webhookEvent": "jira:issue_created",
  "issue_event_type_name": "issue_created",
  "user": {
    "self": "https://account.atlassian.net/rest/api/2/user?accountId=557058%3A69f39959-769f-4dac-8a7a-46eb55b03723",
    "accountId": "557058%3A69f39959-769f-4dac-8a7a-46eb55b03723",
    "avatarUrls": {
      "48x48": "https://secure.gravatar.com/avatar/9df2ac1caa70b0a67ff0561f7d0363e5?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-1.png"
    },
    "displayName": "Your Name",
    "active": true,
    "timeZone": "Europe/London",
    "accountType": "atlassian"
  },
  "issue": {
    "id": "10000",
    "self": "https://account.atlassian.net/rest/api/2/10000",
    "key": "PI-1",
    "fields": {
      "statuscategorychangedate": "2023-06-16T11:51:06.277+0000",
      "issuetype": {
        "self": "https://account.atlassian.net/rest/api/2/issuetype/10002",
        "id": "10002",
        "description": "Epics track collections of related bugs, stories, and tasks.",
        "iconUrl": "https://account.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10307?size=medium",
        "name": "Epic",
        "subtask": false,
        "avatarId": 10307,
        "entityId": "66c6d416-6eb4-4b38-92fa-9a7d68c64165",
        "hierarchyLevel": 1
      },
      "timespent": "None",
      "project": {
        "self": "https://account.atlassian.net/rest/api/2/project/10000",
        "id": "10000",
        "key": "PI",
        "name": "Port Integration",
        "projectTypeKey": "software",
        "simplified": true,
        "avatarUrls": {
          "48x48": "https://account.atlassian.net/rest/api/2/universal_avatar/view/type/project/avatar/10413"
        }
      },
      "fixVersions": [],
      "aggregatetimespent": "None",
      "resolution": "None",
      "resolutiondate": "None",
      "workratio": -1,
      "watches": {
        "self": "https://account.atlassian.net/rest/api/2/issue/PI-1/watchers",
        "watchCount": 0,
        "isWatching": false
      },
      "issuerestriction": {
        "issuerestrictions": {},
        "shouldDisplay": true
      },
      "lastViewed": "None",
      "created": "2023-06-16T11:51:05.291+0000",
      "priority": {
        "self": "https://account.atlassian.net/rest/api/2/priority/3",
        "iconUrl": "https://account.atlassian.net/images/icons/priorities/medium.svg",
        "name": "Medium",
        "id": "3"
      },
      "labels": ["cloud", "infra"],
      "issuelinks": [],
      "assignee": {
        "self": "https://account.atlassian.net/rest/api/2/user?accountId=557058%3A69f39947-769f-4dac-8a7a-46eb55b03705",
        "accountId": "557058:69f39947-769f-4dac-8a7a-46eb55b03705",
        "avatarUrls": {
          "48x48": "https://secure.gravatar.com/avatar/9df2ac1caa70b0a67ff0561f7d0363e5?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-1.png"
        },
        "displayName": "Your Name",
        "active": true,
        "timeZone": "Europe/London",
        "accountType": "atlassian"
      },
      "updated": "2023-06-16T11:51:05.291+0000",
      "status": {
        "self": "https://account.atlassian.net/rest/api/2/status/10000",
        "description": "",
        "iconUrl": "https://account.atlassian.net/",
        "name": "To Do",
        "id": "10000",
        "statusCategory": {
          "self": "https://account.atlassian.net/rest/api/2/statuscategory/2",
          "id": 2,
          "key": "new",
          "colorName": "blue-gray",
          "name": "New"
        }
      },
      "components": [],
      "timeoriginalestimate": "None",
      "description": "We need to migrate our current infrastructure from in-house to the cloud",
      "attachment": [],
      "summary": "Migrate Infra to Cloud",
      "creator": {
        "self": "https://account.atlassian.net/rest/api/2/user?accountId=557058%3A69f39947-769f-4dac-8a7a-46eb55b03705",
        "accountId": "557058:69f39947-769f-4dac-8a7a-46eb55b03705",
        "avatarUrls": {
          "48x48": "https://secure.gravatar.com/avatar/9df2ac1caa70b0a67ff0561f7d0363e5?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-1.png"
        },
        "displayName": "Your Name",
        "active": true,
        "timeZone": "Europe/London",
        "accountType": "atlassian"
      },
      "subtasks": [],
      "reporter": {
        "self": "https://account.atlassian.net/rest/api/2/user?accountId=557058%3A69f39947-769f-4dac-8a7a-46eb55b03705",
        "accountId": "557058:69f39947-769f-4dac-8a7a-46eb55b03705",
        "avatarUrls": {
          "48x48": "https://secure.gravatar.com/avatar/9df2ac1caa70b0a67ff0561f7d0363e5?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FIC-1.png"
        },
        "displayName": "Your Name",
        "active": true,
        "timeZone": "Europe/London",
        "accountType": "atlassian"
      },
      "aggregateprogress": {
        "progress": 0,
        "total": 0
      },
      "environment": "None",
      "duedate": "2023-06-19",
      "progress": {
        "progress": 0,
        "total": 0
      },
      "votes": {
        "self": "https://account.atlassian.net/rest/api/2/issue/PI-1/votes",
        "votes": 0,
        "hasVoted": false
      }
    }
  },
  "changelog": {
    "id": "10001",
    "items": [
      {
        "field": "status",
        "fieldtype": "jira",
        "fieldId": "status",
        "from": "10000",
        "fromString": "To Do",
        "to": "10001",
        "toString": "In Progress"
      }
    ]
  }
}
```

</details>

<h3> Mapping Result </h3>

结合示例有效载荷和 webhook 配置可生成以下 Port 实体: 

```json showLineNumbers
{
  "identifier": "PI-1",
  "title": "PI-1 - Migrate Infra to Cloud",
  "blueprint": "jiraIssue",
  "properties": {
    "summary": "Migrate Infra to Cloud",
    "description": "We need to migrate our current infrastructure from in-house to the cloud",
    "status": "To Do",
    "lastChangeType": "issue_created",
    "changingUser": "Your Name",
    "issueUrl": "https://account.atlassian.net/browse/PI-1",
    "issueType": "Epic"
  },
  "relations": {}
}
```

<h2> Import Jira Historical Issues </h2>

在本示例中，您将使用 Provider 提供的 Python 脚本从 Jira API 获取数据并将其引用到 Port。

<h3> Prerequisites </h3>

本示例使用的是上一节中的[blueprint and webhook](#prerequisites) 定义。

此外，它还需要一个 Jira API 标记，作为参数提供给 Python 脚本

<h4> Create the Jira API token </h4>

1. 登录[Jira account](https://id.atlassian.com/manage-profile/security/api-tokens) ；
2. 单击创建 API 标记；
3. 在出现的对话框中，为您的令牌输入一个简洁易记的标签，然后单击 **创建**；
4. 单击**复制**将令牌复制到剪贴板，离开此页面后将不会再有机会查看令牌值；

使用以下 Python 脚本将历史 Jira 问题引用到 Port: 

<details>
<summary>Jira Python script for historical issues</summary>

<JiraIssueConfigurationPython/>

:::note 

脚本需要以下环境变量

* `PORT_URL` - 创建 webhook 配置后由 Port 生成的 webhook URL；
* `JIRA_SERVER` - 您的 Jira 域名，例如 `https://{YOUR_DOMAIN}.atlassian.net`；
* `USERNAMES` - 您的 Jira 用户名；
* `API_TOKEN` - 您的 Jira API 令牌(在上一步中创建)。

:::

</details>

完成！现在您可以将历史问题从 Jira 导入 Port。 Port 将根据映射解析问题，并相应地更新目录实体。

</details>
