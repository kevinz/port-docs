import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import DockerParameters from "./_sentry-docker-parameters.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'
import SentryCommentsBlueprint from "../webhook/examples/resources/sentry/_example_sentry_comments_blueprint.mdx";
import SentryCommentsConfiguration from "../webhook/examples/resources/sentry/_example_sentry_comment_webhook_configuration.mdx"
import SentryIssuesBluePrint from "../webhook/examples/resources/sentry/_example_sentry_issue_event_blueprint.mdx"
import SentryIssuesConfiguration from "../webhook/examples/resources/sentry/_example_sentry_issue_event_webhook_configuration.mdx"

# 哨兵

通过我们的 Sentry 集成，您可以根据您的映射和定义，将 Sentry 云账户中的 "项目 "和 "问题 "导入 Port。

项目 "本质上是一个容器，可容纳与要监控的特定应用程序或服务相关的所有数据和信息。

问题 "是一组描述您症状相应问题的事件。

## 常见被用于情况

* 将您监控的项目和问题映射到 Port 中。

## 先决条件

<Prerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                               | Description                                                                                                   | Required |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| `port.clientId`                         | Your port client id                                                                                           | ✅       |
| `port.clientSecret`                     | Your port client secret                                                                                       | ✅       |
| `port.baseUrl`                          | Your port base url, relevant only if not using the default port app                                           | ❌       |
| `integration.identifier`                | Change the identifier to describe your integration                                                            | ✅       |
| `integration.type`                      | The integration type                                                                                          | ✅       |
| `integration.eventListener.type`        | The event listener type                                                                                       | ✅       |
| `integration.secrets.sentryToken`       | The Sentry API token                                                                                     | ✅       |
| `integration.config.sentryHost`         | The Sentry host. For example https://sentry.io                                                                | ✅       |
| `integration.config.sentryOrganization` | The Sentry organization slug                                                                                  | ✅       |
| `scheduledResyncInterval`               | The number of minutes between each resync                                                                     | ❌       |
| `initializePortResources`               | Default true, When set to true the integration will create default blueprints and the port App config Mapping | ❌       |


<br/>

<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-sentry-integration port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set port.baseUrl="https://api.getport.io"  \
    --set initializePortResources=true  \
    --set integration.identifier="my-sentry-integration"  \
    --set integration.type="sentry"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.config.sentryHost="https://sentry.io"  \
    --set integration.secrets.sentryToken="string"  \
    --set integration.config.sentryOrganization="string"
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在你的 git 仓库的 `argocd/my-ocean-sentry-integration` 中创建一个 `values.yaml` 文件，内容如下: 

:::note 记住替换 `SENTRY_HOST``SENTRY_ORGANIZATION` 和 `SENTRY_TOKEN` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-sentry-integration
  type: sentry
  eventListener:
    type: POLLING
  config:
  // highlight-start
    sentryHost: SENTRY_HOST
    sentryOrganization: SENTRY_ORGANIZATION
  // highlight-end
  secrets:
  // highlight-next-line
    sentryToken: SENTRY_TOKEN
```

<br/>

2.创建以下 "my-ocean-sentry-integration.yaml "配置清单，安装 "my-ocean-sentry-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-sentry-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-sentry-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-sentry-integration/values.yaml
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
kubectl apply -f my-ocean-sentry-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">
 <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Sentry integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成能实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />

<br/>

下面是 `sentry-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Sentry Exporter Workflow

# This workflow responsible for running Sentry exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Sentry Integration
        run: |
          # Set Docker image and run the container
          integration_type="sentry"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__SENTRY_TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__SENTRY_TOKEN }} \
          -e OCEAN__INTEGRATION__CONFIG__SENTRY_HOST=${{ secrets.OCEAN__INTEGRATION__CONFIG__SENTRY_HOST }} \
          -e OCEAN__INTEGRATION__CONFIG__SENTRY_ORGANIZATION=${{ secrets.OCEAN__INTEGRATION__CONFIG__SENTRY_ORGANIZATION }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Sentry integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/) 的 "Secret Text "类型: 

<DockerParameters />

<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

