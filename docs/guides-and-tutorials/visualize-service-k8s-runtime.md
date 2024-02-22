---
sidebar_position: 4
sidebar_label: 可视化服务的 k8s 运行时间

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# 可视化服务的 k8s 运行时间

本指南只需 10 分钟即可完成，旨在展示 Port 与 Kubernetes 集成的价值。

:::tip  先决条件

* 本指南假定您已拥有 Port 账户，并已完成[onboarding process](/quickstart) 。我们将使用入职过程中创建的 "服务 "蓝图。
* 您需要一个可访问的 k8s 集群。如果没有，下面是如何快速建立一个[minikube cluster](https://minikube.sigs.k8s.io/docs/start/) 。
* [Helm](https://helm.sh/docs/intro/install/) - 安装 Port 的 Kubernetes 输出程序。

:::

<br/>

### 本指南的目标

在本指南中，我们将对服务的 Kubernetes 资源进行建模和可视化。

完成这项工作后，你就会了解它如何使你的组织中的不同角色受益: 

* 开发人员将能轻松查看其服务 k8s 运行时的健康状况和状态。
* 平台工程师将能为组织内不同的利益相关者创建自定义视图和可视化效果。
* 平台工程师将能够设置、维护和跟踪 k8s 资源的标准。
* 研发经理将能够使用定制的视图和仪表板，跟踪服务 k8s 资源的任何数据。

<br/>

### 安装 Port 的 Kubernetes 输出程序

1. 进入[data sources page](https://app.getport.io/dev-portal/data-sources) ，点击 "+ 数据源"，找到 "Kubernetes Stack "类别并选择 "Kubernetes": 
2. 在指定集群名称后复制安装命令，它应该是这样的: 

```bash showLineNumbers
# The following script will install a K8s integration at your K8s cluster using helm
# Change the stateKey to describe your integration.
# For example, the name of the cluster it will be installed on.
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-cluster port-labs/port-k8s-exporter \
  --create-namespace --namespace port-k8s-exporter \
    --set secret.secrets.portClientId="YOUR_PORT_CLIENT_ID"  \
    --set secret.secrets.portClientSecret="YOUR_PORT_CLIENT_SECRET"  \
    --set portBaseUrl="https://api.getport.io"  \
    --set stateKey="my-cluster"  \
    --set eventListenerType="POLLING"  \
    --set "extraEnv[0].name"="CLUSTER_NAME"  \
    --set "extraEnv[0].value"="my-cluster"
```

#### 出口商做什么？

安装后，输出程序将

1. 在[Builder](https://app.getport.io/dev-portal/data-model) 中创建代表 Kubernetes 资源的<PortTooltip id="blueprint">蓝图</PortTooltip>(如[here](https://github.com/port-labs/port-k8s-exporter/blob/main/assets/defaults/blueprints.json) 所定义) : 

<img src='/img/guides/k8sBlueprintsCreated.png' width='95%' />

<br/><br/>

:::info  备注

Workload "是创建和管理 pod 的 Kubernetes 对象(如 "部署"、"StatefulSet"、"DaemonSet")的抽象。

:::

<br/>

2.在[Software catalog](https://app.getport.io/services) 中创建<PortTooltip id="entity">实体</PortTooltip>。您将看到每个<PortTooltip id="blueprint">蓝图</PortTooltip>都有一个新页面，其中包含您的资源，填充来自 Kubernetes 集群的数据(根据定义的默认映射[here](https://github.com/port-labs/port-k8s-exporter/blob/main/assets/defaults/appConfig.yaml) ) : 

<img src='/img/guides/k8sEntitiesCreated.png' width='100%' />

<br/><br/>

3.为代表你的 k8s 资源的蓝图创建<PortTooltip id="scorecard">记分卡</PortTooltip>(如[here](https://github.com/port-labs/port-k8s-exporter/blob/main/assets/defaults/scorecards.json)) 。这些记分卡定义了从 k8s 集群摄取数据的规则和检查，便于检查 K8s 资源是否符合标准。
4.创建仪表盘，为您提供从 K8s 集群摄取的数据的可视化视图。
5.监听Kubernetes集群中的变化，并相应更新你的<PortTooltip id="entity">实体</PortTooltip>。

<br/>

###定义服务与工作负载之间的联系

现在，我们已经建立了<PortTooltip id="blueprint">蓝图</PortTooltip>，我们希望通过将 "服务 "蓝图与 "工作负载 "蓝图联系起来，来模拟它们之间的逻辑联系。 这将在软件目录中为我们提供一些有用的上下文，使我们能够在 "服务 "的上下文中看到相关的 "工作负载"，或在其对应的 "服务 "中直接看到 "工作负载 "的属性。

在本指南中，我们将创建一个名为 "Prod_runtime "的关系，它将代表服务的生产环境。 在实际环境中，我们可以为暂存环境等创建另一个关系。

1. 进入[Builder](https://app.getport.io/dev-portal/data-model) ，展开 "服务 "蓝图，点击 "新建关系"。
2. 像这样填写表格，然后点击 "创建": 

<img src='/img/guides/k8sCreateRelation.png' width='50%' />

<br/><br/>

在查看 "服务 "时，它的某些 "工作量 "属性可能对我们特别重要，我们希望直接在 "服务 "的上下文中查看它们。 这可以通过[mirror properties](https://docs.getport.io/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/mirror-property/) 来实现，所以让我们创建一些: 

3.第一个将是工作量的健康状况。在我们刚刚创建的关系下，点击 "新建镜像属性": 

<img src='/img/guides/k8sCreateMirrorProp.png' width='50%' />

<br/><br/>

4.像这样填写表格，然后点击 "创建": 

<img src='/img/guides/k8sCreateMirrorPropHealth.png' width='50%' />

<br/><br/>

5.第二个将是工作量的镜像标签。创建另一个镜像属性，像这样填写表格，然后点击 "创建": 

<img src='/img/guides/k8sCreateMirrorPropImages.png' width='50%' />

<br/><br/>

###将您的工作负载映射到其服务

您可能已经注意到，我们创建的所有服务的 `Prod_runtime` 属性和镜像属性都是空的。 这是因为我们没有指定哪个 `workload` 属于哪个 `service`。 这可以手动完成，也可以通过映射使用您选择的约定。

在本指南中，我们将使用以下约定: 标签形式为 `portService:<service-identifier>` 的 `workload` 将自动分配给具有该标识符的 `service`。

例如，标签为 `portService: myService` 的 k8s 部署将被分配给标识符为 `myService` 的 `service`。

我们通过在安装出口程序时使用的配置 YAML 中添加[mapping definition](https://github.com/port-labs/template-assets/blob/main/kubernetes/full-configs/k8s-guide/k8s_guide_config.yaml#L111-L123) 来实现这一目标。该定义使用 `jq` 在属性之间执行计算。

**让我们来看看实际操作: **

1. 在集群中创建一个 "部署 "资源，其标签与[Software catalog](https://app.getport.io/services) 中的 "服务 "标识符相匹配。  
您可以使用下面的简单示例，并更改 `metadata.labels.portService` 值以匹配您所需的 `service`。将其复制到名为 `deployment.yaml` 的文件中，然后应用: 

```bash
kubectl apply -f deployment.yaml
```

<details>
<summary><b>Deployment example (Click to expand)</b></summary>

```yaml showLineNumbers
apiVersion: apps/v1
kind: Deployment
metadata:
  name: awesomeservice
  labels:
    app: nginx
    portService: AwesomeService
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

</details>

<br/>

2.要查看新数据，我们需要更新 k8s 输出程序被用来引用数据的映射配置。

要编辑映射，请访问[data sources page](https://app.getport.io/dev-portal/data-sources) ，找到 k8s 输出卡，点击它，就会看到一个显示当前配置的 YAML 编辑器。 在映射配置中添加以下块，然后点击 "Resync": 

```yaml showLineNumbers
resources:
  # ... Other resource mappings installed by the K8s exporter
  - kind: apps/v1/deployments
      port:
        entity:
          mappings:
          - blueprint: '"service"'
            icon: '"Deployment"'
            identifier: .metadata.labels.portService
            properties: {}
            relations:
              prod_runtime: .metadata.name + "-Deployment-" + .metadata.namespace + "-" + "my-cluster"
            title: .metadata.name
      selector:
        query: .metadata.namespace | startswith("kube") | not
```

<br/>

3.进入[Software catalog](https://app.getport.io/services) ，点击 `Services`。单击为其创建部署的 `Service` ，你应该会看到 `Prod_runtime` 属性以及我们镜像的 `Health` 和 `Images` 属性已被填充: 

<img src='/img/guides/k8sEntityAfterIngestion.png' width='80%' />

<br/><br/>

###可视化 Kubernetes 环境中的数据

现在，我们已经掌握了大量有关工作负载的数据，以及一些跟踪其质量的指标。 让我们看看如何以有利于开发人员和管理人员日常工作的方式将这些信息可视化。

#### 在主页上添加 "不健康服务 "表

在 Provider 为本指南提供的配置中，如果 "工作负载 "定义的副本数等于其可用副本数，则该 "工作负载 "被视为 "健康"(当然，您也可以更改此定义)。

1. 进入[homepage](https://app.getport.io/home) ，点击右上角的 "+ 添加 "按钮，然后选择 "表格"。
2. 像这样填写表格，然后点击 "保存": 

<img src='/img/guides/k8sHomepageTableUnhealthyServices.png' width='50%' />

<br/><br/>

3.在新表格中，点击 "过滤器"，然后点击 "+ 添加新过滤器"。像这样填写字段: 

<img src='/img/guides/k8sHomepageTableFilterUnhealthy.png' width='50%' />

<br/><br/>

现在，您可以在主页上跟踪需要您关注的服务。

<img src='/img/guides/k8sHomepageTableUnhealthyFilter.png' width='40%' />

这些服务未列入本指南，但可作为本表的示例_。

#### 使用记分卡清楚地了解工作负载的可用性概况

在本指南提供的配置中，可用性指标是这样定义的: 

* 青铜: >=1 个复制品
* 银奖>=2个复制品
* 金奖>=3个复制品

要全面了解工作负载的可用性，我们可以使用表格操作。

1. 请访问[`Workloads` catalog page](https://app.getport.io/workloads) 。
2. 点击 "分组方式 "按钮，然后从下拉菜单中选择 "高可用性": 

<img src='/img/guides/k8sGroupByAvailability.png' width='40%' />

<br/><br/>

3.点击任何一个指标级别，即可查看相应的工作负载: 

<img src='/img/guides/k8sWorkloadsAfterGroupByAvailability.png' width='90%' />

<br/><br/>

请注意，您也可以通过单击 "保存此视图 "按钮将其设置为默认视图 📝

### 可能的日常整合

* 在 R&amp;D 频道中发送 slack 消息，让大家知道创建了新的部署。
* 在服务可用性下降时通知 Devops 工程师。
* 向研发经理发送周报/月报，显示服务生产运行时间的健康状况。

#### 结论

Kubernetes 是一个复杂的环境，需要高质量的可观察性。 Port 的 Kubernetes 集成使您可以轻松地对 Kubernetes 资源进行建模和可视化，并将其集成到您的日常工作中。 您可以自定义视图，显示与您相关的数据，并按团队、namespace 或任何其他标准进行分组或筛选。 借助 Port，您可以无缝地满足您组织的需求，并为您的 Kubernetes 资源创建一个单一的真相源。

更多指南和教程即将推出，在此期间如有任何问题，请随时通过[community slack](https://www.getport.io/community) 或[Github project](https://github.com/port-labs?view_as=public) 联系我们。