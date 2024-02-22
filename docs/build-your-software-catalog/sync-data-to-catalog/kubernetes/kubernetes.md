---
sidebar_position: 1
---

import Image from "@theme/IdealImage";
import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem"
import KubernetesIllustration from "/static/img/build-your-software-catalog/sync-data-to-catalog/kubernetes/k8s-exporter-illustration.png";
import KubernetesEtl from "/static/img/build-your-software-catalog/sync-data-to-catalog/kubernetes/k8s-etl.png";
import FindCredentials from "/docs/build-your-software-catalog/sync-data-to-catalog/api/_template_docs/_find_credentials_collapsed.mdx";

# Kubernetes

我们与 Kubernetes 的集成可根据您的定义直接查询您的 Kubernetes 集群。 通过使用我们的 Kubernetes 集成，您可以以透明、高效和精确的方式直接从您的 k8s 集群向 Port 中引用实时数据，从而确保只有您需要的信息才会出现在软件目录中，并保持更新。

我们与 Provider 的集成提供了实时事件处理功能，可在 Port 内准确**实时**呈现您的 k8s 集群。

<center>

<Image img={KubernetesIllustration} style={{ width: 700 }} />

</center>

:::tip Port 的 Kubernetes 输出程序已开源，请查看源代码[**here**](https://github.com/port-labs/port-k8s-exporter)

:::

## 💡 Kubernetes 输出程序常见用例

例如，我们的 Kubernetes 导出器可以轻松地将集群中的实时数据直接填入软件目录: 

* 映射集群中的所有资源，包括**namespace**、**pods**、**replica sets**、**cluster nodes**、**deployments**和其他集群对象；
* 从集群中获取实时元数据，如_复制数量_、_部署健康状况_、_节点健康状况_等；
* 使用关系在 Port 内创建完整、易懂的 k8s 集群地图；
* 从 ArgoCD、Istio 等常见 CRD 映射您的 Kubernetes 资源；
* 等等。

## 工作原理

Port 的 Kubernetes 输出程序允许您将 K8s API 支持的所有数据显示正在运行的服务、环境等。 通过开源的 Kubernetes 输出程序，您可以对来自 K8s 的数据执行提取、转换、加载(ETL)，将其转换为所需的软件目录数据模型。

导出器使用安装在集群上的 Helm chart 进行部署，一旦设置完成，它就会持续同步更改，这意味着所有更改、删除或添加都会准确、自动地反映在 Port 中。

helm chart 使用的是存储在门户内集成中的 YAML 配置。 该配置描述了负责将数据加载到开发者门户的 ETL 流程。 该方法反映了一个黄金分割点，即过于主观的 k8s 可视化可能不适合所有人，而过于宽泛的方法可能会给开发者门户引入不必要的复杂性。

下面是集成配置中的一个示例片段，演示了从集群获取 `ReplicaSet` 数据并将其导入软件目录的 ETL 流程: 

<center>

<Image img={KubernetesEtl} style={{ width: 700 }} />

</center>

导出器利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 Kubernetes 对象中的现有字段和值进行选择、修改、连接、转换和其他操作。

###出口商 JQ 配置

导出器配置是您指定要从 k8s 集群中查询的确切资源的方式，也是您指定要从集群中填充数据的实体和属性的方式。

下面是一个配置块示例: 

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
```

### 输出端配置结构

* 配置 YAML 的根键是 `resources` 键: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: apps/v1/replicasets
      selector:
      ...
  ```


* k8s API 或 CRD 按照组/版本/资源(G/V/R)格式为对象指定了 "类型 "键: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: apps/v1/replicasets
        selector:
        ...
  ```


:::提示 要列出、观察和导出可用的 Kubernetes 资源，请访问[**here**](https://kubernetes.io/docs/reference/kubernetes-api/):::: 

* 通过 "选择器 "和 "查询 "键，你可以准确地筛选出指定 "类型 "中的哪些对象将被录入软件目录


  ```yaml showLineNumbers
  resources:
    - kind: apps/v1/replicasets
      # highlight-start
      selector:
        query: .metadata.namespace | startswith("kube") | not # JQ boolean query. If evaluated to false - skip syncing the object.
      # highlight-end
      port:
  ```


一些被用于的示例: 

* 要同步指定 "类型 "的所有对象: 请勿指定 "选择器 "和 "查询 "键；
* 要同步指定`种类`中与 Kubernetes 内部系统无关的所有对象，请使用: 


    ```yaml showLineNumbers
    query: .metadata.namespace | startswith("kube") | not
    ```


* 要同步指定 "类型 "中以 "生产 "开头的所有对象，请使用


    ```yaml showLineNumbers
    query: .metadata.namespace | startswith("production")
    ```


* 等。
* Port"、"实体 "和 "映射 "键打开了用于将 Kubernetes 对象字段映射到Port实体的部分。[entity](/build-your-software-catalog/sync-data-to-catalog/sync-data-to-catalog.md#entity-json-structure)


  ```yaml showLineNumbers
  resources:
    - kind: apps/v1/replicasets
      selector:
        query: .metadata.namespace | startswith("kube") | not
      # highlight-start
      port:
        entity:
          mappings: # Mappings between one K8s object to one or many Port Entities. Each value is a JQ query.
            - identifier: .metadata.name
              title: .metadata.name
              blueprint: '"myBlueprint"'
              properties:
                creationTimestamp: .metadata.creationTimestamp
                annotations: .metadata.annotations
                status: .status
              relations:
                myRelation: .metadata.namespace
      # highlight-end
  ```


## 先决条件

* Port 的 Kubernetes 输出程序是使用[Helm](https://helm.sh) 安装的，因此必须安装 Helm 才能使用输出程序的图表。有关安装说明，请参阅 Helm 的[documentation](https://helm.sh/docs) ；
* 安装 Kubernetes 输出程序需要[Port credentials](/build-your-software-catalog/sync-data-to-catalog/api/api.md#find-your-port-credentials) 。

:::tip 

<FindCredentials />

:::

:::info 出口商 helm chart 可见[here](https://github.com/port-labs/helm-charts/tree/main/charts/port-k8s-exporter)

:::

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="helm" label="Helm" default>

1. 使用以下命令添加 Port 的 Helm 软件源: 


   ```bash showLineNumbers
   helm repo add port-labs https://port-labs.github.io/helm-charts
   ```


:::tip 如果您之前已经添加了 Port 的 Helm repo，请运行 `helm repo update` 来获取最新版本的图表。 然后您可以运行 `helm search repo port-labs` 来查看图表。 
::: 

2.运行以下命令，在 Kubernetes 集群上安装导出器服务: 


   ```bash showLineNumbers
    helm upgrade --install my-port-k8s-exporter port-labs/port-k8s-exporter \
        --create-namespace --namespace port-k8s-exporter \
        --set secret.secrets.portClientId=YOUR_PORT_CLIENT_ID --set secret.secrets.portClientSecret=YOUR_PORT_CLIENT_SECRET \
        --set stateKey="k8s-exporter"  \
        --set eventListenerType="POLLING"  \
        --set "extraEnv[0].name"="CLUSTER_NAME" \
        --set "extraEnv[0].value"=YOUR_PORT_CLUSTER_NAME
    ```

</TabItem>

<TabItem value="argo" label="ArgoCD">

1. 通过创建以下 `my-port-k8s-exporter.yaml` 配置清单，安装 `my-port-k8s-exporter` ArgoCD 应用程序: 
 :::note
 记得替换 `LATEST_HELM_RELEASE``YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。
    您可以在我们的[Releases](https://github.com/port-labs/helm-charts/releases?q=port-k8s-exporter&amp;expanded=true) 页面找到最新版本的 `port-k8s-exporter` 图表。多种来源的 ArgoCD 文档可以在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 找到。
 :::
     <details>
      <summary>ArgoCD Application</summary>


    ```yaml showLineNumbers
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: my-port-k8s-exporter
      namespace: argocd
    spec:
      destination:
        namespace: my-port-k8s-exporter
        server: https://kubernetes.default.svc
      project: default
      sources:
      - repoURL: 'https://port-labs.github.io/helm-charts/'
        chart: port-k8s-exporter
        // highlight-next-line
        targetRevision: LATEST_HELM_RELEASE
        helm:
          valueFiles:
            - $values/argocd/my-port-k8s-exporter/values.yaml
          parameters:
            - name: secret.secrets.portClientId
              // highlight-next-line
              value: YOUR_PORT_CLIENT_ID
            - name: secret.secrets.portClientSecret
              // highlight-next-line
              value: YOUR_PORT_CLIENT_SECRET
            - name: stateKey
              // highlight-next-line
              value: YOUR_CLUSTER_NAME
            - name: extraEnv[0].name
              value: CLUSTER_NAME
            - name: extraEnv[0].value
              // highlight-next-line
              value: YOUR_CLUSTER_NAME
      // highlight-next-line
      - repoURL: YOUR_GIT_REPO_URL
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

2.使用 `kubectl` 配置应用程序清单: 

```bash
kubectl apply -f my-port-k8s-exporter.yaml
```

</TabItem>
</Tabs>

:::info 默认情况下，输出程序会尝试启动预定义的蓝图和资源映射。

:::

完成！导出器将很快开始以 Port 实体的形式从 Kubernetes 集群创建和更新对象。

### 更新出口商配置

要**更新**导出器资源映射，请在 Port 中打开[data sources](https://app.getport.io/dev-portal/data-sources) 页面，点击您的 Kubernetes 集成。 然后编辑导出器配置，点击 "保存并重新同步 "按钮。

## 示例

有关实用配置及其相应的蓝图定义，请参阅[examples](./basic-example.md) 页面。

## 高级

有关高级用例和 Output，请参阅[advanced](./advanced.md) 页面。
