import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import DockerParameters from "./_kubecost-docker-parameters.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'

# 库贝科斯特

通过我们的 Kubecost 集成，您可以根据您的映射和定义，将 Kubecost 实例中的 "kubesystem "和 "cloud "成本分配导入 Port。

## 常见被引用情况

* 在 Kubecost 中映射受监控的 Kubernetes 资源和云成本分配。

## 先决条件

<Prerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

被引用此安装选项意味着集成将能实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                         | Description                                                                                                   | Required |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------- | -------- |
| `port.clientId`                   | Your port client id                                                                                           | ✅       |
| `port.clientSecret`               | Your port client secret                                                                                       | ✅       |
| `port.baseUrl`                    | Your port base url, relevant only if not using the default port app                                           | ❌       |
| `integration.identifier`          | Change the identifier to describe your integration                                                            | ✅       |
| `integration.type`                | The integration type                                                                                          | ✅       |
| `integration.eventListener.type`  | The event listener type                                                                                       | ✅       |
| `integration.config.kubecostHost` | The Kubecost server URL                                                                                       | ✅       |
| `scheduledResyncInterval`         | The number of minutes between each resync                                                                     | ❌       |
| `initializePortResources`         | Default true, When set to true the integration will create default blueprints and the port App config Mapping | ❌       |


<br/>

<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-kubecost-integration port-labs/port-ocean \
  --set port.clientId="CLIENT_ID"  \
  --set port.clientSecret="CLIENT_SECRET"  \
  --set initializePortResources=true  \
  --set scheduledResyncInterval=60 \
  --set integration.identifier="my-kubecost-integration"  \
  --set integration.type="kubecost"  \
  --set integration.eventListener.type="POLLING"  \
  --set integration.config.kubecostHost="https://kubecostInstance:9090"
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在 git 仓库的`argocd/my-ocean-kubecost-integration`中创建一个`values.yaml`文件，内容如下: 

:::note 记住要替换 `KUBECOST_HOST` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-kubecost-integration
  type: kubecost
  eventListener:
    type: POLLING
  config:
  // highlight-next-line
    kubecostHost: KUBECOST_HOST
```

<br/>

2.创建以下 "my-ocean-kubecost-integration.yaml "配置清单，安装 "my-ocean-kubecost-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-kubecost-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-kubecost-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-kubecost-integration/values.yaml
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
kubectl apply -f my-ocean-kubecost-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Kubecost integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成能实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />

<br/>

下面是 `kubecost-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Kubecost Exporter Workflow

