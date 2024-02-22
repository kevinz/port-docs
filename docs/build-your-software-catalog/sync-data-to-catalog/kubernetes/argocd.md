import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import ProjecttBlueprint from '../webhook/examples/resources/argocd/_example_project_blueprint.mdx'
import ApplicationBlueprint from '../webhook/examples/resources/argocd/_example_application_blueprint.mdx'
import EventBlueprint from '../webhook/examples/resources/argocd/_example_event_blueprint.mdx'

import ArgoCDWebhookConfig from '../webhook/examples/resources/argocd/_example_webhook_configuration.mdx'
import ArgoCDEventWebhookConfig from '../webhook/examples/resources/argocd/_example_events_webhook_config.mdx'
import ArgoCDEventManifest from '../webhook/examples/resources/argocd/_example_events_manifest.mdx'

# ArgoCD

通过 ArgoCD 集成，您可以根据映射和定义将 ArgoCD 实例中的 "集群"、"项目"、"应用程序 "和 "部署历史 "资源导入 Port。

## 常见被引用情况

* 在 ArgoCD 中映射受监控的 Kubernetes 资源。
* 实时关注对象变更(创建/更新)，并自动将变更应用到 Port 中的实体。

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
| `integration.secrets.token`      | The ArgoCD API token token                                                                                    | ✅       |
| `integration.config.serverUrl`   | The ArgoCD server url                                                                                         | ✅       |
| `scheduledResyncInterval`        | The number of minutes between each resync                                                                     | ❌       |
| `initializePortResources`        | Default true, When set to true the integration will create default blueprints and the port App config Mapping | ❌       |


<br/>

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-argocd-integration port-labs/port-ocean \
  --set port.clientId="CLIENT_ID"  \
  --set port.clientSecret="CLIENT_SECRET"  \
  --set initializePortResources=true  \
  --set scheduledResyncInterval=60  \
  --set integration.identifier="my-argocd-integration"  \
  --set integration.type="argocd"  \
  --set integration.eventListener.type="POLLING"  \
  --set integration.secrets.token="<your-token>"  \
  --set integration.config.serverUrl="<your-server-url>"
```

</TabItem>

<TabItem value="one-time" label="Scheduled">
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the ArgoCD integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 


| Parameter                                | Description                                                                                                        | Required |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------- |
| `OCEAN__INTEGRATION__CONFIG__TOKEN`      | The ArgoCD API token                                                                                               | ✅       |
| `OCEAN__INTEGRATION__CONFIG__SERVER_URL` | The ArgoCD server URL                                                                                              | ✅       |
| `OCEAN__INITIALIZE_PORT_RESOURCES`       | Default true, When set to false the integration will not create default blueprints and the port App config Mapping | ❌       |
| `OCEAN__INTEGRATION__IDENTIFIER`         | Change the identifier to describe your integration, if not set will use the default one                            | ❌       |
| `OCEAN__PORT__CLIENT_ID`                 | Your port client id                                                                                                | ✅       |
| `OCEAN__PORT__CLIENT_SECRET`             | Your port client secret                                                                                            | ✅       |
| `OCEAN__PORT__BASE_URL`                  | Your port base url, relevant only if not using the default port app                                                | ❌       |


<br/>

下面是 `argocd-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: ArgoCD Exporter Workflow

# This workflow responsible for running ArgoCD exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run ArgoCD Integration
        run: |
          # Set Docker image and run the container
          integration_type="argocd"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__TOKEN }} \
          -e OCEAN__INTEGRATION__CONFIG__SERVER_URL=${{ secrets.OCEAN__INTEGRATION__CONFIG__SERVER_URL }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the ArgoCD integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/)的 "Secret Text "类型: 


| Parameter                                | Description                                                                                                        | Required |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------- |
| `OCEAN__INTEGRATION__CONFIG__TOKEN`      | The ArgoCD API token                                                                                               | ✅       |
| `OCEAN__INTEGRATION__CONFIG__SERVER_URL` | The ArgoCD server URL                                                                                              | ✅       |
| `OCEAN__INITIALIZE_PORT_RESOURCES`       | Default true, When set to false the integration will not create default blueprints and the port App config Mapping | ❌       |
| `OCEAN__INTEGRATION__IDENTIFIER`         | Change the identifier to describe your integration, if not set will use the default one                            | ❌       |
| `OCEAN__PORT__CLIENT_ID`                 | Your port client id                                                                                                | ✅       |
| `OCEAN__PORT__CLIENT_SECRET`             | Your port client secret                                                                                            | ✅       |
| `OCEAN__PORT__BASE_URL`                  | Your port base url, relevant only if not using the default port app                                                | ❌       |


