---
sidebar_position: 6
---

# GitOps 被引用 Port CRDs

您可以使用 GitOps、[Port's K8s exporter](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md) 和[Port's Entity CRDs](/build-your-software-catalog/sync-data-to-catalog/kubernetes/port-crd.md) 将自定义实体导入 Port。

:::note 要全面了解如何被引用 GitOps 与 Port 的 CRD 映射实体，请务必熟悉以下内容: 

* * [Ports Kubernetes exporter](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md)
* [Ports Entity CRDs](/build-your-software-catalog/sync-data-to-catalog/kubernetes/port-crd.md)

:::

### 导出具有 GitOps 和 Kubernetes 用例的实体

* 将您的 k8s 集群作为**微服务**、**包**、**库**和其他软件目录资产的真实来源；
* 以 "仅推送 "的方式更新 Port，Port 与基础架构交互时无需提升权限；
* 允许开发人员通过更新其 Git 仓库中的 Kubernetes 配置清单文件来保持目录的最新状态；
* 创建一种标准化的方式来记录组织内的软件目录资产；
* 等等。

## 管理被引用 CRD 和 GitOps 定义的实体

Port 的 CRD 允许使用 Kubernetes 定义和映射任何类型的实体。您可以在[Port CRDs - mapping a microservice example](/build-your-software-catalog/sync-data-to-catalog/kubernetes/port-crd.md#example---mapping-a-microservice-using-port-crds) 中找到这样做的示例。映射实体可以使用任何持续部署(CD)解决方案来完成，例如 ArgoCD 或 FluxCD，方法是使用 CD 解决方案部署 Port 自定义资源，并使用 Port 的 k8s 输出程序将其定义映射到 Port。

为此，任何光盘解决方案都应遵循以下一般步骤: 

1. 导航至[Port's CRDs document](/build-your-software-catalog/sync-data-to-catalog/kubernetes/port-crd.md#deploying-ports-crds) 页面，了解如何部署 CRD。您可以将 CRD 配置清单放在 CD 源目录/应用程序中，将其作为 CD Pipelines 的一部分进行部署；
2. 使用 Port 的 CRD 定义一个 Kubernetes Port 实体配置清单，其中包含您希望映射到 Port 的数据模型和数据，然后使用您的 CD 解决方案将其部署到您的 kubernetes 集群；
3.  [Update](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md#updating-exporter-configuration) Port 的 k8s 输出程序资源映射，以映射您刚刚创建的 Port CRD。

被引用 Port 的 CRD 定义的实体将出现在 Port 环境中。