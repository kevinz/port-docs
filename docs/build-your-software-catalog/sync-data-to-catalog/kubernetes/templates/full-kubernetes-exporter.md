---
sidebar_position: 1
description: 扩展 Kubernetes 安装

---

import TemplateInstallation from "./_template_installation.mdx";
import TemplatePrerequisites from "./_template_prerequisites.mdx";

# Kubernetes(扩展自)

:::info 本用例是[basic Kubernetes use-case](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md) 的扩展。如果您还没有阅读，建议您先阅读本用例。

:::

Kubernetes 已成为部署基于微服务的应用程序的最流行方式之一。 随着微服务数量的增长，以及在多个区域部署更多集群，跟踪所有部署、服务和作业变得复杂而乏味。

使用 Port 的 Kubernetes Exporter，您可以跟踪 K8s 资源并将所有数据导出到 Port。 您将使用 K8s 内置的元数据在 Port 中创建实体并跟踪其状态。

:::tip 了解 Kubernetes 输出程序的基础知识[here!](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md)

:::

## 先决条件

<TemplatePrerequisites />

## 设置蓝图和资源映射

下文将指导您使用安装脚本设置蓝图和资源映射。您可以阅读有关安装脚本的更多信息[here](#how-does-the-installation-script-work) 。

### 创建蓝图

安装脚本提供了一种创建蓝图的便捷方法。 使用 `CUSTOM_BP_PATH` 环境变量，您可以获取预定义的 `blueprints.json` 来创建蓝图。 在本例中，您将使用[this file](https://github.com/port-labs/template-assets/blob/main/kubernetes/blueprints/kubernetes_complete_usecase_bps.json) 来定义蓝图。请通过运行

```bash showLineNumbers
export CUSTOM_BP_PATH="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/blueprints/kubernetes_complete_usecase_bps.json"
```

该 `blueprints.json` 文件定义了以下蓝图: 

* 集群；
* namespace；
* 节点
* Pod
* 复制集
* 工作负载 *；

:::note 

* Workload "是创建和管理 pod 的 Kubernetes 对象的抽象。通过创建该蓝图，可以避免为每种工作负载类型创建一个专用蓝图，因为所有这些蓝图可能看起来都非常相似。
以下是 "Workload "将代表的 kubernetes 对象列表: 
    - 部署；
    - StatefulSet；
    - DaemonSet。

:::

### 导出自定义资源映射

使用 `CONFIG_YAML_URL` 参数，可以定义自定义资源映射，以便在安装导出程序时使用。

在本例中，您将被引用 ** [this configuration file](https://github.com/port-labs/template-assets/blob/main/kubernetes/kubernetes_v1_config.yaml)**。为此，请运行

```bash showLineNumbers
export CONFIG_YAML_URL="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/kubernetes_v1_config.yaml"
```

现在，您可以使用以下代码片段运行安装脚本: 

```bash showLineNumbers
export CLUSTER_NAME="my-cluster"
export PORT_CLIENT_ID="my-port-client-id"
export PORT_CLIENT_SECRET="my-port-client-secret"
curl -s https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/install.sh | bash
```

就是这样！现在您可以浏览 Port 环境，查看蓝图是否已创建，实体是否已使用新安装的 k8s 导出器报告到 Port。

## 摘要

在这种被引用的情况下，您可以使用安装脚本: 

* 通过创建定义不同 k8s 资源的蓝图来设置 Port 环境；
* 通过配置安装 Port 的 k8s 导出器，以便从集群导出重要数据；
* 将集群中的 k8s 资源作为实体提取到 Port 环境中

## 安装脚本如何工作？

<TemplateInstallation />