<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

```text showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run ArgoCD Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__TOKEN'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__SERVER_URL', variable: 'OCEAN__INTEGRATION__CONFIG__SERVER_URL'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="argocd"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__TOKEN=$OCEAN__INTEGRATION__CONFIG__TOKEN \
                                -e OCEAN__INTEGRATION__CONFIG__SERVER_URL=$OCEAN__INTEGRATION__CONFIG__SERVER_URL \
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

### 生成 ArgoCD 令牌

1. 导航至 `<serverURL>/settings/accounts/<user>`。例如，如果您在 `https://localhost:8080` 访问 ArgoCD，则应导航到 `https://localhost:8080/settings/accounts/<user>`。
2. 在 "令牌 "下，单击 "**生成新**"创建新令牌。

## 接收 ArgoCD 对象

ArgoCD 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 ArgoCD 获取 "应用程序 "数据的过程: 

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: application
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .metadata.uid
          title: .metadata.name
          blueprint: '"argocdApplication"'
          properties:
            gitRepo: .spec.source.repoURL
            gitPath: .spec.source.path
            destinationServer: .spec.destination.server
            namespace: .metadata.namespace
            syncStatus: .status.sync.status
            healthStatus: .status.health.status
            createdAt: .metadata.creationTimestamp
          relations:
            project: .spec.project
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 从 ArgoCD 的 API 事件中对现有字段和 Values 进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了从 ArgoCD 中查询哪些资源，以及在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被引用来映射 ArgoCD 中的数据，可以引用下面链接的 API 响应中出现的任何字段来进行映射配置。

