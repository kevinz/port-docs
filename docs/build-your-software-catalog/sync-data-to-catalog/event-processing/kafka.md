import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import DockerParameters from "./_kafka_one_time_docker_params.mdx"
import HelmParameters from "../templates/_ocean-advanced-parameters-helm.mdx"

# 卡夫卡

通过我们的 Kafka 集成，您可以根据您的映射和定义，将 Kafka "集群 "中的 "经纪人 "和 "专题 "导入 Port。

## 常见被用于情况

* 映射 Kafka 集群中的代理和主题。
* 按计划关注对象变更(创建/更新/删除)，并自动将变更应用到 Port 中的实体。
* 使用自助操作创建/删除 Kafka 对象。

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
| `integration.secrets.clusterConfMapping` | The Mapping of Kafka cluster names to Kafka client config  |  | ✅       |


<HelmParameters/>

<br/>
<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install kafka port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set port.baseUrl="https://api.getport.io"  \
    --set initializePortResources=true  \
    --set scheduledResyncInterval=60  \
    --set integration.identifier="my-kafka-integration"  \
    --set integration.type="kafka"  \
    --set integration.eventListener.type="POLLING"  \
    --set-json integration.secrets.clusterConfMapping='{"local": {"bootstrap.servers": "localhost:9092"}}'
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在你的 git 仓库的 `argocd/my-ocean-kafka-integration` 中创建一个内容为 `values.yaml` 的文件: 

:::note 请记住替换 `KAFKA_CLUSTER_CONFIG_MAPPING` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-kafka-integration
  type: kafka
  eventListener:
    type: POLLING
  secrets:
  // highlight-next-line
    clusterConfMapping: KAFKA_CLUSTER_CONFIG_MAPPING
```

<br/>

2.创建下面的 "my-ocean-kafka-integration.yaml "配置清单，安装 "my-ocean-kafka-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-kafka-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-kafka-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-kafka-integration/values.yaml
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
kubectl apply -f my-ocean-kafka-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">

  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Kafka integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项。

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters/>

<br/>

下面是 `kafka-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Kafka Exporter Workflow

