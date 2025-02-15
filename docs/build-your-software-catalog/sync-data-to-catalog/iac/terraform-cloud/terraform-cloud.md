---
sidebar_position: 1
title: Terraform 云

description: Port 中的 Terraform 集成

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import DockerParameters from "./_terraform_one_time_docker_parameters.mdx"

# Terraform 云

Terraform Cloud Integration for Port 可将 Terraform 基础架构管理中的 "组织"、"项目"、"工作空间"、"运行 "和 "状态版本 "无缝导入和同步到 Port 中。 通过该集成，您可以在 Port 平台中有效监控和管理 Terraform Cloud 工作空间和运行。

组织 "是一个或多个团队协作工作空间的共享空间。

Terraform 云中的 "项目 "是基础架构配置的集合，通常与代码库相对应。 它作为主要的组织单位，将相关的 "Workspace"、"run "和 "state 版本 "分组，以管理和构建 Terraform 代码，实现高效部署和协作。

Workspace "表示 Terraform 云中的一个工作区。 工作区是 Terraform 管理基础设施(如一组云资源)的逻辑环境。

运行"(Run)表示在 Workspace 中执行的 Terraform 操作(计划、应用或销毁)的一个实例。 每个运行保存有关操作状态、持续时间和其他相关元数据的信息。

状态版本 "代表 Terraform 中的版本化状态文件。 每个状态版本都是不可变的，代表受管基础架构在特定时间点的状态。 状态版本被用于来跟踪基础架构中的变更，有助于审计、回滚和历史分析。

## 常见被用于情况

* 同步基础架构管理: 自动将 Terraform Cloud 中的 Workspace、运行和状态版本数据同步到 Port 中，以便集中跟踪和管理。
* 监控运行状态: 跟踪运行结果(成功、失败等)和持续时间，深入了解基础架构管理流程的健康状况和性能。
* 识别 Terraform 配置与云中有效部署之间的偏差。

## 先决条件

要安装集成，你需要一个 Kubernetes 集群，集成的容器图将部署到该集群中。