* * [`cluster`](https://cd.apps.argoproj.io/swagger-ui#operation/ClusterService_List)
* [`project`](https://cd.apps.argoproj.io/swagger-ui#operation/ProjectService_List)
* [`application`](https://cd.apps.argoproj.io/swagger-ui#operation/ApplicationService_List)
* [`deployment-history`](https://cd.apps.argoproj.io/swagger-ui#operation/ApplicationService_List)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: application
      selector:
      ...
  ```


* kind "键是 ArgoCD 对象的说明符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: application
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: application
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
      # highlight-end
      port:
  ```


* Port"、"实体 "和 "映射 "键被用来将 ArgoCD 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: application
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one ArgoCD object to a Port entity. Each value is a JQ query.
            identifier: .metadata.uid
            title: .metadata.name
            blueprint: '"argocdApplication"'
            properties:
              gitRepo: .spec.source.repoURL
              gitPath: .spec.source.path
              destinationServer: .spec.destination.server
              namespace: .metadata.namespace
              syncStatus: .status.sync.status
              healthStatus: .status.health.status
              createdAt: .metadata.creationTimestamp
            relations:
              project: .spec.project
        # highlight-end
    - kind: application # In this instance application is mapped again with a different filter
      selector:
        query: '.metadata.name == "MyApplicationName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): ::: 

## 配置实时更新

目前，ArgoCD REST API 不支持以编程方式创建 webhook。 要在 ArgoCD 中设置 webhook 配置，以便向 Ocean 集成发送通知，请按以下步骤操作: 

### 前提条件

1. 您可以访问部署 ArgoCD 的 Kubernetes 集群。
2. 安装并配置了 `kubectl` 以访问集群。

### 步骤

1. 配置 ArgoCD 通知清单；

```bash showLineNumbers
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/manifests/install.yaml
```

2.安装 ArgoCD 触发器和模板配置清单；

```bash showLineNumbers
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/catalog/install.yaml
```

3.使用 `kubectl` 连接到部署 ArgoCD 实例的 Kubernetes 集群；

```bash showLineNumbers
kubectl config use-context <your-cluster-context>
```

4.将当前名称空间设置为 ArgoCD 名称空间，请使用以下命令；

```bash showLineNumbers
kubectl config set-context --current --namespace=<your-namespace>
```

5.创建一个 YAML 文件(例如 `argocd-webhook-config.yml`)，配置 webhook 通知服务。下面的示例展示了如何设置 webhook，以便在 ArgoCD 应用程序更新时发送实时事件。YAML 文件包括以下组件: 
    1.通知服务定义；
    2.webhook 消息正文模板；
    3.触发器定义；
    4.订阅通知。
    下面是一个 YAML 示例。请确保将 `<WEBHOOK_URL>` 替换为部署海洋集成的 ingress 或服务的实际 URL。默认情况下，传入的 Webhook 事件会发送到 Ocean 中的 `/integration/webhook` 路径，因此请勿替换路径参数。
<details>
<summary>webhook manifest file </summary>


   ```yaml showLineNumbers
   apiVersion: v1
   kind: ConfigMap
   metadata:
   name: argocd-notifications-cm
   data:
   trigger.on-sync-operation-change: |
     - description: Application syncing has updated
     send:
     - app-status-change
     when: app.status.operationState.phase in ['Error', 'Failed', 'Succeeded', 'Running']
   trigger.on-deployed: |
     - description: Application is synced and healthy
     send:
     - app-status-change
     when: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'
   trigger.on-health-degraded: |
     - description: Application has degraded
     send:
     - app-status-change
     when: app.status.health.status == 'Degraded'
   service.webhook.port-ocean: |
     url: <WEBHOOK_URL>
     headers:
     - name: Content-Type
     value: application/json
   template.app-status-change: |
     webhook:
     port-ocean:
         method: POST
         path: /integration/webhook
         body: |
         {
             "action": "upsert",
             "application_name": "{{.app.metadata.name}}"
         }
   subscriptions: |
     - recipients:
     - port-ocean
     triggers:
     - on-deployed
     - on-health-degraded
     - on-sync-operation-change
   ```


   </details>

6.使用 `kubectl` 将 YAML 文件引用到集群。运行以下命令，将 `<your-namespace>` 替换为 ArgoCD 名称空间，将 `<path-to-yaml-file>` 替换为 YAML 文件的实际路径: 

```bash
kubectl apply -n <your-namespace> -f <path-to-yaml-file>
```

该命令将 webhook 通知配置部署到 ArgoCD 通知 configmaps("argocd-notifications-cm")中，使 Ocean 能够接收实时事件。

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 ArgoCD 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 ArgoCD 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在 Kubernetes Stack Provider 类别下选择 ArgoCD。
5. 根据需要修改[configuration](#configuration-structure) 。
6. 单击 "Resync"。

## 示例

蓝图和相关集成配置示例: 

#### 集群

<details>
<summary>Cluster blueprint</summary>

```json showLineNumbers
{
  "identifier": "argocdCluster",
  "description": "This blueprint represents an ArgoCD cluster",
  "title": "ArgoCD Cluster",
  "icon": "Argo",
  "schema": {
    "properties": {
      "namespaces": {
        "items": {
          "type": "string"
        },
        "icon": "DefaultProperty",
        "title": "Namespace",
        "type": "array",
        "description": "Holds list of namespaces which are accessible in that cluster."
      },
      "applicationsCount": {
        "title": "Applications Count",
        "type": "number",
        "description": "The number of applications managed by Argo CD on the cluster",
        "icon": "DefaultProperty"
      },
      "serverVersion": {
        "title": "Server Version",
        "type": "string",
        "description": "Contains information about the Kubernetes version of the cluster",
        "icon": "DefaultProperty"
      },
      "labels": {
        "title": "Labels",
        "type": "object",
        "description": "Contains information about cluster metadata",
        "icon": "DefaultProperty"
      },
      "updatedAt": {
        "icon": "DefaultProperty",
        "title": "Updated At",
        "type": "string",
        "format": "date-time"
      },
      "server": {
        "title": "Server",
        "description": "The API server URL of the Kubernetes cluster",
        "type": "string",
        "format": "url",
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
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: cluster
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .name
          title: .name
          blueprint: '"argocdCluster"'
          properties:
            namespaces: .namespaces
            applicationsCount: .info.applicationsCount
            serverVersion: .serverVersion
            labels: .labels
            updatedAt: .connectionState.attemptedAt
            server: .server
```

</details>

### 项目

<details>
<summary> Project blueprint</summary>

```json showlineNumbers
{
  "identifier": "argocdProject",
  "description": "This blueprint represents an ArgoCD Project",
  "title": "ArgoCD Project",
  "icon": "Argo",
  "schema": {
    "properties": {
      "namespace": {
        "title": "Namespace",
        "type": "string",
        "icon": "DefaultProperty"
      },
      "createdAt": {
        "title": "Created At",
        "type": "string",
        "format": "date-time",
        "icon": "DefaultProperty"
      },
      "description": {
        "title": "Description",
        "description": "Project description",
        "type": "string",
        "icon": "DefaultProperty"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "cluster": {
      "title": "Cluster",
      "target": "argocdCluster",
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
  - kind: project
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .metadata.name
          title: .metadata.name
          blueprint: '"argocdProject"'
          properties:
            namespace: .metadata.namespace
            createdAt: .metadata.creationTimestamp
            description: .spec.description
          relations:
            cluster: '[.spec.destinations[].name | select(test("^[a-zA-Z0-9@_.:/=-]+$"))]'
```

</details>

### 申请

<details>
<summary> Application blueprint</summary>

```json showlineNumbers
{
  "identifier": "argocdApplication",
  "description": "This blueprint represents an ArgoCD Application",
  "title": "ArgoCD Application",
  "icon": "Argo",
  "schema": {
    "properties": {
      "gitRepo": {
        "type": "string",
        "format": "url",
        "icon": "Git",
        "title": "Repository URL",
        "description": "The URL of the Git repository containing the application source code"
      },
      "gitPath": {
        "type": "string",
        "title": "Path",
        "description": "The path within the Git repository where the application manifests are located"
      },
      "destinationServer": {
        "type": "string",
        "title": "Destination Server",
        "format": "url"
      },
      "namespace": {
        "type": "string",
        "title": "Namespace"
      },
      "syncStatus": {
        "type": "string",
        "title": "Sync Status",
        "enum": ["Synced", "OutOfSync", "Unknown"],
        "enumColors": {
          "Synced": "green",
          "OutOfSync": "red",
          "Unknown": "lightGray"
        },
        "description": "The sync status of the application"
      },
      "healthStatus": {
        "type": "string",
        "title": "Health Status",
        "enum": [
          "Healthy",
          "Missing",
          "Suspended",
          "Degraded",
          "Progressing",
          "Unknown"
        ],
        "enumColors": {
          "Healthy": "green",
          "Missing": "yellow",
          "Suspended": "purple",
          "Degraded": "red",
          "Progressing": "blue",
          "Unknown": "lightGray"
        },
        "description": "The health status of the application"
      },
      "createdAt": {
        "title": "Created At",
        "type": "string",
        "format": "date-time"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "project": {
      "title": "Project",
      "target": "argocdProject",
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
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: application
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .metadata.uid
          title: .metadata.name
          blueprint: '"argocdApplication"'
          properties:
            gitRepo: .spec.source.repoURL
            gitPath: .spec.source.path
            destinationServer: .spec.destination.server
            namespace: .metadata.namespace
            syncStatus: .status.sync.status
            healthStatus: .status.health.status
            createdAt: .metadata.creationTimestamp
          relations:
            project: .spec.project
```

</details>

### 部署历史

<details>
<summary> Deployment history blueprint</summary>

```json showlineNumbers
{
  "identifier": "argocdDeploymentHistory",
  "description": "This blueprint represents an ArgoCD deployment history",
  "title": "ArgoCD Deployment History",
  "icon": "Argo",
  "schema": {
    "properties": {
      "deployedAt": {
        "title": "Deployed At",
        "type": "string",
        "format": "date-time",
        "icon": "DefaultProperty"
      },
      "deployStartedAt": {
        "title": "Deploy Started At",
        "type": "string",
        "format": "date-time",
        "icon": "DefaultProperty"
      },
      "revision": {
        "title": "Revision",
        "type": "string",
        "icon": "DefaultProperty"
      },
      "initiatedBy": {
        "title": "Initiated By",
        "type": "string",
        "icon": "DefaultProperty"
      },
      "repoURL": {
        "icon": "DefaultProperty",
        "title": "Repository URL",
        "type": "string",
        "format": "url"
      },
      "sourcePath": {
        "title": "Source Path",
        "type": "string",
        "icon": "DefaultProperty"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {
    "application": {
      "title": "Application",
      "target": "argocdApplication",
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
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: deployment-history
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .__applicationId + "-" + (.id | tostring)
          title: .id | tostring
          blueprint: '"argocdDeploymentHistory"'
          properties:
            deployedAt: .deployedAt
            deployStartedAt: .deployStartedAt
            revision: .revision
            initiatedBy: .initiatedBy.username
            repoURL: .source.repoURL
            sourcePath: .source.path
          relations:
            application: .__applicationId
```

</details>

## 通过 webhook 进行替代安装

虽然上述 Ocean 集成是推荐的安装方法，但您可能更喜欢使用 webhook 从 ArgoCD 引用数据。 如果是这样，请使用以下说明: 

<details>
<summary><b>Webhook installation (click to expand)</b></summary>

在本例中，您将在[ArgoCD](https://argo-cd.readthedocs.io/en/stable/) 和 Port 之间创建一个 webhook 集成，该集成将摄取应用程序实体并将其映射到 ArgoCD 项目。

<h2>Port configuration</h2>

创建以下蓝图定义: 

<details>
<summary>Project blueprint</summary>

<ProjecttBlueprint/>

</details>

<details>
<summary>Application blueprint</summary>

<ApplicationBlueprint/>

</details>

创建以下 webhook 配置[using Port UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>

<summary>Application webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `ArgoCD Application Mapper`；
    2.标识符 : `argocd_application_mapper`；
    3.Description : `将 ArgoCD 应用程序映射到 Port` 的 webhook 配置；
    4.图标 : `Argo`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <ArgoCDWebhookConfig/>
3.点击页面底部的**保存**。

</details>

<h2>Create a webhook in ArgoCD</h2>

要在 ArgoCD 中设置向 Port 发送通知的 webhook 配置，请按以下步骤操作: 

<h3>Prerequisite</h3>

1. 您可以访问部署 ArgoCD 的 Kubernetes 集群。
2. 安装并配置了 `kubectl` 以访问集群。

<h3>Steps</h3>

1. 配置 ArgoCD 通知清单；

```bash showLineNumbers
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/manifests/install.yaml
```

2.安装 ArgoCD 触发器和模板配置清单；

```bash showLineNumbers
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/catalog/install.yaml
```

3.使用 `kubectl` 连接到部署 ArgoCD 实例的 Kubernetes 集群；

```bash showLineNumbers
kubectl config use-context <your-cluster-context>
```

4.将当前名称空间设置为 ArgoCD 名称空间，请使用以下命令；

```bash showLineNumbers
kubectl config set-context --current --namespace=<your-namespace>
```

5.创建一个 YAML 文件(例如 `argocd-webhook-config.yml`)，配置 webhook 通知服务。下面的示例展示了如何设置 webhook，以便在 ArgoCD 应用程序更新时发送实时事件。YAML 文件包括以下组件: 
    1.通知服务定义；
    2.webhook 消息正文模板；
    3.触发器定义；
    4.订阅通知。
    下面是一个 YAML 示例。请确保将 `<YOUR_WEBHOOK_URL>` 替换为创建 webhook 配置后收到的 `url` 键值。
<details>
<summary>webhook manifest file </summary>


   ```yaml showLineNumbers
   apiVersion: v1
   kind: ConfigMap
   metadata:
   name: argocd-notifications-cm
   data:
   trigger.on-sync-operation-change: |
     - description: Application syncing has updated
     send:
     - app-status-change
     when: app.status.operationState.phase in ['Error', 'Failed', 'Succeeded', 'Running']
   trigger.on-deployed: |
     - description: Application is synced and healthy
     send:
     - app-status-change
     when: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'
   trigger.on-health-degraded: |
     - description: Application has degraded
     send:
     - app-status-change
     when: app.status.health.status == 'Degraded'
   service.webhook.port-webhook: |
     url: <YOUR_WEBHOOK_URL>
     headers:
     - name: Content-Type
     value: application/json
   template.app-status-change: |
     webhook:
     port-webhook:
         method: POST
         body: |
         {
             "uid": "{{.app.metadata.uid}}",
             "name": "{{.app.metadata.name}}",
             "namespace": "{{.app.metadata.namespace}}",
             "sync_status": "{{.app.status.sync.status}}",
             "health_status": "{{.app.status.health.status}}",
             "git_repo": "{{.app.spec.source.repoURL}}",
             "git_path": "{{.app.spec.source.path}}",
             "destination_server": "{{.app.spec.destination.server}}",
             "created_at": "{{.app.metadata.creationTimestamp}}",
             "project": "{{.app.spec.project}}"
         }
   subscriptions: |
     - recipients:
     - port-webhook
     triggers:
     - on-deployed
     - on-health-degraded
     - on-sync-operation-change
   ```


   </details>

6.使用 `kubectl` 将 YAML 文件引用到集群。运行以下命令，将 `<your-namespace>` 替换为 ArgoCD 名称空间，将 `<path-to-yaml-file>` 替换为 YAML 文件的实际路径: 

```bash
kubectl apply -n <your-namespace> -f <path-to-yaml-file>
```

该命令将 webhook 通知配置部署到 ArgoCD 通知 configmaps (`argocd-notifications-cm`)，使 Port 能够接收实时事件。

完成！您在 ArgoCD 中的应用程序发生的任何变化都将触发指向 Port 提供的 webhook URL 的 webhook 事件。 Port 将根据映射解析事件，并相应地更新目录实体。

<h2>Argocd Events</h2>

在本示例中，您将在[ArgoCD](https://argo-cd.readthedocs.io/en/stable/) 和 Port 之间创建一个 webhook 集成，该集成将摄取所有事件实体并将其映射到 ArgoCD 应用程序。

<h3>Port configuration</h3>

创建以下蓝图定义: 

<details>
<summary>Events blueprint</summary>

<EventBlueprint/>

</details>

创建以下 webhook 配置[using Port UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>

<summary>Application webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `ArgoCD Event Mapper`；
    2.标识符 : `argocd_event_mapper`；
    3.Description : `将 ArgoCD 事件映射到 Port` 的 webhook 配置；
    4.图标 : `Argo`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <ArgoCDEventWebhookConfig/>
3.点击页面底部的**保存**。

</details>

<h3> Create a webhook in ArgoCD </h3>

要在 ArgoCD 中设置向 Port 发送通知的 webhook 配置，请按以下步骤操作: 

<h4> Prerequisite </h4>

1. 您可以访问部署 ArgoCD 的 Kubernetes 集群。
2. 安装并配置了 `kubectl` 以访问集群。

<h4> Steps </h4>

1. 配置 ArgoCD 通知清单；

```bash showLineNumbers
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/manifests/install.yaml
```

2.安装 ArgoCD 触发器和模板配置清单；

```bash showLineNumbers
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-notifications/release-1.0/catalog/install.yaml
```

3.使用 `kubectl` 连接到部署 ArgoCD 实例的 Kubernetes 集群；

```bash showLineNumbers
kubectl config use-context <your-cluster-context>
```

4.将当前名称空间设置为 ArgoCD 名称空间，请使用以下命令；

```bash showLineNumbers
kubectl config set-context --current --namespace=<your-namespace>
```

5.创建一个 YAML 文件(例如 `argocd-events-config.yaml`)，配置 webhook 通知服务。下面的示例展示了如何设置 webhook 从 ArgoCD 发送实时事件。YAML 文件包括以下组件: 
    1.通知服务定义；
    2.webhook 消息正文模板；
    3.触发器定义；
    4.订阅通知。
    下面是一个 YAML 示例。请确保将 `<YOUR_WEBHOOK_URL>` 替换为创建 webhook 配置后收到的 `url` 键值。
<details>
<summary>event manifest file </summary>
    <ArgoCDEventManifest/></details>
6.使用 `kubectl` 将 YAML 文件引用到集群。运行以下命令，将 `<your-namespace>` 替换为 ArgoCD 名称空间，将 `<path-to-yaml-file>` 替换为 YAML 文件的实际路径: 

```bash
kubectl apply -n <your-namespace> -f <path-to-yaml-file>
```

该命令将 webhook 通知配置部署到 ArgoCD 通知 configmaps (`argocd-notifications-cm`)，使 Port 能够接收实时事件。

完成！您在 ArgoCD 中的应用程序发生的任何变化都将触发指向 Port 提供的 webhook URL 的 webhook 事件。 Port 将根据映射解析事件，并相应地更新目录实体。

</details>