# This workflow responsible for running Kafka exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Kafka Integration
        run: |
          # Set Docker image and run the container
          integration_type="kafka"
          version="latest"
          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__CLUSTER_CONF_MAPPING=${{ secrets.OCEAN__INTEGRATION__CONFIG__CLUSTER_CONF_MAPPING }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Kafka integration once and then exit, this is useful for **scheduled** ingestion of data.

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
        stage('Run Kafka Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__CLUSTER_CONF_MAPPING', variable: 'OCEAN__INTEGRATION__CONFIG__CLUSTER_CONF_MAPPING'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="kafka"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__CLUSTER_CONF_MAPPING=$OCEAN__INTEGRATION__CONFIG__CLUSTER_CONF_MAPPING \
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
<AzurePremise name="Kafka" />

<DockerParameters />

<br/>

下面是 `kafka-integration.yml` Pipelines 文件的示例: 

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
    integration_type="kafka"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
        -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
        -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
        -e OCEAN__INTEGRATION__CONFIG__CLUSTER_CONF_MAPPING=${OCEAN__INTEGRATION__CONFIG__CLUSTER_CONF_MAPPING} \
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

## 接收 Kafka 对象

Kafka 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 Kafka 获取 `集群`数据的过程: 

```yaml showLineNumbers
createMissingRelatedEntities: false
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
          blueprint: '"kafkaCluster"'
          properties:
            controllerId: .controller_id
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 Kafka 元数据对象中的现有字段和值进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了将从 Kafka 查询哪些资源，以及将在 Port 中创建哪些实体和属性。

:::tip  支持的资源 以下资源可被用来映射来自 Kafka 的数据，可以引用下面示例中出现的任何字段进行映射配置。

<details>
<summary>Cluster example</summary>

```json showLineNumbers
{
  "name": "local",
  "controller_id": "1"
}
```

</details>

<details>
<summary>Broker example</summary>

```json showLineNumbers
{
  "id": "1",
  "address": "localhost:9092/1",
  "cluster_name": "local",
  "config": {"key": "value", ...}
}
```

</details>

<details>
<summary>Topic example</summary>

```json showLineNumbers
{
  "name": "_consumer_offsets",
  "cluster_name": "local",
  "partitions": [
    {
      "id": 0,
      "leader": 2,
      "replicas": [2, 1, 3],
      "isrs": [3, 2, 1]
    }
  ],
  "config": {"key": "value", ...}
}
```

</details>

:::

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: cluster
      selector:
      ...
  ```


* 类型 "键是 Kafka 对象的说明符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: cluster
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: cluster
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
      # highlight-end
      port:
  ```


* Port"、"实体 "和 "映射 "键被用来将 Kafka 对象字段映射到Port实体。要创建多个同类映射，可以在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: cluster
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one Kafka cluster to a Port entity. Each value is a JQ query.
            identifier: .name
            title: .name
            blueprint: '"kafkaCluster"'
            properties:
              controllerId: .controller_id
        # highlight-end
    - kind: cluster # In this instance cluster is mapped again with a different filter
      selector:
        query: '.name == "MyClusterName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): ::: 

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 被用于 Kafka 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要使用 Kafka 进行引用的蓝图。
3. 从菜单中选择**摄取数据**选项。
4. 在事件处理 Provider 类别下选择 Kafka。
5. 根据需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

#### 集群

<details>
<summary>Cluster blueprint</summary>

```json showLineNumbers
{
  "identifier": "kafkaCluster",
  "title": "Cluster",
  "icon": "Kafka",
  "schema": {
    "properties": {
      "controllerId": {
        "title": "Controller ID",
        "type": "string"
      }
    }
  }
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
createMissingRelatedEntities: false
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
          blueprint: '"kafkaCluster"'
          properties:
            controllerId: .controller_id
```

</details>

### 经纪人

<details>
<summary>Broker blueprint</summary>

```json showLineNumbers
{
  "identifier": "kafkaBroker",
  "title": "Broker",
  "icon": "Kafka",
  "schema": {
    "properties": {
      "address": {
        "title": "Address",
        "type": "string"
      },
      "region": {
        "title": "Region",
        "type": "string"
      },
      "version": {
        "title": "Version",
        "type": "string"
      },
      "config": {
        "title": "Config",
        "type": "object"
      }
    }
  },
  "relations": {
    "cluster": {
      "target": "kafkaCluster",
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
createMissingRelatedEntities: false
deleteDependentEntities: true
resources:
  - kind: broker
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .cluster_name + "_" + (.id | tostring)
          title: .cluster_name + " " + (.id | tostring)
          blueprint: '"kafkaBroker"'
          properties:
            address: .address
            region: .config."broker.rack"
            version: .config."inter.broker.protocol.version"
            config: .config
          relations:
            cluster: .cluster_name
```

</details>

#### 主题

<details>
<summary>Topic blueprint</summary>

```json showLineNumbers
{
  "identifier": "kafkaTopic",
  "title": "Topic",
  "icon": "Kafka",
  "schema": {
    "properties": {
      "replicas": {
        "title": "Replicas",
        "type": "number"
      },
      "partitions": {
        "title": "Partitions",
        "type": "number"
      },
      "compaction": {
        "title": "Compaction",
        "type": "boolean"
      },
      "retention": {
        "title": "Retention",
        "type": "boolean"
      },
      "deleteRetentionTime": {
        "title": "Delete Retention Time",
        "type": "number"
      },
      "partitionsMetadata": {
        "title": "Partitions Metadata",
        "type": "array"
      },
      "config": {
        "title": "Config",
        "type": "object"
      }
    }
  },
  "relations": {
    "cluster": {
      "target": "kafkaCluster",
      "required": true,
      "many": false
    },
    "brokers": {
      "target": "kafkaBroker",
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
createMissingRelatedEntities: false
deleteDependentEntities: true
resources:
  - kind: topic
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .cluster_name + "_" + .name
          title: .cluster_name + " " + .name
          blueprint: '"kafkaTopic"'
          properties:
            replicas: .partitions[0].replicas | length
            partitions: .partitions | length
            compaction: .config."cleanup.policy" | contains("compact")
            retention: .config."cleanup.policy" | contains("delete")
            deleteRetentionTime: .config."delete.retention.ms"
            partitionsMetadata: .partitions
            config: .config
          relations:
            cluster: .cluster_name
            brokers: '[.cluster_name + "_" + (.partitions[].replicas[] | tostring)] | unique'
```

</details>