```yml showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run Sentry Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__SENTRY_TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__SENTRY_TOKEN'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__SENTRY_HOST', variable: 'OCEAN__INTEGRATION__CONFIG__SENTRY_HOST'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__SENTRY_ORGANIZATION', variable: 'OCEAN__INTEGRATION__CONFIG__SENTRY_ORGANIZATION'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="sentry"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__SENTRY_TOKEN=$OCEAN__INTEGRATION__CONFIG__SENTRY_TOKEN \
                                -e OCEAN__INTEGRATION__CONFIG__SENTRY_HOST=$OCEAN__INTEGRATION__CONFIG__SENTRY_HOST \
                                -e OCEAN__INTEGRATION__CONFIG__SENTRY_ORGANIZATION=$OCEAN__INTEGRATION__CONFIG__SENTRY_ORGANIZATION \
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

  <AzurePremise name="Sentry" />

<DockerParameters />

<br/>

下面是 `sentry-integration.yml` Pipelines 文件的示例: 

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
    integration_type="sentry"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
       -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
      -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
      -e OCEAN__INTEGRATION__CONFIG__SENTRY_TOKEN=${OCEAN__INTEGRATION__CONFIG__SENTRY_TOKEN} \
      -e OCEAN__INTEGRATION__CONFIG__SENTRY_HOST=${OCEAN__INTEGRATION__CONFIG__SENTRY_HOST} \
      -e OCEAN__INTEGRATION__CONFIG__SENTRY_ORGANIZATION=${OCEAN__INTEGRATION__CONFIG__SENTRY_ORGANIZATION} \
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

## 接收哨兵对象

Sentry 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 Sentry 获取 "Issue "数据的过程: 

```yaml showLineNumbers
resources:
  - kind: issue
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: ".id"
          title: ".title"
          blueprint: '"sentryIssue"'
          properties:
            link: ".permalink"
            status: ".status"
            isUnhandled: ".isUnhandled"
          relations:
            project: ".project.slug"
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 Sentry API 事件的现有字段和值进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了将从 Sentry 查询哪些资源，以及将在 Port 中创建哪些实体和属性。

:::tip  支持的资源 下列资源可被用于来映射来自 Sentry 的数据，可以引用下面链接的 API 响应中出现的任何字段来进行映射配置。