请确保您的计算机上安装了[`kubectl`](https://kubernetes.io/docs/tasks/tools/#kubectl) 和[`helm`](https://helm.sh/) ，并确保您的 `kubectl` CLI 已连接到计划安装集成的 Kubernetes 集群。

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                                | Description                                                                                                   | Required |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| `port.clientId`                          | Your Port client id                                                                                           | ✅       |
| `port.clientSecret`                      | Your Port client secret                                                                                       | ✅       |
| `port.baseUrl`                           | Your Port base url, relevant only if not using the default Port app                                           | ❌       |
| `integration.identifier`                 | Change the identifier to describe your integration                                                            | ✅       |
| `integration.type`                       | The integration type                                                                                          | ✅       |
| `integration.eventListener.type`         | The event listener type                                                                                       | ✅       |
| `integration.config.terraformCloudHost` | Your Terraform host. For example https://app.terraform.io token                                                                           | ✅       |
| `integration.config.terraformCloudToken` | The Terraform cloud API token                                                                           | ✅       |
| `integration.config.appHost`             | Your application's host url                                                                                   | ❌       |
| `scheduledResyncInterval`                | The number of minutes between each resync                                                                     | ❌       |
| `initializePortResources`                | Default true, When set to true the integration will create default blueprints and the port App config Mapping | ❌       |


<br/>
<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm">
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install terraform port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set port.baseUrl="https://api.getport.io"  \
    --set initializePortResources=true  \
    --set integration.identifier="my-terraform-cloud-integration"  \
    --set integration.type="terraform-cloud"  \
    --set integration.eventListener.type="POLLING"  \
        --set integration.secrets.terraformCloudHost="string" \
    --set integration.secrets.terraformCloudToken="string"
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在你的 git 仓库的 `argocd/my-ocean-terraform-cloud-integration` 中创建一个 `values.yaml` 文件，内容如下: 

:::note 请记住替换 `TERRAFORM_CLOUD_HOST` 和 `TERRAFORM_CLOUD_TOKEN` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-terraform-cloud-integration
  type: terraform-cloud
  eventListener:
    type: POLLING
  secrets:
  // highlight-start
    terraformCloudHost: TERRAFORM_CLOUD_HOST
    terraformCloudToken: TERRAFORM_CLOUD_TOKEN
  // highlight-end
```

<br/>

2.创建以下 "my-ocean-terraform-cloud-integration.yaml "配置清单，安装 "my-ocean-terraform-cloud-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-terraform-cloud-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-terraform-cloud-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-terraform-cloud-integration/values.yaml
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
kubectl apply -f my-ocean-terraform-cloud-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">

 <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Terraform cloud integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成能实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters/>

<br/>

下面是 `terraform-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Terraform Exporter Workflow

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Terraform Cloud Integration
        run: |
          # Set Docker image and run the container
          integration_type="terraform-cloud"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_HOST=${{ secrets.OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_HOST }} \
          -e OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_TOKEN }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Terraform cloud integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Terraform Cloud Credentials](https://www.jenkins.io/doc/book/using/using-credentials/) 的 "Secret Text "类型: 

<DockerParameters/>

<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

```yml showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run Terraform Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_HOST', variable: 'OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_HOST'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_TOKEN'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET')
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="terraform-cloud"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__TERRAFORM_COUD_HOST=$OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_HOST \
                                -e OCEAN__INTEGRATION__CONFIG__TERRAFORM_COUD_TOKEN=$OCEAN__INTEGRATION__CONFIG__TERRAFORM_CLOUD_TOKEN \
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

### 事件监听器

该集成使用轮询方式，每分钟从 Port 中提取一次配置，并检查配置是否有变化。 如果有变化，就会重新同步。

## 摄取 Terraform 云对象

Terraform 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 Terraform 云获取 "Workspace "的过程: 

```yaml showLineNumbers
resources:
  - kind: workspace
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id
          title: .attributes.name
          blueprint: '"terrafomCloudWorkspace"'
          properties:
            organization: .relationships.organization.data.id
            createdAt: .attributes."created-at"
            updatedAt: .attributes."updated-at"
            terraformVersion: .attributes."terraform-version"
            locked: .attributes.locked
            executionMode: .attributes."execution-mode"
            resourceCount: .attributes."resource-count"
            latestChangeAt: .attributes."latest-change-at"
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 Terraform API 事件中的现有字段和值进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了将从 Terraform Cloud 查询哪些资源，以及将在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可用于映射 Terraform Cloud 中的数据，可以引用下面链接的 API 响应中出现的任何字段进行映射配置。

* * [`Organization`](https://developer.hashicorp.com/terraform/cloud-docs/api-docs/organizations)
* [`Project`](https://developer.hashicorp.com/terraform/cloud-docs/api-docs/projects)
* [`Workspace`](https://www.terraform.io/docs/cloud/api/workspaces.html)
* [`Run`](https://www.terraform.io/docs/cloud/api/runs.html)
* [`State Version`](https://developer.hashicorp.com/terraform/cloud-docs/api-docs/state-versions)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: workspace
      selector:
      ...
  ```


* 类型 "键是 Terraform 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: run
        selector:
        ...
  ```


* Port"、"实体 "和 "映射 "键被用来将 Terraform Cloud 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；

```yaml showLineNumbers
resources:
  - kind: workspace
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .id
          title: .attributes.name
          blueprint: '"terrafomCloudWorkspace"'
          properties:
            organization: .relationships.organization.data.id
            createdAt: .attributes."created-at"
            updatedAt: .attributes."updated-at"
            terraformVersion: .attributes."terraform-version"
            locked: .attributes.locked
            executionMode: .attributes."execution-mode"
            resourceCount: .attributes."resource-count"
            latestChangeAt: .attributes."latest-change-at"
          relations:
            currentStateVersion: .relationships."current-state-version".data.id
```

:::tip  Blueprint key 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，则需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再使用一对双引号 (`"`)。

:::

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Terraform Cloud 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要使用 Terraform Cloud 进行引用的蓝图。
3. 从菜单中选择**摄取数据**选项。
4. 选择 IaC 类别下的 Terraform Cloud。
5. 将[integration configuration](#configuration-structure) 的内容添加到编辑器中。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

#### 组织

<details>
<summary>Organization blueprint</summary>

```json showLineNumbers
{
    "identifier": "terraformCloudOrganization",
    "description": "This blueprint represents an organization in Terraform Cloud",
    "title": "Terraform Cloud Organization",
    "icon": "Terraform",
    "schema": {
      "properties": {
        "externalId": {
          "type": "string",
          "title": "External ID",
          "description": "The external ID of the organization"
        },
        "ownerEmail": {
          "type": "string",
          "title": "Owner Email",
          "description": "The email associated with the organization"
        },
        "collaboratorAuthPolicy": {
          "type": "string",
          "title": "Collaborator Authentication Policy",
          "description": "Policy for collaborator authentication"
        },
        "planExpired": {
          "type": "string",
          "title": "Plan Expired",
          "description": "Indicates if plan is expired"
        },
        "planExpiresAt": {
          "type": "string",
          "format": "date-time",
          "title": "Plan Expiry Date",
          "description": "The data and time which the plan expires"
        },
        "permissions": {
          "type": "object",
          "title": "Permissions",
          "description": "Permissions associated with the organization"
        },
        "samlEnabled": {
          "type": "boolean",
          "title": "SAML Enabled",
          "description": "Indicates if SAML is enabled for the organization"
        },
        "defaultExecutionMode": {
          "type": "string",
          "title": "Default Execution Mode",
          "description": "The default execution mode for the organization"
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
- kind: organization
  selector:
    query: "true"
  port:
    entity:
      mappings:
        identifier: .id
        title: .attributes.name
        blueprint: '"terraformCloudOrganization"'
        properties:
          externalId: .attributes."external-id"
          ownerEmail: .attributes.email
          collaboratorAuthPolicy: .attributes."collaborator-auth-policy"
          planExpired: .attributes."plan-expired"
          planExpiresAt: .attributes."plan-expires-at"
          permissions: .attributes.permissions
          samlEnabled: .attributes."saml-enabled"
          defaultExecutionMode: .attributes."default-execution-mode"
```

</details>

### 项目

<details>
<summary>Project blueprint</summary>

```json showLineNumbers
{
    "identifier": "terraformCloudProject",
    "description": "This blueprint represents a project in Terraform Cloud",
    "title": "Terraform Cloud Project",
    "icon": "Terraform",
    "schema": {
      "properties": {
        "name": {
          "type": "string",
          "title": "Project Name",
          "description": "The name of the Terraform Cloud project"
        },
        "permissions": {
          "type": "object",
          "title": "Permissions",
          "description": "The permisssions on the project"
        }
      }
    },
    "mirrorProperties": {},
    "calculationProperties": {},
    "aggregationProperties": {},
    "relations": {
      "organization": {
        "title": "Terraform Cloud Organization",
        "target": "terraformCloudOrganization",
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
    query: "true"
  port:
    entity:
      mappings:
        identifier: .id
        title: .attributes.name
        blueprint: '"terraformCloudProject"'
        properties:
          name: .attributes.name
          permissions: .attributes.permissions
        relations:
          organization: .relationships.organization.data.id
```

</details>

### Workspace

<details>
<summary>Workspace blueprint</summary>

```json showLineNumbers
{
    "identifier": "terraformCloudWorkspace",
    "description": "This blueprint represents a workspace in Terraform Cloud",
    "title": "Terraform Cloud Workspace",
    "icon": "Terraform",
    "schema": {
      "properties": {
        "organization": {
          "type": "string",
          "title": "Organization",
          "description": "The organization within which the workspace belongs to"
        },
        "createdAt": {
          "type": "string",
          "format": "date-time",
          "title": "Creation Time",
          "description": "The creation timestamp of the workspace"
        },
        "updatedAt": {
          "type": "string",
          "format": "date-time",
          "title": "Last Updated",
          "description": "The last update timestamp of the workspace"
        },
        "terraformVersion": {
          "type": "string",
          "title": "Terraform Cloud Version",
          "description": "Version of Terraform cloud used by the workspace"
        },
        "locked": {
          "type": "boolean",
          "title": "Locked Status",
          "description": "Indicates whether the workspace is locked"
        },
        "executionMode": {
          "type": "string",
          "title": "Execution Mode",
          "description": "The execution mode of the workspace"
        },
        "resourceCount": {
          "type": "number",
          "title": "Resource Count",
          "description": "Number of resources managed by the workspace"
        },
        "latestChangeAt": {
          "type": "string",
          "format": "date-time",
          "title": "Latest Change",
          "description": "Timestamp of the latest change in the workspace"
        },
        "tags": {
          "type": "array",
          "title": "Workspace Tags",
          "description": "Terraform workspace tags"
        }
      }
    },
    "mirrorProperties": {},
    "calculationProperties": {},
    "aggregationProperties": {},
    "relations": {
      "currentStateVersion": {
        "title": "Current State Version",
        "target": "terraformCloudStateVersion",
        "required": false,
        "many": false
      },
      "project": {
        "title": "Terraform Cloud Project",
        "target": "terraformCloudProject",
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
- kind: workspace
  selector:
    query: "true"
  port:
    entity:
      mappings:
        identifier: .id
        title: .attributes.name
        blueprint: '"terraformCloudWorkspace"'
        properties:
          organization: .relationships.organization.data.id
          createdAt: .attributes."created-at"
          updatedAt: .attributes."updated-at"
          terraformVersion: .attributes."terraform-version"
          locked: .attributes.locked
          executionMode: .attributes."execution-mode"
          resourceCount: .attributes."resource-count"
          latestChangeAt: .attributes."latest-change-at"
          tags: .__tags
        relations:
          currentStateVersion: .relationships."current-state-version".data.id
          project: .relationships.project.data.id
```

</details>

#### 运行

<details>
<summary>Run blueprint</summary>

```json showLineNumbers
{
  "identifier": "terraformCloudRun",
  "description": "This blueprint represents a run in Terraform cloud",
  "title": "Terraform Cloud Run",
  "icon": "Terraform",
  "schema": {
    "properties": {
      "createdAt": {
        "type": "string",
        "format": "date-time",
        "title": "Creation Time",
        "description": "The creation timestamp of the run"
      },
      "status": {
        "type": "string",
        "title": "Run Status",
        "description": "The current status of the run"
      },
      "hasChanges": {
        "type": "boolean",
        "title": "Has Changes",
        "description": "Indicates whether the run has changes"
      },
      "isDestroy": {
        "type": "boolean",
        "title": "Is Destroy",
        "description": "Indicates whether the run is a destroy operation"
      },
      "message": {
        "type": "string",
        "title": "Run Message",
        "description": "Message associated with the run"
      },
      "terraformVersion": {
        "type": "string",
        "title": "Terraform Cloud Version",
        "description": "Version of Terraform cloud used in the run"
      },
      "appliedAt": {
        "type": "string",
        "format": "date-time",
        "title": "Applied Time",
        "description": "Timestamp when the run was applied"
      },
      "plannedAt": {
        "type": "string",
        "format": "date-time",
        "title": "Planned Time",
        "description": "Timestamp when the run was planned"
      },
      "source": {
        "type": "string",
        "title": "Run Source",
        "description": "The source of the run initiation"
      }
    }
  },
  "relations": {
    "terraformCloudWorkspace": {
      "title": "Terraform Cloud Workspace",
      "target": "terraformCloudWorkspace",
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
- kind: run
  selector:
    query: "true"
  port:
    entity:
      mappings:
        identifier: .id
        title: .attributes.message
        blueprint: '"terraformCloudRun"'
        properties:
          createdAt: .attributes."created-at"
          status: .attributes.status
          hasChanges: .attributes."has-changes"
          isDestroy: .attributes."is-destroy"
          message: .attributes.message
          terraformVersion: .attributes."terraform-version"
          appliedAt: .attributes."status-timestamps"."applied-at"
          plannedAt: .attributes."status-timestamps"."planned-at"
          source: .attributes.source
        relations:
          terraformCloudWorkspace: .relationships.workspace.data.id
```

</details>

#### 国家版本

<details>
<summary>State Version blueprint</summary>

```json showLineNumbers
{
  "identifier": "terraformCloudStateVersion",
  "description": "This blueprint represents a version of a Terraform state version",
  "title": "Terraform Cloud State Versions",
  "icon": "Terraform",
  "schema": {
    "properties": {
      "createdAt": {
        "type": "string",
        "format": "date-time",
        "title": "Creation Time",
        "description": "Timestamp when the state version was created"
      },
      "serial": {
        "type": "number",
        "title": "Serial Number",
        "description": "A unique identifier for this version within the workspace"
      },
      "status": {
        "type": "string",
        "title": "Status",
        "description": "The current status of the state version (e.g., 'queued', 'finished')"
      },
      "size": {
        "type": "number",
        "title": "Size",
        "description": "The size of the resources"
      },
      "isResourcesProcessed": {
        "type": "boolean",
        "title": "Is Resources Processed",
        "description": "Whethere the resources has been processed"
      },
      "hostedStateDownloadUrl": {
        "type": "string",
        "title": "Download Url",
        "format": "url",
        "description": "Hosted state version download url "
      },
      "hostedJsonDownloadUrl": {
        "type": "string",
        "title": "Download Url",
        "format": "url",
        "description": "Url for downloading state version in json format"
      },
      "outputData": {
        "type": "array",
        "title": "Output",
        "description": "output returned from state version"
      },
      "vcsCommitUrl": {
        "type": "string",
        "title": "VCS Commit URL",
        "format": "url",
        "description": "URL of the VCS commit that triggered this state version"
      }
    }
  },
  "relations": {},
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {}
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
- kind: state-version
  selector:
    query: "true"
  port:
    entity:
      mappings:
        identifier: .id
        title: .id
        blueprint: '"terraformCloudStateVersion"'
        properties:
          createdAt: .attributes."created-at"
          serial: .attributes.serial
          status: .attributes.status
          size: .attributes.size
          isResourcesProcessed: .attributes."resources-processed"
          hostedStateDownloadUrl: .attributes."hosted-state-download-url"
          hostedJsonDownloadUrl: .attributes."hosted-json-state-download-url"
          vcsCommitUrl: .attributes."vcs-commit-url"
          outputData: .__output
```

</details>

## 让我们来测试一下

本节包括来自 Terrform Cloud 的响应数据示例。 此外，还包括根据上一节提供的 Ocean 配置从重新同步事件中创建的实体。

### 有效载荷

下面是 Terraform 提供的有效载荷结构示例: 

<details>
<summary> Organization response data</summary>

```json showLineNumbers
{
  "id":"example-org-484f07",
  "type":"organizations",
  "attributes":{
      "external-id":"org-S8Wo1fyUtpSFHbGQ",
      "created-at":"2023-12-19T14:16:20.491Z",
      "email":"example@getport.io",
      "session-timeout":"None",
      "session-remember":"None",
      "collaborator-auth-policy":"password",
      "plan-expired":false,
      "plan-expires-at":"2024-01-18T14:16:20.637Z",
      "plan-is-trial":true,
      "plan-is-enterprise":false,
      "plan-identifier":"free_standard",
      "cost-estimation-enabled":false,
      "managed-resource-count":5,
      "send-passing-statuses-for-untriggered-speculative-plans":false,
      "allow-force-delete-workspaces":false,
      "assessments-enforced":false,
      "is-in-degraded-mode":false,
      "default-execution-mode":"remote",
      "remaining-testable-count":5,
      "aggregated-commit-status-enabled":false,
      "name":"example-org-484f07",
      "permissions":{
        "can-update":true,
        "can-update-authentication":true,
        "can-destroy":true,
        "can-access-via-teams":true,
        "can-create-module":true,
        "can-create-team":false,
        "can-create-workspace":true,
        "can-manage-users":true,
        "can-manage-subscription":true,
        "can-manage-sso":true,
        "can-update-oauth":true,
        "can-update-sentinel":true,
        "can-update-ssh-keys":true,
        "can-update-api-token":true,
        "can-traverse":true,
        "can-view-usage":true,
        "can-start-trial":false,
        "can-update-agent-pools":true,
        "can-manage-tags":true,
        "can-manage-varsets":true,
        "can-read-varsets":true,
        "can-manage-public-providers":true,
        "can-create-provider":true,
        "can-manage-public-modules":true,
        "can-manage-custom-providers":true,
        "can-manage-run-tasks":true,
        "can-read-run-tasks":true,
        "can-create-project":true,
        "can-manage-assessments":true,
        "can-read-assessments":true,
        "can-view-explorer":true,
        "can-deploy-no-code-modules":false,
        "can-manage-no-code-modules":false,
        "can-use-new-pnp-activation-ui":true
      },
      "saml-enabled":false,
      "fair-run-queuing-enabled":false,
      "owners-team-saml-role-id":"None",
      "two-factor-conformant":true
  },
  "relationships":{
      "default-agent-pool":{
        "data":"None"
      },
      "meta":{
        "links":{
            "related":"/api/v2/organizations/example-org-484f07/meta"
        }
      },
      "oauth-tokens":{
        "links":{
            "related":"/api/v2/organizations/example-org-484f07/oauth-tokens"
        }
      },
      "authentication-token":{
        "links":{
            "related":"/api/v2/organizations/example-org-484f07/authentication-token"
        }
      },
      "entitlement-set":{
        "data":{
            "id":"org-S8Wo1fyUtpSFHbGQ",
            "type":"entitlement-sets"
        },
        "links":{
            "related":"/api/v2/organizations/example-org-484f07/entitlement-set"
        }
      },
      "subscription":{
        "data":{
            "id":"sub-THrhah4DkbD4etzy",
            "type":"subscriptions"
        },
        "links":{
            "related":"/api/v2/organizations/example-org-484f07/subscription"
        }
      },
      "default-project":{
        "data":{
            "id":"prj-d1JbKcLJBhwN66Vs",
            "type":"projects"
        },
        "links":{
            "related":"/api/v2/projects/prj-d1JbKcLJBhwN66Vs"
        }
      }
  },
  "links":{
      "self":"/api/v2/organizations/example-org-484f07"
  }
}
```

</details>

<details>
<summary> Workspace response data</summary>

```json showLineNumbers
{
  "id":"ws-DBJebej1fNCZomkr",
  "type":"workspaces",
  "attributes":{
      "allow-destroy-plan":true,
      "auto-apply":false,
      "auto-apply-run-trigger":false,
      "auto-destroy-activity-duration":"None",
      "auto-destroy-at":"None",
      "auto-destroy-status":"None",
      "created-at":"2023-12-12T12:43:36.192Z",
      "environment":"default",
      "locked":false,
      "name":"example-workspace-2",
      "queue-all-runs":false,
      "speculative-enabled":true,
      "structured-run-output-enabled":true,
      "terraform-version":"1.6.5",
      "working-directory":"",
      "global-remote-state":false,
      "updated-at":"2023-12-12T12:43:36.192Z",
      "resource-count":0,
      "apply-duration-average":"None",
      "plan-duration-average":"None",
      "policy-check-failures":"None",
      "run-failures":"None",
      "workspace-kpis-runs-count":"None",
      "latest-change-at":"2023-12-12T12:43:36.192Z",
      "operations":true,
      "execution-mode":"remote",
      "vcs-repo":"None",
      "vcs-repo-identifier":"None",
      "permissions":{
        "can-update":true,
        "can-destroy":true,
        "can-queue-run":true,
        "can-read-variable":true,
        "can-update-variable":true,
        "can-read-state-versions":true,
        "can-read-state-outputs":true,
        "can-create-state-versions":true,
        "can-queue-apply":true,
        "can-lock":true,
        "can-unlock":true,
        "can-force-unlock":true,
        "can-read-settings":true,
        "can-manage-tags":true,
        "can-manage-run-tasks":true,
        "can-force-delete":true,
        "can-manage-assessments":true,
        "can-manage-ephemeral-workspaces":false,
        "can-read-assessment-results":true,
        "can-queue-destroy":true
      },
      "actions":{
        "is-destroyable":true
      },
      "description":"None",
      "file-triggers-enabled":false,
      "trigger-prefixes":[

      ],
      "trigger-patterns":[

      ],
      "assessments-enabled":false,
      "last-assessment-result-at":"None",
      "source":"tfe-ui",
      "source-name":"None",
      "source-url":"None",
      "tag-names":[
        "foo",
        "bar"
      ],
      "setting-overwrites":{
        "execution-mode":false,
        "agent-pool":false
      }
  },
  "relationships":{
      "organization":{
        "data":{
            "id":"example-org-162af6",
            "type":"organizations"
        }
      },
      "current-run":{
        "data":"None"
      },
      "latest-run":{
        "data":"None"
      },
      "outputs":{
        "data":[

        ]
      },
      "remote-state-consumers":{
        "links":{
            "related":"/api/v2/workspaces/ws-DBJebej1fNCZomkr/relationships/remote-state-consumers"
        }
      },
      "current-state-version":{
        "data":"None"
      },
      "current-configuration-version":{
        "data":"None"
      },
      "agent-pool":{
        "data":"None"
      },
      "readme":{
        "data":"None"
      },
      "project":{
        "data":{
            "id":"prj-wnLLjhXa3XArrRFR",
            "type":"projects"
        }
      },
      "current-assessment-result":{
        "data":"None"
      },
      "vars":{
        "data":[

        ]
      }
  },
  "links":{
      "self":"/api/v2/organizations/example-org-162af6/workspaces/example-workspace-2",
      "self-html":"/app/example-org-162af6/workspaces/example-workspace-2"
  },
  "__tags":[
      {
        "id":"tag-moR1pPNpT2vowy55",
        "type":"tags",
        "attributes":{
            "name":"foo",
            "created-at":"2024-01-09T19:41:45.183Z",
            "instance-count":1
        },
        "relationships":{
            "organization":{
              "data":{
                  "id":"example-org-162af6",
                  "type":"organizations"
              }
            }
        }
      },
      {
        "id":"tag-PNyYYGibnxZcnVho",
        "type":"tags",
        "attributes":{
            "name":"bar",
            "created-at":"2024-01-09T19:41:45.197Z",
            "instance-count":1
        },
        "relationships":{
            "organization":{
              "data":{
                  "id":"example-org-162af6",
                  "type":"organizations"
              }
            }
        }
      }
  ]
}
```

</details>

<details>
<summary> Run response data</summary>

```json showLineNumbers
{
  "data": [
    {
      "id": "run-SFSeL9fg6Kibje8L",
      "type": "runs",
      "attributes": {
        "actions": {
          "is-cancelable": false,
          "is-confirmable": false,
          "is-discardable": false,
          "is-force-cancelable": false
        },
        "allow-config-generation": true,
        "allow-empty-apply": false,
        "auto-apply": false,
        "canceled-at": "None",
        "created-at": "2023-12-13T12:12:40.252Z",
        "has-changes": false,
        "is-destroy": false,
        "message": "just checking this out",
        "plan-only": false,
        "refresh": true,
        "refresh-only": false,
        "replace-addrs": [],
        "save-plan": false,
        "source": "tfe-ui",
        "status-timestamps": {
          "planned-at": "2023-12-13T12:12:54+00:00",
          "queuing-at": "2023-12-13T12:12:40+00:00",
          "planning-at": "2023-12-13T12:12:49+00:00",
          "plan-queued-at": "2023-12-13T12:12:40+00:00",
          "plan-queueable-at": "2023-12-13T12:12:40+00:00",
          "planned-and-finished-at": "2023-12-13T12:12:54+00:00"
        },
        "status": "planned_and_finished",
        "target-addrs": "None",
        "trigger-reason": "manual",
        "terraform-version": "1.6.5",
        "permissions": {
          "can-apply": true,
          "can-cancel": true,
          "can-comment": true,
          "can-discard": true,
          "can-force-execute": true,
          "can-force-cancel": true,
          "can-override-policy-check": true
        },
        "variables": []
      },
      "relationships": {
        "workspace": {
          "data": {
            "id": "ws-WWhD18B59v5ndTTP",
            "type": "workspaces"
          }
        },
        "apply": {
          "data": {
            "id": "apply-ToVWRgBe4mmGwTf7",
            "type": "applies"
          },
          "links": {
            "related": "/api/v2/runs/run-SFSeL9fg6Kibje8L/apply"
          }
        },
        "configuration-version": {
          "data": {
            "id": "cv-ompZmuF15X68njap",
            "type": "configuration-versions"
          },
          "links": {
            "related": "/api/v2/runs/run-SFSeL9fg6Kibje8L/configuration-version"
          }
        },
        "created-by": {
          "data": {
            "id": "user-Vg6uYxyhrQSHNrKU",
            "type": "users"
          },
          "links": {
            "related": "/api/v2/runs/run-SFSeL9fg6Kibje8L/created-by"
          }
        },
        "plan": {
          "data": {
            "id": "plan-3rXS4BMT8TEkdchh",
            "type": "plans"
          },
          "links": {
            "related": "/api/v2/runs/run-SFSeL9fg6Kibje8L/plan"
          }
        },
        "run-events": {
          "data": [
            {
              "id": "re-WgvYmckRJjafwU5R",
              "type": "run-events"
            },
            {
              "id": "re-46PfZixftNeifEG9",
              "type": "run-events"
            },
            {
              "id": "re-LCCwB2pQNPrGnveF",
              "type": "run-events"
            },
            {
              "id": "re-YoviSEov4cscqfi7",
              "type": "run-events"
            }
          ],
          "links": {
            "related": "/api/v2/runs/run-SFSeL9fg6Kibje8L/run-events"
          }
        },
        "task-stages": {
          "data": [],
          "links": {
            "related": "/api/v2/runs/run-SFSeL9fg6Kibje8L/task-stages"
          }
        },
        "policy-checks": {
          "data": [],
          "links": {
            "related": "/api/v2/runs/run-SFSeL9fg6Kibje8L/policy-checks"
          }
        },
        "comments": {
          "data": [],
          "links": {
            "related": "/api/v2/runs/run-SFSeL9fg6Kibje8L/comments"
          }
        }
      },
      "links": {
        "self": "/api/v2/runs/run-SFSeL9fg6Kibje8L"
      }
    }
  ]
}
```

</details>

<details>
<summary> State Version response data</summary>

```json showLineNumbers
{
  "id": "sv-wZEoyjPg1KYsjYZg",
  "type": "state-versions",
  "attributes": {
    "created-at": "2023-12-20T07:08:14.113Z",
    "size": 8554,
    "hosted-state-download-url": "https://app.terraform.io/api/state-versions/sv-wZEoyjPg1KYsjYZg/hosted_state",
    "hosted-json-state-download-url": "https://app.terraform.io/api/state-versions/sv-wZEoyjPg1KYsjYZg/hosted_json_state",
    "modules": {
      "root": {
        "random_string": 1,
        "aws_db_instance": 1,
        "aws_db_subnet_group": 1,
        "data.aws_availability_zones": 1
      },
      "root.vpc": {
        "aws_eip": 2,
        "aws_vpc": 1,
        "aws_route": 3,
        "aws_subnet": 4,
        "aws_nat_gateway": 2,
        "aws_route_table": 3,
        "aws_internet_gateway": 1,
        "aws_route_table_association": 4
      },
      "root.elb-http.elb": {
        "aws_elb": 1
      },
      "root.ec2-instances": {
        "aws_instance": 4,
        "data.aws_ami": 1
      },
      "root.lb-security-group.sg": {
        "aws_security_group": 1,
        "aws_security_group_rule": 6
      },
      "root.app-security-group.sg": {
        "aws_security_group": 1,
        "aws_security_group_rule": 6
      },
      "root.elb-http.elb-attachment": {
        "aws_elb_attachment": 4
      }
    },
    "providers": {
      "provider[\"registry.terraform.io/hashicorp/aws\"]": {
        "aws_eip": 2,
        "aws_elb": 1,
        "aws_vpc": 1,
        "aws_route": 3,
        "aws_subnet": 4,
        "aws_instance": 4,
        "data.aws_ami": 1,
        "aws_db_instance": 1,
        "aws_nat_gateway": 2,
        "aws_route_table": 3,
        "aws_elb_attachment": 4,
        "aws_security_group": 2,
        "aws_db_subnet_group": 1,
        "aws_internet_gateway": 1,
        "aws_security_group_rule": 12,
        "aws_route_table_association": 4,
        "data.aws_availability_zones": 1
      },
      "provider[\"registry.terraform.io/hashicorp/random\"]": {
        "random_string": 1
      }
    },
    "resources": [
      {
        "name": "available",
        "type": "data.aws_availability_zones",
        "count": 1,
        "module": "root",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "database",
        "type": "aws_db_instance",
        "count": 1,
        "module": "root",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "private",
        "type": "aws_db_subnet_group",
        "count": 1,
        "module": "root",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "lb_id",
        "type": "random_string",
        "count": 1,
        "module": "root",
        "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
        "index-keys": []
      },
      {
        "name": "this_name_prefix",
        "type": "aws_security_group",
        "count": 1,
        "module": "root.app_security_group.sg",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "egress_rules",
        "type": "aws_security_group_rule",
        "count": 1,
        "module": "root.app_security_group.sg",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "ingress_rules",
        "type": "aws_security_group_rule",
        "count": 4,
        "module": "root.app_security_group.sg",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "ingress_with_self",
        "type": "aws_security_group_rule",
        "count": 1,
        "module": "root.app_security_group.sg",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "amazon_linux",
        "type": "data.aws_ami",
        "count": 1,
        "module": "root.ec2_instances",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "app",
        "type": "aws_instance",
        "count": 4,
        "module": "root.ec2_instances",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "this",
        "type": "aws_elb",
        "count": 1,
        "module": "root.elb_http.elb",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "this",
        "type": "aws_elb_attachment",
        "count": 4,
        "module": "root.elb_http.elb_attachment",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "this_name_prefix",
        "type": "aws_security_group",
        "count": 1,
        "module": "root.lb_security_group.sg",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "egress_rules",
        "type": "aws_security_group_rule",
        "count": 1,
        "module": "root.lb_security_group.sg",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "ingress_rules",
        "type": "aws_security_group_rule",
        "count": 4,
        "module": "root.lb_security_group.sg",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "ingress_with_self",
        "type": "aws_security_group_rule",
        "count": 1,
        "module": "root.lb_security_group.sg",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "nat",
        "type": "aws_eip",
        "count": 2,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "this",
        "type": "aws_internet_gateway",
        "count": 1,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "this",
        "type": "aws_nat_gateway",
        "count": 2,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "private_nat_gateway",
        "type": "aws_route",
        "count": 2,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "public_internet_gateway",
        "type": "aws_route",
        "count": 1,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "private",
        "type": "aws_route_table",
        "count": 2,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "public",
        "type": "aws_route_table",
        "count": 1,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "private",
        "type": "aws_route_table_association",
        "count": 2,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "public",
        "type": "aws_route_table_association",
        "count": 2,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "private",
        "type": "aws_subnet",
        "count": 2,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "public",
        "type": "aws_subnet",
        "count": 2,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      },
      {
        "name": "this",
        "type": "aws_vpc",
        "count": 1,
        "module": "root.vpc",
        "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
        "index-keys": []
      }
    ],
    "resources-processed": true,
    "serial": 3,
    "state-version": 4,
    "status": "finalized",
    "terraform-version": "1.6.6",
    "vcs-commit-url": "None",
    "vcs-commit-sha": "None"
  },
  "relationships": {
    "run": {
      "data": {
        "id": "run-9XBzSuHzfgwTZNak",
        "type": "runs"
      }
    },
    "rollback-state-version": {
      "data": "None"
    },
    "created-by": {
      "data": {
        "id": "user-VGzYMVaTX6nVYZ3U",
        "type": "users"
      },
      "links": {
        "self": "/api/v2/users/user-VGzYMVaTX6nVYZ3U",
        "related": "/api/v2/runs/run-9XBzSuHzfgwTZNak/created-by"
      }
    },
    "workspace": {
      "data": {
        "id": "ws-pN259iR1J3cW8ivr",
        "type": "workspaces"
      }
    },
    "outputs": {
      "data": [
        {
          "id": "wsout-uJCPifEGM1fC5UF1",
          "type": "state-version-outputs"
        },
        {
          "id": "wsout-SD3QFXBkcK41c8Vq",
          "type": "state-version-outputs"
        },
        {
          "id": "wsout-ryDfZJ4dwxuqR5NU",
          "type": "state-version-outputs"
        },
        {
          "id": "wsout-5vnxsMuMaorwojVL",
          "type": "state-version-outputs"
        },
        {
          "id": "wsout-cP7abYGPeXpgJfrt",
          "type": "state-version-outputs"
        }
      ],
      "links": {
        "related": "/api/v2/state-versions/sv-wZEoyjPg1KYsjYZg/outputs"
      }
    }
  },
  "links": {
    "self": "/api/v2/state-versions/sv-wZEoyjPg1KYsjYZg"
  },
  "__output": [
    {
      "id": "wsout-syLHF4vVm1ELesRH",
      "type": "state-version-outputs",
      "attributes": {
        "name": "lb_url",
        "sensitive": false,
        "type": "string",
        "value": "http://lb-r4c-project-alpha-dev-81440499.eu-west-1.elb.amazonaws.com/",
        "detailed-type": "string"
      },
      "links": {
        "self": "/api/v2/state-version-outputs/wsout-syLHF4vVm1ELesRH"
      }
    },
    {
      "id": "wsout-yCuZCAT1MBrpW9rL",
      "type": "state-version-outputs",
      "attributes": {
        "name": "vpc_id",
        "sensitive": false,
        "type": "string",
        "value": "vpc-085a343aa3f06a9d7",
        "detailed-type": "string"
      },
      "links": {
        "self": "/api/v2/state-version-outputs/wsout-yCuZCAT1MBrpW9rL"
      }
    },
    {
      "id": "wsout-ujnf5q4ZS3LTEFmL",
      "type": "state-version-outputs",
      "attributes": {
        "name": "web_server_count",
        "sensitive": false,
        "type": "number",
        "value": 4,
        "detailed-type": "number"
      },
      "links": {
        "self": "/api/v2/state-version-outputs/wsout-ujnf5q4ZS3LTEFmL"
      }
    }
  ]
}
```

</details>

#### 映射结果

结合样本有效载荷和 Ocean 配置，可生成以下 Port 实体: 

<details>
<summary> organization entity in Port</summary>

```json showLineNumbers
{
  "identifier": "example-org-484f07",
  "title": "example-org-484f07",
  "team": [],
  "properties": {
    "externalId": "org-S8Wo1fyUtpSFHbGQ",
    "collaboratorAuthPolicy": "password",
    "permissions": {
      "can-update": true,
      "can-update-authentication": true,
      "can-destroy": true,
      "can-access-via-teams": true,
      "can-create-module": true,
      "can-create-team": false,
      "can-create-workspace": true,
      "can-manage-users": true,
      "can-manage-subscription": true,
      "can-manage-sso": true,
      "can-update-oauth": true,
      "can-update-sentinel": true,
      "can-update-ssh-keys": true,
      "can-update-api-token": true,
      "can-traverse": true,
      "can-view-usage": true,
      "can-start-trial": false,
      "can-update-agent-pools": true,
      "can-manage-tags": true,
      "can-manage-varsets": true,
      "can-read-varsets": true,
      "can-manage-public-providers": true,
      "can-create-provider": true,
      "can-manage-public-modules": true,
      "can-manage-custom-providers": true,
      "can-manage-run-tasks": true,
      "can-read-run-tasks": true,
      "can-create-project": true,
      "can-manage-assessments": true,
      "can-read-assessments": true,
      "can-view-explorer": true,
      "can-deploy-no-code-modules": false,
      "can-manage-no-code-modules": false,
      "can-use-new-pnp-activation-ui": true
    },
    "samlEnabled": false,
    "defaultExecutionMode": "remote",
    "ownerEmail": "example@getport.io",
    "planExpired": "false",
    "planExpiresAt": "2024-01-18T14:16:20.637Z"
  },
  "relations": {},
  "icon": "Terraform"
}
```

</details>

<details>
<summary> project entity in Port</summary>

```json showLineNumbers
{
  "identifier": "prj-wnLLjhXa3XArrRFR",
  "title": "Default Project",
  "team": [],
  "properties": {
    "name": "Default Project",
    "permissions": {
      "can-read": true,
      "can-update": true,
      "can-destroy": true,
      "can-create-workspace": true,
      "can-move-workspace": true,
      "can-deploy-no-code-modules": true,
      "can-read-teams": true,
      "can-manage-teams": true
    },
    "organizationId": "example-org-162af6"
  },
  "relations": {},
  "icon": "Terraform"
}
```

</details>

<details>
<summary> workspace entity in Port</summary>

```json showLineNumbers
{
  "identifier": "ws-DBJebej1fNCZomkr",
  "title": "example-workspace-2",
  "team": [],
  "properties": {
    "createdAt": "2023-12-12T12:43:36.192Z",
    "updatedAt": "2023-12-12T12:43:36.192Z",
    "terraformVersion": "1.6.5",
    "locked": false,
    "executionMode": "remote",
    "resourceCount": 0,
    "latestChangeAt": "2023-12-12T12:43:36.192Z",
    "organization": "example-org-162af6",
    "tags": [
      {
        "id": "tag-moR1pPNpT2vowy55",
        "type": "tags",
        "attributes": {
          "name": "foo",
          "created-at": "2024-01-09T19:41:45.183Z",
          "instance-count": 1
        },
        "relationships": {
          "organization": {
            "data": {
              "id": "example-org-162af6",
              "type": "organizations"
            }
          }
        }
      },
      {
        "id": "tag-PNyYYGibnxZcnVho",
        "type": "tags",
        "attributes": {
          "name": "bar",
          "created-at": "2024-01-09T19:41:45.197Z",
          "instance-count": 1
        },
        "relationships": {
          "organization": {
            "data": {
              "id": "example-org-162af6",
              "type": "organizations"
            }
          }
        }
      }
    ]
  },
  "relations": {
    "project": "prj-wnLLjhXa3XArrRFR"
  },
  "icon": "Terraform"
}
```

</details>

<details>
<summary> Run entity in Port</summary>

```json showLineNumbers
{
  "identifier": "run-SFSeL9fg6Kibje8L",
  "title": "just checking this out",
  "blueprint": "terraformRun",
  "properties": {
    "runId": "run-SFSeL9fg6Kibje8L",
    "createdAt": "2021-08-16T21:50:58.726Z",
    "status": "planned_and_finished",
    "hasChanges": false,
    "isDestroy": false,
    "message": "just checking this out",
    "terraformVersion": "0.11.1",
    "appliedAt": null,
    "plannedAt": "2023-12-13T12:12:54+00:00",
    "source": "tfe-api"
  },
  "relations": {}
}
```

</details>

<details>
<summary> State Version in Port</summary>

```json showLineNumbers
{
  "identifier": "sv-wZEoyjPg1KYsjYZg",
  "title": "sv-wZEoyjPg1KYsjYZg",
  "team": [],
  "properties": {
    "createdAt": "2023-12-20T07:08:14.113Z",
    "serial": 3,
    "status": "finalized",
    "hostedStateDownloadUrl": "https://app.terraform.io/api/state-versions/sv-wZEoyjPg1KYsjYZg/hosted_state",
    "hostedJsonDownloadUrl": "https://app.terraform.io/api/state-versions/sv-wZEoyjPg1KYsjYZg/hosted_json_state",
    "outputData": [
      {
        "id": "wsout-syLHF4vVm1ELesRH",
        "type": "state-version-outputs",
        "attributes": {
          "name": "lb_url",
          "sensitive": false,
          "type": "string",
          "value": "http://lb-r4c-project-alpha-dev-81440499.eu-west-1.elb.amazonaws.com/",
          "detailed-type": "string"
        },
        "links": {
          "self": "/api/v2/state-version-outputs/wsout-syLHF4vVm1ELesRH"
        }
      },
      {
        "id": "wsout-yCuZCAT1MBrpW9rL",
        "type": "state-version-outputs",
        "attributes": {
          "name": "vpc_id",
          "sensitive": false,
          "type": "string",
          "value": "vpc-085a343aa3f06a9d7",
          "detailed-type": "string"
        },
        "links": {
          "self": "/api/v2/state-version-outputs/wsout-yCuZCAT1MBrpW9rL"
        }
      },
      {
        "id": "wsout-ujnf5q4ZS3LTEFmL",
        "type": "state-version-outputs",
        "attributes": {
          "name": "web_server_count",
          "sensitive": false,
          "type": "number",
          "value": 4,
          "detailed-type": "number"
        },
        "links": {
          "self": "/api/v2/state-version-outputs/wsout-ujnf5q4ZS3LTEFmL"
        }
      }
    ],
    "size": 8554,
    "isResourcesProcessed": true
  },
  "relations": {},
  "icon": "Terraform"
}
```

</details>