# This workflow responsible for running Kubecost exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Kubecost Integration
        run: |
          # Set Docker image and run the container
          integration_type="kubecost"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__KUBECOST_HOST=${{ secrets.OCEAN__INTEGRATION__CONFIG__KUBECOST_HOST }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Kubecost integration once and then exit, this is useful for **scheduled** ingestion of data.

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
        stage('Run Kubecost Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__KUBECOST_HOST', variable: 'OCEAN__INTEGRATION__CONFIG__KUBECOST_HOST'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="kubecost"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__KUBECOST_HOST=$OCEAN__INTEGRATION__CONFIG__KUBECOST_HOST \
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

下面是 `kubecost-integration.yml` Pipelines 文件的示例: 

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
    integration_type="kubecost"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
        -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
        -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
        -e OCEAN__INTEGRATION__CONFIG__KUBECOST_HOST=${OCEAN__INTEGRATION__CONFIG__KUBECOST_HOST} \
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

## 接收 Kubecost 对象

Kubecost 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是一个配置示例片段，演示了从 Kubecost 获取成本分配数据的过程: 

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: kubesystem
    selector:
      query: "true"
      window: "month"
    port:
      entity:
        mappings:
          blueprint: '"kubecostResourceAllocation"'
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
            pvBytes: .pvBytes
            ramBytes: .ramBytes
            ramCost: .ramCost
            ramEfficiency: .ramEfficiency
            sharedCost: .sharedCost
            externalCost: .externalCost
            totalCost: .totalCost
            totalEfficiency: .totalEfficiency
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 Kubecost API 事件的现有字段和 Values 进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了将从 Kubecost 查询哪些资源，以及将在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被引用来映射来自 Kubecost 的数据，可以引用下面链接的 API 响应中出现的任何字段来进行映射配置。

* * [`kubesystem`](https://docs.kubecost.com/apis/apis-overview/api-allocation#allocation-schema)
* [`cloud`](https://docs.kubecost.com/apis/apis-overview/cloud-cost-api#cloud-cost-aggregate-api)

:::

:::note 按照以下步骤在 Kubecost 实例上成功配置云计费 API 后，您就可以查看 "云 "成本数据了[documentation](https://docs.kubecost.com/install-and-configure/install/cloud-integration)

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: kubesystem
      selector:
      ...
  ```


* 类型 "键是 Kubecost 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: kubesystem
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: kubesystem
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
        window: "month"
        aggregate: "pod"
        idle: true
      # highlight-end
      port:
  ```


* **window** - 要查询的时间长度。接受: "today"、"week"、"month"、"yesterday"、"lastweek"、"lastmonth "等单词；"30m"、"12h"、"7d "等持续时间；"2021-01-02T15:04:05Z,2021-02-02T15:04:05Z "等 RFC3339 日期对；"1578002645,1580681045 "等 Unix 时间戳。
* **aggregate** - 用于汇总结果的字段。接受: 集群"、"节点"、"名称空间"、"控制器类型"、"控制器"、"服务"、"pod"、"容器"、"标签:名称 "和 "Annotations:名称"。也接受以逗号分隔的列表进行多分类，如 `namespace,label:app`。
* **step** - 单个分配集的持续时间。如果未指定，则默认为窗口，这样整个窗口都会收到一个分配集。如果指定，如 `30m`、`2h`、`1d` 等，则会按时间倒序工作，以步长为单位查询，直到覆盖整个窗口。默认为 `window`。
* **accumulate** - 如果为 "true"，则将整个范围内的集合汇总为一个集合。默认值为 `false`。
* **idle** - 如果为 "true"，将闲置成本(即未分配资产的成本)作为其自身的分配。默认值为`true`。
* **external** - 如果为 "true"，则在每次分配中包含外部或集群外成本。默认为 `false`。
* **filterClusters** - 以逗号分隔的要匹配的集群列表；例如，`cluster-one,cluster-two`将只返回这两个集群的结果。
* **filterNodes** - 以逗号分隔的节点列表；例如，"node-one,node-two "将只返回这两个节点的结果。
* **filterNamespaces** - 以逗号分隔的名称空间匹配列表；例如，"namespace-one,namespace-two "将只返回这两个名称空间的结果。
* **filterControllerKinds** - 以逗号分隔的控制器类型匹配列表；例如，"deployment"，任务将只返回这两种控制器类型的结果。
* **filterControllers** - 以逗号分隔的控制器匹配列表；例如，"deployment-one,statefulset-two "将只返回来自这两个控制器的结果。
* **filterPods** - 以逗号分隔的 pod 列表，例如，`pod-one,pod-two` 只返回这两个 pod 的结果。
* **filterAnnotations** - 以逗号分隔的注释匹配列表；例如，`name:annotation-one,name:annotation-two` 将返回这两个注释键值对中任何一个的结果。
* **filterControllerKinds** - 以逗号分隔的控制器类型匹配列表；例如，"deployment"，任务将仅返回包含这两种控制器类型的结果。
* **filterLabels** - 以逗号分隔的要匹配的 Annotations 列表；例如，"app:cost-analyzer"、"app:prometheus "将返回包含这两个标签键值对中任何一个的结果。
* **filterServices** - 以逗号分隔的要匹配的服务列表；例如，`frontend-one,frontend-two` 将返回包含这两个服务中任何一个的结果。
* **shareIdle** - 如果为 "true"，闲置成本将按比例分配给每个资源的所有非闲置分配。也就是说，闲置 CPU 成本将根据 CPU 总成本的百分比与每个非闲置分配的 CPU 成本共享。默认为 `false`。
* **splitIdle** - 如果为 "true"，且 shareIdle == false，则闲置分配将按集群或节点创建，而不是汇总为一个闲置分配。默认为 false。
* **idleByNode** - 如果为 "true"，空闲分配将按每个节点创建。这将导致共享时产生不同的 Values，拆分时产生更多空闲分配。默认为 false。
* 以及任何可在[Kubecost allocation API](https://docs.kubecost.com/apis/apis-overview/api-allocation#allocation-api) 和[Kubecost Cloud API](https://docs.kubecost.com/apis/apis-overview/cloud-cost-api#cloud-cost-aggregate-api)
* Port"、"实体 "和 "映射 "键被引用，用于将 Kubecost 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: kubesystem
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one Kubecost object to a Port entity. Each value is a JQ query.
            identifier: .name
            title: .name
            blueprint: '"KubecostResourceAllocation"'
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
    - kind: kubesystem # In this instance cost is mapped again with a different filter
      selector:
        query: '.name == "MyNodeName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): ::: 

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 被引用 Kubecost 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要使用 Kubecost 进行引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在云成本 Provider 类别下选择 Kubecost。
5. 根据您的需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

#### 费用分配

<details>
<summary>Cost allocation blueprint</summary>

```json showLineNumbers
{
  "identifier": "kubecostResourceAllocation",
  "description": "This blueprint represents an Kubecost resource allocation in our software catalog",
  "title": "Kubecost Resource Allocation",
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
      "pvBytes": {
        "title": "PV Bytes",
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
  - kind: kubesystem
    selector:
      query: "true"
    port:
      entity:
        mappings:
          blueprint: '"kubecostResourceAllocation"'
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
            pvBytes: .pvBytes
            ramBytes: .ramBytes
            ramCost: .ramCost
            ramEfficiency: .ramEfficiency
            sharedCost: .sharedCost
            externalCost: .externalCost
            totalCost: .totalCost
            totalEfficiency: .totalEfficiency
```

</details>

#### 云计算成本

<details>
<summary> Cloud cost blueprint</summary>

```json showlineNumbers
{
  "identifier": "kubecostCloudAllocation",
  "description": "This blueprint represents an Kubecost cloud resource allocation in our software catalog",
  "title": "Kubecost Cloud Allocation",
  "icon": "Cluster",
  "schema": {
    "properties": {
      "provider": {
        "type": "string",
        "title": "Provider"
      },
      "accountID": {
        "type": "string",
        "title": "Account ID"
      },
      "invoiceEntityID": {
        "type": "string",
        "title": "Invoice Entity ID"
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
      "listCost": {
        "title": "List Cost Value",
        "type": "number"
      },
      "listCostPercent": {
        "title": "List Cost Percent",
        "type": "number"
      },
      "netCost": {
        "title": "Net Cost Value",
        "type": "number"
      },
      "netCostPercent": {
        "title": "Net Cost Percent",
        "type": "number"
      },
      "amortizedNetCost": {
        "title": "Amortized Net Cost",
        "type": "number"
      },
      "amortizedNetCostPercent": {
        "title": "Amortized Net Cost Percent",
        "type": "number"
      },
      "invoicedCost": {
        "title": "Invoice Cost",
        "type": "number"
      },
      "invoicedCostPercent": {
        "title": "Invoice Cost Percent",
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
  - kind: cloud
    selector:
      query: "true"
    port:
      entity:
        mappings:
          blueprint: '"kubecostCloudAllocation"'
          identifier: .properties.service
          title: .properties.service
          properties:
            provider: .properties.provider
            accountID: .properties.accountID
            invoiceEntityID: .properties.invoiceEntityID
            startDate: .window.start
            endDate: .window.end
            listCost: .listCost.cost
            listCostPercent: .listCost.kubernetesPercent
            netCost: .netCost.cost
            netCostPercent: .netCost.kubernetesPercent
            amortizedNetCost: .amortizedNetCost.cost
            amortizedNetCostPercent: .amortizedNetCost.kubernetesPercent
            invoicedCost: .invoicedCost.cost
            invoicedCostPercent: .invoicedCost.kubernetesPercent
```

</details>

## 让我们来测试一下

本节包括来自 Kubecost 的响应数据示例。 此外，还包括根据上一节提供的 Ocean 配置从重新同步事件中创建的实体。

### 有效载荷

下面是 Kubecost 提供的有效载荷结构示例: 

<details>
<summary> Cost response data</summary>

```json showLineNumbers
{
  "name": "argocd",
  "properties": {
    "cluster": "cluster-one",
    "node": "gke-my-regional-cluster-default-pool-e8093bfa-0bjg",
    "namespace": "argocd",
    "providerID": "gke-my-regional-cluster-default-pool-e8093bfa-0bjg",
    "namespaceLabels": {
      "kubernetes_io_metadata_name": "argocd"
    }
  },
  "window": {
    "start": "2023-10-30T00:00:00Z",
    "end": "2023-10-30T01:00:00Z"
  },
  "start": "2023-10-30T00:00:00Z",
  "end": "2023-10-30T01:00:00Z",
  "minutes": 60,
  "cpuCores": 0.00515,
  "cpuCoreRequestAverage": 0,
  "cpuCoreUsageAverage": 0.00514,
  "cpuCoreHours": 0.00515,
  "cpuCost": 0.00012,
  "cpuCostAdjustment": 0,
  "cpuEfficiency": 1,
  "gpuCount": 0,
  "gpuHours": 0,
  "gpuCost": 0,
  "gpuCostAdjustment": 0,
  "networkTransferBytes": 2100541.53,
  "networkReceiveBytes": 2077024.88318,
  "networkCost": 0,
  "networkCrossZoneCost": 0,
  "networkCrossRegionCost": 0,
  "networkInternetCost": 0,
  "networkCostAdjustment": 0,
  "loadBalancerCost": 0.02708,
  "loadBalancerCostAdjustment": 0,
  "pvBytes": 0,
  "pvByteHours": 0,
  "pvCost": 0,
  "pvs": "None",
  "pvCostAdjustment": 0,
  "ramBytes": 135396181.33333,
  "ramByteRequestAverage": 0,
  "ramByteUsageAverage": 135394433.70477,
  "ramByteHours": 135396181.33333,
  "ramCost": 0.00041,
  "ramCostAdjustment": 0,
  "ramEfficiency": 1,
  "externalCost": 0,
  "sharedCost": 0,
  "totalCost": 0.02761,
  "totalEfficiency": 1,
  "proportionalAssetResourceCosts": {},
  "lbAllocations": {
    "cluster-one/argocd/argocd-server": {
      "service": "argocd/argocd-server",
      "cost": 0.027083333333333334,
      "private": false,
      "ip": ""
    }
  },
  "sharedCostBreakdown": {}
}
```

</details>

#### 映射结果

结合样本有效载荷和 Ocean 配置，可生成以下 Port 实体: 

<details>
<summary> Cost entity in Port</summary>

```json showLineNumbers
{
  "identifier": "argocd",
  "title": "argocd",
  "icon": null,
  "blueprint": "kubecostResourceAllocation",
  "team": [],
  "properties": {
    "cluster": "cluster-one",
    "namespace": "argocd",
    "startDate": "2023-10-30T04:00:00.000Z",
    "endDate": "2023-10-30T05:00:00.000Z",
    "cpuCoreHours": 0.0051,
    "cpuCost": 0.00012,
    "cpuEfficiency": 1,
    "gpuHours": 0,
    "gpuCost": 0,
    "networkCost": 0,
    "loadBalancerCost": 0.02708,
    "pvCost": 0,
    "pvBytes": 0,
    "ramBytes": 135396181.33333,
    "ramCost": 0.00041,
    "ramEfficiency": 1,
    "sharedCost": 0,
    "externalCost": 0,
    "totalCost": 0.02761,
    "totalEfficiency": 1
  },
  "relations": {},
  "createdAt": "2023-10-30T13:25:42.717Z",
  "createdBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW",
  "updatedAt": "2023-10-30T13:28:37.379Z",
  "updatedBy": "hBx3VFZjqgLPEoQLp7POx5XaoB0cgsxW"
}
```

</details>