* * [`Project`](https://docs.sentry.io/api/projects/list-your-projects/)
* [`Issue`](https://docs.sentry.io/api/events/list-a-projects-issues/)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: project
      selector:
      ...
  ```


* 类型 "键是哨兵对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: project
        selector:
        ...
  ```


* Port"、"实体 "和 "映射 "键被用来将 Sentry 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: project
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings:
            identifier: .slug
            title: .name
            blueprint: '"sentryProject"'
            properties:
              dateCreated: .dateCreated
              platform: .platform
              status: .status
              link: .organization.links.organizationUrl + "/projects/" + .name
          # highlight-end
  ```


:::tip  Blueprint key 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，则需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再使用一对双引号 (`"`)。

:::

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Sentry 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要使用 Sentry 进行引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在 APM 和警报类别下选择 Sentry。
5. 将[integration configuration](#configuration-structure) 的内容添加到编辑器中。
6. 单击 "Resync"。

## 示例

蓝图和相关集成配置示例: 

### 项目

<details>
<summary>Project blueprint</summary>

```json showLineNumbers
{
  "identifier": "sentryProject",
  "title": "project",
  "icon": "Sentry",
  "schema": {
    "properties": {
      "dateCreated": {
        "title": "dateCreated",
        "type": "string",
        "format": "date-time"
      },
      "platform": {
        "type": "string",
        "title": "platform"
      },
      "status": {
        "title": "status",
        "type": "string",
        "enum": [
          "active",
          "disabled",
          "pending_deletion",
          "deletion_in_progress"
        ]
      },
      "link": {
        "title": "link",
        "type": "string",
        "format": "url"
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
  - kind: project
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .slug
          title: .name
          blueprint: '"sentryProject"'
          properties:
            dateCreated: .dateCreated
            platform: .platform
            status: .status
            link: .organization.links.organizationUrl + "/projects/" + .name
```

</details>

### 问题

<details>
<summary>Issue blueprint</summary>

```json showLineNumbers
{
  "identifier": "sentryIssue",
  "title": "issue",
  "icon": "Sentry",
  "schema": {
    "properties": {
      "link": {
        "title": "link",
        "type": "string",
        "format": "url"
      },
      "status": {
        "title": "status",
        "type": "string"
      },
      "isUnhandled": {
        "icon": "DefaultProperty",
        "title": "isUnhandled",
        "type": "boolean"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "project": {
      "title": "project",
      "target": "sentryProject",
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
- kind: issue
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: ".id"
          title: ".title"
          blueprint: '"sentryIssue"'
          properties:
            link: ".permalink"
            status: ".status"
            isUnhandled: ".isUnhandled"
          relations:
            project: ".project.slug"
```

</details>

## 让我们来测试一下

本节包括来自 Sentry 的响应数据示例。 此外，还包括根据上一节提供的 Ocean 配置从重新同步事件中创建的实体。

### 有效载荷

以下是 Sentry 有效载荷结构的示例: 

<details>
<summary> Project response data</summary>

```json showLineNumbers
{
  "id": "4504931759095808",
  "slug": "python-fastapi",
  "name": "python-fastapi",
  "platform": "python-fastapi",
  "dateCreated": "2023-03-31T06:18:37.290732Z",
  "isBookmarked": false,
  "isMember": false,
  "features": [
    "alert-filters",
    "minidump",
    "race-free-group-creation",
    "similarity-indexing",
    "similarity-view",
    "span-metrics-extraction",
    "span-metrics-extraction-resource",
    "releases"
  ],
  "firstEvent": "2023-03-31T06:25:54.666640Z",
  "firstTransactionEvent": false,
  "access": [],
  "hasAccess": true,
  "hasMinifiedStackTrace": false,
  "hasMonitors": false,
  "hasProfiles": false,
  "hasReplays": false,
  "hasFeedbacks": false,
  "hasSessions": false,
  "isInternal": false,
  "isPublic": false,
  "avatar": {
    "avatarType": "letter_avatar",
    "avatarUuid": null
  },
  "color": "#913fbf",
  "status": "active",
  "organization": {
    "id": "4504931754901504",
    "slug": "test-org",
    "status": {
      "id": "active",
      "name": "active"
    },
    "name": "Test Org",
    "dateCreated": "2023-03-31T06:17:33.619189Z",
    "isEarlyAdopter": false,
    "require2FA": false,
    "requireEmailVerification": false,
    "avatar": {
      "avatarType": "letter_avatar",
      "avatarUuid": null,
      "avatarUrl": null
    },
    "features": [
      "performance-tracing-without-performance",
      "performance-consecutive-http-detector",
      "performance-large-http-payload-detector",
      "escalating-issues",
      "minute-resolution-sessions",
      "performance-issues-render-blocking-assets-detector",
      "event-attachments"
    ],
    "links": {
      "organizationUrl": "https://test-org.sentry.io",
      "regionUrl": "https://us.sentry.io"
    },
    "hasAuthProvider": false
  }
}
```

</details>

<details>
<summary> Issue response data</summary>

```json showLineNumbers
{
  "id": "4605173695",
  "shareId": "None",
  "shortId": "PYTHON-FASTAPI-2",
  "title": "ZeroDivisionError: division by zero",
  "culprit": "index",
  "permalink": "https://test-org.sentry.io/issues/4605173695/",
  "logger": "None",
  "level": "error",
  "status": "unresolved",
  "statusDetails": {},
  "substatus": "new",
  "isPublic": false,
  "platform": "python",
  "project": {
    "id": "4504931759095808",
    "name": "python-fastapi",
    "slug": "python-fastapi",
    "platform": "python-fastapi"
  },
  "type": "error",
  "metadata": {
    "value": "division by zero",
    "type": "ZeroDivisionError",
    "filename": "app.py",
    "function": "index",
    "display_title_with_tree_label": false,
    "in_app_frame_mix": "mixed"
  },
  "numComments": 0,
  "assignedTo": "None",
  "isBookmarked": false,
  "isSubscribed": false,
  "subscriptionDetails": "None",
  "hasSeen": false,
  "annotations": [],
  "issueType": "error",
  "issueCategory": "error",
  "isUnhandled": true,
  "count": "1",
  "userCount": 0,
  "firstSeen": "2023-11-06T08:31:27.058163Z",
  "lastSeen": "2023-11-06T08:31:27.058163Z",
  "stats": {
    "24h": [
      [1699174800, 0],
      [1699178400, 0],
      [1699182000, 0],
      [1699250400, 0],
      [1699254000, 0],
      [1699257600, 1]
    ]
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
  "identifier": "python-fastapi",
  "title": "python-fastapi",
  "icon": null,
  "blueprint": "sentryProject",
  "team": [],
  "properties": {
    "dateCreated": "2023-03-31T06:18:37.290732Z",
    "platform": "python-fastapi",
    "status": "active",
    "link": "https://test-org.sentry.io/projects/python-fastapi"
  },
  "relations": {},
  "createdAt": "2023-11-06T08:49:17.700Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-11-06T08:59:11.446Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>

<details>
<summary> Issue entity in Port</summary>

```json showLineNumbers
{
  "identifier": "4605173695",
  "title": "ZeroDivisionError: division by zero",
  "icon": null,
  "blueprint": "sentryIssue",
  "team": [],
  "properties": {
    "link": "https://test-org.sentry.io/issues/4605173695/",
    "status": "unresolved",
    "isUnhandled": true
  },
  "relations": {
    "project": "python-fastapi"
  },
  "createdAt": "2023-11-06T08:49:20.406Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-11-06T08:49:20.406Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>

## 通过 webhook 进行替代安装

虽然上述 Ocean 集成是推荐的安装方法，但您可能更喜欢使用 webhook 从 Sentry 引用数据。 如果是这样，请使用以下说明: 

<details>

<summary><b>Webhook installation (click to expand)</b></summary>

在本例中，您将在[Sentry](https://sentry.io) 和 Port 之间创建一个 webhook 集成，该集成将摄取问题实体。

<h2>Port configuration</h2>

创建以下蓝图定义: 

<details>

<summary>Sentry issue blueprint</summary>
<SentryIssuesBluePrint/>

</details>

创建以下 webhook 配置[using Port UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints) : 

<details>

<summary>Sentry issue webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: "Sentry 问题映射器"；
    2.标识符: `sentry_issue_mapper`；
    3.Description : `将 Sentry 问题映射到 Port` 的 webhook 配置；
    4.图标 : `Sentry`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <SentryIssuesConfiguration/>
3.向下滚动到**高级设置**，输入以下详细信息: 
    1.签名头名称: `sentry-hooks-signature`；
    2.签名算法: 从下拉选项中选择 `sha256`；
    3.点击页面底部的**保存**。

</details>

:::tip 我们省略了 webhook 配置中安全对象的 "secret "字段，因为secret值是由 Sentry 在创建 webhook 时生成的。 因此，在遵循此示例时，请先在 Port 中创建 webhook 配置。 使用响应中的 webhook URL 并在 Sentry 中创建 webhook。 从 Sentry 获取secret后，您可以返回 Port 并使用secret更新[webhook configuration](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints) 。

:::

<h2>Create a webhook in Sentry</h2>

1. 使用贵组织的凭据登录 Sentry；
2. 单击页面左侧边栏的齿轮图标(设置)；
3. 选择 **开发设置**；
4. 在页面上角，点击**创建新集成**；
5. Sentry 提供两种类型的集成: 内部集成和公共集成。在本指南中，请选择**内部集成**，然后点击**下一步**按钮；
6. 输入以下详细信息: 
    1. `Name` - 被用于一个有意义的名称，如 Port Webhook；
    2. Webhook URL` - 输入创建 Webhook 配置后收到的 `url` 键的值；
    3. `Overview` - 输入 webhook 的描述；
    4. 权限"- 授予 webhook 在**问题和事件**类别的**读取**权限；
    5.Webhooks` - 在该部分下，启用问题复选框，允许 Sentry 向 Port 报告问题事件；
7.单击页面底部的**保存更改**。

:::tip 网络钩子已创建，您可以引用 Sentry 生成的secret值，并用它更新 Port 网络钩子配置中的 `security` 对象

:::

<h2>Relate comments to Issues</h2>

下面的示例除了上一个示例中的 "sentryIssue "蓝图外，还添加了一个 "sentryComment "蓝图。 此外，它还添加了一个 "sentryIssue "关系。 webhook 将创建或更新这两个现有实体之间的关系，使您可以映射评论是针对哪个问题提出的: 

<details>

<summary>Sentry comments blueprint (including the sentryIssue relation)</summary>
<SentryCommentsBlueprint/>

</details>

创建以下 webhook 配置[using Port UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints) : 

<details>

<summary>Sentry comments webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: "条目注释映射器"；
    2.标识符: `sentry_comment_mapper`；
    3.Description : `将 Sentry 注释映射到 Port` 的 webhook 配置；
    4.图标 : `Sentry`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <SentryCommentsConfiguration/>
3.向下滚动到**高级设置**，输入以下详细信息: 
    1.签名头名称: `sentry-hooks-signature`；
    2.签名算法: 从下拉选项中选择 `sha256`；
    3.点击页面底部的**保存**。

</details>

:::tip 要查看 Sentry webhooks 中的不同有效载荷和事件、[click here](https://docs.sentry.io/product/integrations/integration-platform/webhooks/)

:::

完成！Sentry 中的任何问题和评论都将触发 webhook 事件。 Port 将根据映射解析事件，并相应地更新目录实体。

<h2>Let's Test It</h2>

本节包括创建问题或评论时 Sentry 发送的 webhook 事件示例。 此外，本节还包括根据上一节提供的 webhook 配置从事件中创建的实体。

<h3>Payload</h3>

下面是创建 Sentry 问题或评论时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Sentry issue webhook event payload</summary>

```json showLineNumbers
{
  "action": "created",
  "installation": {
    "uuid": "54a3e698-f389-4d86-b9f8-50093a228449"
  },
  "data": {
    "issue": {
      "id": "4253613038",
      "shareId": "None",
      "shortId": "PYTHON-B",
      "title": "NameError: name 'total' is not defined",
      "culprit": "__main__ in <module>",
      "permalink": "None",
      "logger": "None",
      "level": "error",
      "status": "unresolved",
      "statusDetails": {},
      "substatus": "new",
      "isPublic": false,
      "platform": "python",
      "project": {
        "id": "4504989602480128",
        "name": "python",
        "slug": "python",
        "platform": "python"
      },
      "type": "error",
      "metadata": {
        "value": "name 'total' is not defined",
        "type": "NameError",
        "filename": "sentry.py",
        "function": "<module>",
        "display_title_with_tree_label": false
      },
      "numComments": 0,
      "assignedTo": "None",
      "isBookmarked": false,
      "isSubscribed": false,
      "subscriptionDetails": "None",
      "hasSeen": false,
      "annotations": [],
      "issueType": "error",
      "issueCategory": "error",
      "isUnhandled": true,
      "count": "1",
      "userCount": 0,
      "firstSeen": "2023-06-15T17:10:09.914274Z",
      "lastSeen": "2023-06-15T17:10:09.914274Z"
    }
  },
  "actor": {
    "type": "application",
    "id": "sentry",
    "name": "Sentry"
  }
}
```

</details>

<details>
<summary> Sentry comment webhook event payload</summary>

```json showLineNumbers
{
  "action": "created",
  "installation": {
    "uuid": "d5a2de51-0138-496a-8e79-c17747c3a40d"
  },
  "data": {
    "comment_id": "1729635072",
    "issue_id": "4253613038",
    "project_slug": "python",
    "timestamp": "2023-06-15T17:15:53.383120Z",
    "comment": "Hello admin please take a look at this"
  },
  "actor": {
    "type": "user",
    "id": 2683666,
    "name": "user@domain.com"
  }
}
```

</details>

<h3>Mapping Result</h3>

结合示例有效载荷和 webhook 配置可生成以下 Port `sentryIssue` 实体: 

```json showLineNumbers
{
  "identifier": "4253613038",
  "title": "NameError: name 'total' is not defined",
  "blueprint": "sentryIssue",
  "properties": {
    "action": "created",
    "level": "error",
    "platform": "python",
    "status": "unresolved",
    "projectID": "4504989602480128"
  },
  "relations": {}
}
```

此外，还将生成以下 Port `sentryComment` 实体: 

```json showLineNumbers
{
  "identifier": "1729635072",
  "title": "Comment Event",
  "blueprint": "sentryComment",
  "properties": {
    "action": "created",
    "comment": "Hello admin please take a look at this",
    "project": "python",
    "issue_id": "4253613038",
    "timestamp": "2023-06-15T17:15:53.383120Z"
  },
  "relations": {
    "sentryIssue": "4253613038"
  }
}
```

</details>