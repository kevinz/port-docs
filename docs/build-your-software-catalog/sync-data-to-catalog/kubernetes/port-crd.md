---

sidebar_position: 7

---

# Port Entity CRD

[Port's K8s exporter](./kubernetes.md) 允许从您的 Kubernetes 集群中的任何资源导出数据，包括[Custom Resource Definitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)(CRDs) 。为了利用 Port 的 K8s 导出器的灵活性，Port 提供了额外的 CRDs，使得在您的软件目录中使用 K8s 资源定义作为[entities](/build-your-software-catalog/sync-data-to-catalog/sync-data-to-catalog.md#creating-entities) 的来源成为可能。

:::tip Port 提供的所有 CRD 均可查阅[here.](https://github.com/port-labs/port-crds)

:::

# Port CRDs

Port 实体可以代表基础架构中的任何类型数据，从节点、pod 到非 Kubernetes 相关实体(如存储库或微服务)。 为实现这一抽象层次，提供了 2 个 CRD: 

## 名称空间作用域实体 CRD - `getport.io/v1/Entity`

<details>
  <summary>Entity CRD</summary>

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: entities.getport.io
spec:
  group: getport.io
  versions:
  - name: v1
    served: true
    storage: true
    additionalPrinterColumns:
      - name: Blueprint ID
        type: string
        jsonPath: .spec.blueprint
      - name: Entity ID
        type: string
        jsonPath: .spec.identifier
      - name: Properties
        type: string
        jsonPath: .spec.properties
      - name: Relations
        type: string
        jsonPath: .spec.relations
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              blueprint:
                type: string
              identifier:
                type: string
              properties:
                type: object
                x-kubernetes-preserve-unknown-fields: true
              relations:
                type: object
                x-kubernetes-preserve-unknown-fields: true
            required:
              - blueprint
              - identifier
  scope: Namespaced
  names:
    plural: entities
    singular: entity
    kind: Entity
    shortNames:
    - ent
```

</details>

## 集群作用域实体 CRD - `getport.io/v1/ClusterEntity`

<details>
  <summary>Cluster Entity CRD</summary>

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: clusterentities.getport.io
spec:
  group: getport.io
  versions:
  - name: v1
    served: true
    storage: true
    additionalPrinterColumns:
      - name: Blueprint ID
        type: string
        jsonPath: .spec.blueprint
      - name: Entity ID
        type: string
        jsonPath: .spec.identifier
      - name: Properties
        type: string
        jsonPath: .spec.properties
      - name: Relations
        type: string
        jsonPath: .spec.relations
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              blueprint:
                type: string
              identifier:
                type: string
              properties:
                type: object
                x-kubernetes-preserve-unknown-fields: true
              relations:
                type: object
                x-kubernetes-preserve-unknown-fields: true
            required:
              - blueprint
              - identifier
  scope: Cluster
  names:
    plural: clusterentities
    singular: clusterentity
    kind: ClusterEntity
    shortNames:
    - cent
```

</details>

## Port CRDs 结构

Port CRD 提供四个关键属性: 

* 蓝图 ID ` **必填**: [blueprint](/build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md#what-is-a-blueprint) 您希望映射的实体的标识符(字符串) ；
* 实体 ID ` **必填**: [entity](/build-your-software-catalog/sync-data-to-catalog/sync-data-to-catalog.md#creating-entities) 您希望映射的实体的标识符(字符串) ；
* `Properties` **可选**: [properties](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/properties.md) 字段(对象) ，其中包含要映射的实体的属性数据；
* 关系` **可选**: [relations](/build-your-software-catalog/define-your-data-model/relate-blueprints/relate-blueprints.md) 字段(对象) 保存要映射的实体的关系数据。

<details>
  <summary>CRD examples</summary>

```yaml showLineNumbers
# Namespaced Port Entity CRD example
apiVersion: getport.io/v1
kind: Entity
metadata:
  name: example-entity-resource
  namespace: example-namespace
spec:
  blueprint: blueprint-identifier
  identifier: entity-identifier
  properties:
    myStringProp: string
    myArrayProp:
      - string_1
      - string_2
      - string_3
    myUrlProp: https://test-url.com
  relations:
    mySingleRelation: relation-target-id
    myManyRelations:
      - relation-target-id-1
      - relation-target-id-2

# Namespaced Port Cluster Entity CRD example
apiVersion: getport.io/v1
kind: ClusterEntity
metadata:
  name: example-cluster-entity-resource
spec:
  blueprint: blueprint-identifier
  identifier: entity-identifier
  properties:
    myStringProp: string
    myArrayProp:
      - string_1
      - string_2
      - string_3
    myUrlProp: https://test-url.com
  relations:
    mySingleRelation: relation-target-id
    myManyRelation:
      - relation-target-id-1
      - relation-target-id-2
```

</details>

## 部署 Port 的 CRDs

要在 K8s 集群中部署 Port 的 CRD，请运行以下命令: 

```bash showLineNumbers
# Run this to install Port's namespace-scoped CRD
kubectl apply -f https://raw.githubusercontent.com/port-labs/port-crds/main/port-entity-crd-namespace.yaml

# Run this to install Port's cluster-scoped CRD
kubectl apply -f https://raw.githubusercontent.com/port-labs/port-crds/main/port-entity-crd-cluster.yaml
```

## 导出 Port 的自定义资源

要使用 Port 的 k8s 导出器导出 Port 实体 CRD，您需要在导出器配置中添加一个新资源。 该映射配置将与您在软件目录中定义的蓝图数据模型相匹配。

要学习如何使用 Port CRD 来满足您的需求，您将按照一个示例来操作。 它将让您大致了解如何映射您想要的任何数据。

#### 示例 - 使用 Port CRD 映射微服务

本例的目标是使用 Port 的 CRD 和 Port 的 k8s 输出程序映射一个微服务。 在本例中，您将把一个微服务映射为 Port 实体。

:::note  开始之前的先决条件

* 准备好您的[Port credentials](/build-your-software-catalog/sync-data-to-catalog/api/api.md#find-your-port-credentials) ；
* 熟悉[Port's K8s exporter](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md) 和配置；
* 确保使用 `kubectl` 连接到 k8s 集群。

:::

1. **部署 Port CRD** - 按照[deployment step](/build-your-software-catalog/sync-data-to-catalog/kubernetes/port-crd.md#deploying-ports-crds) 部署 Port CRD。您只需要集群作用域的实体 CRD。
2. **创建蓝图** - 首先要定义蓝图，它将代表软件目录中的微服务。
在 Port 环境中创建以下蓝图: 

```json showLineNumbers
{
  "identifier": "microservice",
  "title": "Microservice",
  "icon": "Microservice",
  "description": "This blueprint represents a microservice",
  "schema": {
    "properties": {
      "description": {
        "title": "Description",
        "description": "A short description for this microservice",
        "type": "string"
      },
      "onCall": {
        "title": "On-Call",
        "description": "Who the on-call for this microservice is",
        "type": "string"
      },
      "slackChannel": {
        "title": "Slack Channel",
        "description": "The URL of this microservice's Slack channel",
        "format": "url",
        "type": "string"
      },
      "datadogUrl": {
        "title": "Datadog URL",
        "description": "The URL of this microservice's DataDog dashboard",
        "format": "url",
        "type": "string"
      },
      "language": {
        "type": "string",
        "description": "The language this microservice is written in",
        "title": "Language"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

3. **在集群中创建 Port 实体自定义资源** - 使用蓝图中定义的方案，创建一个将代表微服务的实体 CR: 
    1.创建以下文件作为 `port-entity.yaml`: 

```yaml showLineNumbers
# Namespaces Port Entity CRD example
apiVersion: getport.io/v1
kind: ClusterEntity
metadata:
  name: my-microservice-entity
spec:
  blueprint: microservice
  identifier: my-awesome-microservice
  properties:
    description: This entity represents my awesome microservice
    onCall: some@email.com
    slackChannel: https://domain.slack.com/archives/12345678
    datadogUrl: https://www.datadoghq.com/
    language: typescript
```

4.将 CRD 配置清单应用到集群: 

```bash showLineNumbers
kubectl apply -f port-entity.yaml
```

4. **为 k8s 输出程序创建映射配置** - 创建(或添加到现有的)以下输出程序配置，以便使用 Port 的 k8s 输出程序映射此 CRD: 
    1.在您的 Port 环境中打开[data sources](https://app.getport.io/dev-portal/data-sources) 页面，点击您希望添加映射的集成；
    2.将以下映射配置添加到导出器配置中: 
    :::tip 添加映射配置
    如果您已经有了出口程序配置，可将以下映射添加到现有配置中，方法是将 "resources "键后面的行添加到现有配置中。
    :::


   ```yaml showLineNumbers
   resources:
     - kind: getport.io/v1/ClusterEntities # Map entities of type 'Port Cluster Entities'
       selector:
         query: .spec.blueprint == "microservice" # This will make sure to only query ClusterEntites were .spec.blueprint == 'microservice'
       port:
         entity:
           mappings:
             - identifier: .spec.identifier
               title: .spec.identifier
               blueprint: .spec.blueprint
               properties:
                 description: .spec.properties.description
                 onCall: .spec.properties.onCall
                 slackChannel: .spec.properties.slackChannel
                 datadogUrl: .spec.properties.datadogUrl
                 language: .spec.properties.language
   ```


3.单击 "保存和重新同步 "按钮，将配置应用到集成中。

现在您可以在 Port 环境中看到新导出的实体。