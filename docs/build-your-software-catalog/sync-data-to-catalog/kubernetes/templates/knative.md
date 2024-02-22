---

sidebar_position: 3
description: Knative 快速入门

---

import TemplateInstallation from "./_template_installation.mdx";
import TemplatePrerequisites from "./_template_prerequisites.mdx";

# Knative

[Knative](https://knative.dev/docs/concepts/) 是一个开源社区项目，它通过引入组件来增强 Kubernetes，从而促进无服务器云原生应用程序的部署、执行和管理。

使用 Port 的 Kubernetes Exporter，您可以跟踪不同集群中的 Knative 资源，并将所有数据导出到 Port。 您将使用来自 kubernetes 资源和 CRD 的内置元数据在 Port 中创建实体，并跟踪其状态。

:::tip 了解 Kubernetes 输出程序的基础知识[here!](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md)

:::

## 先决条件

<TemplatePrerequisites />

## 设置蓝图和资源映射

下文将指导您使用安装脚本设置蓝图和资源映射。您可以阅读有关安装脚本的更多信息[here](#how-does-the-installation-script-work) 。

### 创建蓝图

安装脚本提供了一种创建蓝图的便捷方法。 使用 `CUSTOM_BP_PATH` 环境变量，您可以获取预定义的 `blueprints.json` 来创建蓝图。 在本例中，您将使用[this file](https://github.com/port-labs/template-assets/blob/main/kubernetes/blueprints/kubernetes_knative_usecase.json) 来定义蓝图。请通过运行

```bash showLineNumbers
export CUSTOM_BP_PATH="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/blueprints/kubernetes_knative_usecase.json"
```

该 `blueprints.json` 文件定义了以下蓝图: 

* 集群；
* namespace；
* 节点
* Pod
* 工作负载 *；
* Knative 服务；
* Knative 配置；
* Knative Revision；
* Knative 路由。

:::note 

* Workload "是创建和管理 pod 的 Kubernetes 对象的抽象。通过创建该蓝图，可以避免为每种工作负载类型创建一个专用蓝图，因为所有这些蓝图可能看起来都非常相似。
以下是 "Workload "将代表的 kubernetes 对象列表: 
    - 部署；
    - 复制集
    - StatefulSet；
    - DaemonSet。

:::

### 导出自定义资源映射

使用 `CONFIG_YAML_URL` 参数，可以定义自定义资源映射，以便在安装导出程序时使用。

在本例中，您将被引用 ** [this configuration file](https://github.com/port-labs/template-assets/blob/main/kubernetes/full-configs/kubernetes_kantive_usecase.yaml)**。为此，请运行

```bash showLineNumbers
export CONFIG_YAML_URL="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/full-configs/kubernetes_kantive_usecase.yaml"
```

现在，您可以使用以下代码片段运行安装脚本: 

```bash showLineNumbers
export CLUSTER_NAME="my-cluster"
export PORT_CLIENT_ID="my-port-client-id"
export PORT_CLIENT_SECRET="my-port-client-secret"
curl -s https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/install.sh | bash
```

现在，您可以浏览 Port 环境，查看蓝图是否已创建，您的 k8s 和 Knative 资源是否正在使用新安装的 k8s 导出器向 Port 报告。

## 安装脚本如何工作？

<TemplateInstallation />