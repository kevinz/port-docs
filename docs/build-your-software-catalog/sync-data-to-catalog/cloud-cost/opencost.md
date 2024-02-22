import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import DockerParameters from "./_opencost-docker-parameters.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'

# OpenCost

通过 OpenCost 集成，您可以根据您的映射和定义从 OpenCost 实例将 "成本 "导入 Port。

## 常见被用于情况

* 在 OpenCost 中映射受监控的 Kubernetes 资源和成本分配。

## 先决条件

<Prerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

被用于此安装选项意味着集成将能实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                         | Description                                                                                                   | Required |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| `port.clientId`                   | Your port client id                                                                                           | ✅       |
| `port.clientSecret`               | Your port client secret                                                                                       | ✅       |
| `port.baseUrl`                    | Your port base url, relevant only if not using the default port app                                           | ❌       |
| `integration.identifier`          | Change the identifier to describe your integration                                                            | ✅       |
| `integration.type`                | The integration type                                                                                          | ✅       |
| `integration.eventListener.type`  | The event listener type                                                                                       | ✅       |
| `integration.config.opencostHost` | The Opencost server URL                                                                                       | ✅       |
| `scheduledResyncInterval`         | The number of minutes between each resync                                                                     | ❌       |
| `initializePortResources`         | Default true, When set to true the integration will create default blueprints and the port App config Mapping | ❌       |


<br/>

<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-opencost-integration port-labs/port-ocean \
    --set port.clientId="CLIENT_ID"  \
    --set port.clientSecret="CLIENT_SECRET"  \
    --set initializePortResources=true  \
    --set integration.identifier="my-opencost-integration"  \
    --set integration.type="opencost"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.config.opencostHost="https://myOpenCostInstance:9003"
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在 git 仓库的`argocd/my-ocean-opencost-integration`中创建内容为`values.yaml`的文件: 

:::note 请记住替换 `OPENCOST_HOST` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-opencost-integration
  type: opencost
  eventListener:
    type: POLLING
  config:
  // highlight-next-line
    opencostHost: OPENCOST_HOST
```

<br/>

2.创建以下 "my-ocean-opencost-integration.yaml "配置清单，安装 "my-ocean-opencost-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-opencost-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-opencost-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-opencost-integration/values.yaml
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
kubectl apply -f my-ocean-opencost-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Opencost integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成能实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />

<br/>

以下是 `opencost-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Opencost Exporter Workflow

