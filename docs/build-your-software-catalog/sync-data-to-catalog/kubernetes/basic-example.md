---

sidebar_position: 3
description: 基本示例

---

import Image from "@theme/IdealImage";
import SpecificEntityPage from "/static/img/integrations/k8s-exporter/DeploymentConfigAndPods.png"
import AuditLogPage from "/static/img/integrations/k8s-exporter/AuditLog.png"

# 基本示例

## 基本 Pod 和 ReplicaSet 解决方案

在下面的示例中，您将把 Kubernetes 的 "复制集 "和 "节点 "导出到 Port，您可以使用以下 Port 蓝图定义和导出器配置: 

* **部署配置** - 将代表 K8s 集群中的副本集；
* **部署的服务 pod** - 将代表来自 K8s 集群的 pod。

<details>
<summary> Deployment Config Blueprint </summary>

```json showLineNumbers
{
  "identifier": "deploymentConfig",
  "title": "Deployment Config",
  "icon": "Cluster",
  "schema": {
    "properties": {
      "newRelicUrl": {
        "type": "string",
        "format": "url",
        "title": "New Relic",
        "description": "Link to the new relic dashboard of the service",
        "default": "https://newrelic.com"
      },
      "sentryUrl": {
        "type": "string",
        "format": "url",
        "title": "Sentry URL",
        "description": "Link to the new sentry dashboard of the service",
        "default": "https://sentry.io/"
      },
      "prometheusUrl": {
        "type": "string",
        "format": "url",
        "title": "Prometheus URL",
        "default": "https://prometheus.io"
      },
      "locked": {
        "type": "boolean",
        "title": "Locked",
        "default": false,
        "description": "Are deployments currently allowed for this configuration",
        "icon": "Lock"
      },
      "creationTimestamp": {
        "type": "string",
        "title": "Creation Timestamp",
        "format": "date-time"
      },
      "annotations": {
        "type": "object",
        "title": "Annotations"
      },
      "status": {
        "type": "object",
        "title": "Status"
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
<summary> Deployed Service Pod Blueprint </summary>

```json showLineNumbers
{
  "identifier": "deployedServicePod",
  "title": "Deployed Service Pod",
  "icon": "Cluster",
  "schema": {
    "properties": {
      "startTime": {
        "type": "string",
        "title": "Start Time",
        "format": "date-time"
      },
      "phase": {
        "type": "string",
        "title": "Phase",
        "enum": ["Pending", "Running", "Succeeded", "Failed", "Unknown"],
        "enumColors": {
          "Pending": "yellow",
          "Running": "blue",
          "Succeeded": "green",
          "Failed": "red",
          "Unknown": "darkGray"
        }
      },
      "labels": {
        "type": "object",
        "title": "Labels"
      },
      "containers": {
        "title": "Containers",
        "type": "array"
      },
      "conditions": {
        "type": "array",
        "title": "Conditions"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "deploymentConfig": {
      "target": "deploymentConfig",
      "required": false,
      "many": false
    }
  }
}
```

</details>

<details>
<summary> Port K8s Exporter configuration </summary>

```yaml showLineNumbers
resources: # List of K8s resources to list, watch, and export to Port.
  - kind: apps/v1/replicasets # group/version/resource (G/V/R) format
    selector:
      query: .metadata.namespace | startswith("kube") | not # JQ boolean query. If evaluated to false - skip syncing the object.
    port:
      entity:
        mappings: # Mappings between one K8s object to one or many Port Entities. Each value is a JQ query.
          - identifier: .metadata.name
            title: .metadata.name
            blueprint: '"deploymentConfig"'
            properties:
              creationTimestamp: .metadata.creationTimestamp
              annotations: .metadata.annotations
              status: .status
  - kind: v1/pods
    selector:
      query: .metadata.namespace | startswith("kube") | not
    port:
      entity:
        mappings:
          - identifier: .metadata.name
            title: .metadata.name
            blueprint: '"deployedServicePod"'
            properties:
              startTime: .status.startTime
              phase: .status.phase
              labels: .metadata.labels
              containers: (.spec.containers | map({image, resources})) + .status.containerStatuses | group_by(.image) | map(add)
              conditions: .status.conditions
            relations:
              deploymentConfig: .metadata.ownerReferences[0].name
```

</details>

创建蓝图后，在[data sources](https://app.getport.io/dev-portal/data-sources) 页面打开集成，应用 Provider 提供的配置，然后点击 "保存并重新同步 "按钮。

完成！导出器将很快开始以 Port 实体的形式从 Kubernetes 集群创建和更新对象。

例如，您可以在单个 Port 实体页面中看到 "部署配置 "及其 "节点": 

<center>

<Image img={SpecificEntityPage} style={{ width: 1000 }} />

</center>

:::note  注意 Kubernetes 输出程序只被指示在 "部署配置 "实体中填写某些属性。 根据其性质，它将保持其他属性的值不变。

:::

您还可以查找相应的审计 logging，并注明 Kubernetes 输出程序为来源: 

<center>

<Image img={AuditLogPage} style={{ width: 1000 }} />

</center>

## 绘制完整的 k8s 生态系统图

要了解如何可视化一个完整的 k8s 集群，包括**节点**、**名称空间**、**集群角色**、**部署**、**节点**等，请查看我们的 Kubernetes 映射。[complete use case](./templates/full-kubernetes-exporter.md)
