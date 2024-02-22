---
sidebar_position: 2
description: Istio 快速入门

---

import TemplateInstallation from "./_template_installation.mdx";
import TemplatePrerequisites from "./_template_prerequisites.mdx";

# Istio

[Istio](https://istio.io/latest/docs/setup/getting-started/) 是一种开源服务网格，它提供了一种连接、管理和保护微服务的统一方式。

使用 Port 的 Kubernetes Exporter，您可以跟踪不同集群中的所有 Istio 资源，并将所有数据导出到 Port。 您将使用来自 kubernetes 资源和 CRD 的内置元数据在 Port 中创建实体并跟踪其状态。

:::tip 了解 Kubernetes 输出程序的基础知识[here!](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md)

:::

## 先决条件

<TemplatePrerequisites />

## 设置蓝图和资源映射

下文将指导您使用安装脚本设置蓝图和资源映射。您可以阅读有关安装脚本的更多信息[here](#how-does-the-installation-script-work) 。

### 创建蓝图

安装脚本提供了一种创建蓝图的便捷方法。 使用 `CUSTOM_BP_PATH` 环境变量，您可以获取预定义的 `blueprints.json` 来创建蓝图。 在本例中，您将使用[this file](https://github.com/port-labs/template-assets/blob/main/kubernetes/blueprints/istio-blueprints.json) 来定义蓝图。请通过运行

```bash showLineNumbers
export CUSTOM_BP_PATH="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/blueprints/istio-blueprints.json"
```

该 `blueprints.json` 文件定义了以下蓝图: 

* 集群；
* namespace；
* 节点
* Pod
* 工作负载 *；
* Istio Gateway；
* Istio 虚拟服务。

:::note 

* Workload "是创建和管理 pod 的 Kubernetes 对象的抽象。
通过创建该蓝图，可以避免为每种工作负载类型创建专用蓝图，因为所有这些蓝图都可能
看起来非常相似。
以下是 "Workload "将代表的 kubernetes 对象列表: 
    - 部署；
    - 复制集
    - StatefulSet；
    - DaemonSet。

:::

### 导出自定义资源映射

使用 `CONFIG_YAML_URL` 参数，可以定义自定义资源映射，以便在安装导出程序时使用。

在本例中，您将被用于 ** [this configuration file](https://github.com/port-labs/template-assets/blob/main/kubernetes/templates/istio-kubernetes_v1_config.yaml)**。为此，请运行

```bash showLineNumbers
export CONFIG_YAML_URL="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/templates/istio-kubernetes_v1_config.yaml"
```

现在，您可以使用以下代码片段运行安装脚本: 

```bash showLineNumbers
export CLUSTER_NAME="my-cluster"
export PORT_CLIENT_ID="my-port-client-id"
export PORT_CLIENT_SECRET="my-port-client-secret"
curl -s https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/install.sh | bash
```

现在，您可以浏览 Port 环境，查看蓝图是否已创建，您的 k8s 和 Istio 资源是否已使用新安装的 k8s 导出器报告到 Port。

## 安装脚本如何工作？

<TemplateInstallation />