# This workflow responsible for running Opencost exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Opencost Integration
        run: |
          # Set Docker image and run the container
          integration_type="opencost"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__OPENCOST_HOST=${{ secrets.OCEAN__INTEGRATION__CONFIG__OPENCOST_HOST }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the OpenCost integration once and then exit, this is useful for **scheduled** ingestion of data.

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
        stage('Run Opencost Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__OPENCOST_HOST', variable: 'OCEAN__INTEGRATION__CONFIG__OPENCOST_HOST'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="opencost"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__OPENCOST_HOST=$OCEAN__INTEGRATION__CONFIG__OPENCOST_HOST \
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

<AzurePremise name="Kubecost" />

<DockerParameters />

<br/>

下面是 `opencost-integration.yml` Pipelines 文件的示例: 

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
    integration_type="opencost"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
        -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
        -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
        -e OCEAN__INTEGRATION__CONFIG__OPENCOST_HOST=${OCEAN__INTEGRATION__CONFIG__OPENCOST_HOST} \
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

## 接收 OpenCost 对象

OpenCost 集成使用 YAML 配置来描述将数据加载到开发人员门户的过程。

下面是配置中的一个示例片段，演示了从 OpenCost 获取 "成本 "数据的过程: 

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: cost
    selector:
      query: "true"
      window: "month"
      aggregate: "pod"
      step: "window"
      resolution: "1m"
    port:
      entity:
        mappings:
          blueprint: '"openCostResourceAllocation"'
          identifier: .name
          title: .name
          properties:
            cluster: .properties.cluster
            namespace: .properties.namespace
            startDate: .start
            endDate: .end
            cpuCoreHours: .cpuCoreHours
            cpuCost: .cpuCost
            cpuEfficiency: .cpuEfficiency
            gpuHours: .gpuHours
            gpuCost: .gpuCost
            networkCost: .networkCost
            loadBalancerCost: .loadBalancerCost
            pvCost: .pvCost
            ramBytes: .ramBytes
            ramCost: .ramCost
            ramEfficiency: .ramEfficiency
            sharedCost: .sharedCost
            externalCost: .externalCost
            totalCost: .totalCost
            totalEfficiency: .totalEfficiency
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 从 OpenCost 的 API 事件中对现有字段和 Values 进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了从 OpenCost 中查询哪些资源，以及在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被用于用于映射 OpenCost 中的数据，可以引用下面链接的 API 响应中出现的任何字段进行映射配置。

* * [`Cost`](https://www.opencost.io/docs/integrations/api-examples)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: cost
      selector:
      ...
  ```


* 类型 "键是 OpenCost 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: cost
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: cost
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
        window: "month"
        aggregate: "pod"
        step: "window"
        resolution: "1m"
      # highlight-end
      port:
  ```


* **window** - 要查询的时间长度。接受: "today"、"week"、"month"、"yesterday"、"lastweek"、"lastmonth "等单词；"30m"、"12h"、"7d "等持续时间；"2021-01-02T15:04:05Z,2021-02-02T15:04:05Z "等 RFC3339 日期对；"1578002645,1580681045 "等 Unix 时间戳。
* **aggregate** - 用于汇总结果的字段。接受: 集群"、"节点"、"名称空间"、"控制器类型"、"控制器"、"服务"、"pod"、"容器"、"标签:名称 "和 "Annotations:名称"。也接受以逗号分隔的列表进行多分类，如 `namespace,label:app`。
* **step** - 单个分配集的持续时间。如果未指定，则默认为窗口，这样整个窗口都会收到一个分配集。如果指定，如 `30m`、`2h`、`1d` 等，则会按时间倒序工作，以步长为单位查询，直到覆盖整个窗口。默认为 `window`。
* **resolution** - 在 Prometheus 查询中被用于为分辨率的持续时间。数值越小(即分辨率越高)，精确度越高，但性能越差(即查询时间越慢、内存占用越高)。数值越大(即分辨率越低)，性能越好，但对于短时间运行的工作负载来说，准确性会降低。默认值为 `1m`。
* Port"、"实体 "和 "映射 "键被用来将 OpenCost 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: cost
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one OpenCost object to a Port entity. Each value is a JQ query.
            identifier: .name
            title: .name
            blueprint: '"openCostResourceAllocation"'
            properties:
              cluster: .properties.cluster
              namespace: .properties.namespace
              startDate: .start
              endDate: .end
              cpuCoreHours: .cpuCoreHours
              cpuCost: .cpuCost
              cpuEfficiency: .cpuEfficiency
              gpuHours: .gpuHours
              gpuCost: .gpuCost
              networkCost: .networkCost
              loadBalancerCost: .loadBalancerCost
              pvCost: .pvCost
              ramBytes: .ramBytes
              ramCost: .ramCost
              ramEfficiency: .ramEfficiency
              sharedCost: .sharedCost
              externalCost: .externalCost
              totalCost: .totalCost
              totalEfficiency: .totalEfficiency
        # highlight-end
    - kind: cost # In this instance cost is mapped again with a different filter
      selector:
        query: '.name == "MyNodeName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): ::: 

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 OpenCost 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要使用 OpenCost 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在云成本 Provider 类别下选择 OpenCost。
5. 根据您的需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

### 费用

<details>
<summary>Cost blueprint</summary>

```json showLineNumbers
{
  "identifier": "openCostResourceAllocation",
  "description": "This blueprint represents an OpenCost resource allocation in our software catalog",
  "title": "OpenCost Resource Allocation",
  "icon": "Cluster",
  "schema": {
    "properties": {
      "cluster": {
        "type": "string",
        "title": "Cluster"
      },
      "namespace": {
        "type": "string",
        "title": "Namespace"
      },
      "startDate": {
        "title": "Start Date",
        "type": "string",
        "format": "date-time"
      },
      "endDate": {
        "title": "End Date",
        "type": "string",
        "format": "date-time"
      },
      "cpuCoreHours": {
        "title": "CPU Core Hours",
        "type": "number"
      },
      "cpuCost": {
        "title": "CPU Cost",
        "type": "number"
      },
      "cpuEfficiency": {
        "title": "CPU Efficiency",
        "type": "number"
      },
      "gpuHours": {
        "title": "GPU Hours",
        "type": "number"
      },
      "gpuCost": {
        "title": "GPU Cost",
        "type": "number"
      },
      "networkCost": {
        "title": "Network Cost",
        "type": "number"
      },
      "loadBalancerCost": {
        "title": "Load Balancer Cost",
        "type": "number"
      },
      "pvCost": {
        "title": "PV Cost",
        "type": "number"
      },
      "ramBytes": {
        "title": "RAM Bytes",
        "type": "number"
      },
      "ramCost": {
        "title": "RAM Cost",
        "type": "number"
      },
      "ramEfficiency": {
        "title": "RAM Efficiency",
        "type": "number"
      },
      "sharedCost": {
        "title": "Shared Cost",
        "type": "number"
      },
      "externalCost": {
        "title": "External Cost",
        "type": "number"
      },
      "totalCost": {
        "title": "Total Cost",
        "type": "number"
      },
      "totalEfficiency": {
        "title": "Total Efficiency",
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
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: cost
    selector:
      query: "true"
      window: "month"
      aggregate: "pod"
      step: "window"
      resolution: "1m"
    port:
      entity:
        mappings:
          blueprint: '"openCostResourceAllocation"'
          identifier: .name
          title: .name
          properties:
            cluster: .properties.cluster
            namespace: .properties.namespace
            startDate: .start
            endDate: .end
            cpuCoreHours: .cpuCoreHours
            cpuCost: .cpuCost
            cpuEfficiency: .cpuEfficiency
            gpuHours: .gpuHours
            gpuCost: .gpuCost
            networkCost: .networkCost
            loadBalancerCost: .loadBalancerCost
            pvCost: .pvCost
            ramBytes: .ramBytes
            ramCost: .ramCost
            ramEfficiency: .ramEfficiency
            sharedCost: .sharedCost
            externalCost: .externalCost
            totalCost: .totalCost
            totalEfficiency: .totalEfficiency
```

</details>

## 让我们来测试一下

本节包括 OpenCost 的响应数据示例。 此外，还包括根据上一节提供的 Ocean 配置从重新同步事件中创建的实体。

### 有效载荷

下面是 OpenCost 在 "namespace "层面上汇总的有效载荷结构示例: 

<details>
<summary> Cost response data</summary>

```json showLineNumbers
{
  "name": "ingress-nginx",
  "properties": {
    "cluster": "cluster-one",
    "node": "minikube",
    "container": "controller",
    "controller": "ingress-nginx-controller",
    "controllerKind": "deployment",
    "namespace": "ingress-nginx",
    "pod": "ingress-nginx-controller-7799c6795f-29n7j",
    "services": [
      "ingress-nginx-controller-admission",
      "ingress-nginx-controller"
    ],
    "labels": {
      "app_kubernetes_io_component": "controller",
      "app_kubernetes_io_instance": "ingress-nginx",
      "app_kubernetes_io_name": "ingress-nginx",
      "gcp_auth_skip_secret": "true",
      "kubernetes_io_metadata_name": "ingress-nginx",
      "pod_template_hash": "7799c6795f"
    },
    "namespaceLabels": {
      "app_kubernetes_io_instance": "ingress-nginx",
      "app_kubernetes_io_name": "ingress-nginx",
      "kubernetes_io_metadata_name": "ingress-nginx"
    }
  },
  "window": {
    "start": "2023-10-30T00:00:00Z",
    "end": "2023-10-31T00:00:00Z"
  },
  "start": "2023-10-30T09:05:00Z",
  "end": "2023-10-30T11:50:00Z",
  "minutes": 165,
  "cpuCores": 0.1,
  "cpuCoreRequestAverage": 0.1,
  "cpuCoreUsageAverage": 0,
  "cpuCoreHours": 0.275,
  "cpuCost": 0.00869,
  "cpuCostAdjustment": 0,
  "cpuEfficiency": 0,
  "gpuCount": 0,
  "gpuHours": 0,
  "gpuCost": 0,
  "gpuCostAdjustment": 0,
  "networkTransferBytes": 0,
  "networkReceiveBytes": 0,
  "networkCost": 0,
  "networkCrossZoneCost": 0,
  "networkCrossRegionCost": 0,
  "networkInternetCost": 0,
  "networkCostAdjustment": 0,
  "loadBalancerCost": 0,
  "loadBalancerCostAdjustment": 0,
  "pvBytes": 0,
  "pvByteHours": 0,
  "pvCost": 0,
  "pvs": "None",
  "pvCostAdjustment": 0,
  "ramBytes": 94371840,
  "ramByteRequestAverage": 94371840,
  "ramByteUsageAverage": 0,
  "ramByteHours": 259522560,
  "ramCost": 0.00102,
  "ramCostAdjustment": 0,
  "ramEfficiency": 0,
  "externalCost": 0,
  "sharedCost": 0,
  "totalCost": 0.00972,
  "totalEfficiency": 0,
  "lbAllocations": "None"
}
```

</details>

#### 映射结果

结合样本有效载荷和 Ocean 配置，可生成以下 Port 实体: 

<details>
<summary> Cost entity in Port</summary>

```json showLineNumbers
{
  "identifier": "ingress-nginx",
  "title": "ingress-nginx",
  "blueprint": "openCostResourceAllocation",
  "team": [],
  "properties": {
    "cluster": "cluster-one",
    "namespace": "ingress-nginx",
    "startDate": "2023-10-30T09:05:00.000Z",
    "endDate": "2023-10-30T11:50:00.000Z",
    "cpuCoreHours": 0.275,
    "cpuCost": 0.00869,
    "cpuEfficiency": 0,
    "gpuHours": 0,
    "gpuCost": 0,
    "networkCost": 0,
    "loadBalancerCost": 0,
    "pvCost": 0,
    "ramBytes": 94371840,
    "ramCost": 0.00102,
    "ramEfficiency": 0,
    "sharedCost": 0,
    "externalCost": 0,
    "totalCost": 0.00972,
    "totalEfficiency": 0
  },
  "relations": {},
  "createdAt": "2023-10-15T09:30:57.924Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-10-30T11:49:20.881Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>