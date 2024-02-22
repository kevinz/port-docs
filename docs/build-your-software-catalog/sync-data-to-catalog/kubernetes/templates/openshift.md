---
title: Red Hat Openshift

sidebar_position: 4
description: Openshift 快速入门

---

import TemplateInstallation from "./_template_installation.mdx";
import TemplatePrerequisites from "./_template_prerequisites.mdx";

# Red Hat Openshift

[Red Hat Openshift](https://www.redhat.com/en/technologies/cloud-computing/openshift) 是通过 Kubernetes 进行可扩展应用程序开发、现代化和部署的多功能平台，可为在您首选的基础设施上交付应用程序提供一整套服务。

使用 Port 的 Kubernetes Exporter，您可以跟踪不同集群中的重要 Openshift 资源，并将数据导出到 Port。 您将从 Openshift 资源和 CRD 中引用内置元数据，在 Port 中创建实体并跟踪其状态。

:::tip 了解 Kubernetes 输出程序的基础知识[here!](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md)

:::

## 绘制 Red Hat Openshift - 目标

虽然 Red Hat Openshift 在您的 Openshift(Kubernetes)环境方面提供了很好的可视性，但仍然存在一些问题，例如您的 Openshift 环境如何与基础架构的其他部分进行连接和交互: 

* 集群在哪个云 Provider 中运行？
* 集群在哪个 VPC 中运行？
* 某个集群的值班人员是谁？
* 一个云区域中所有不同的 Openshift 集群提供的所有端点是什么？

将您的 Openshift 资源被用于到 Port 后，就可以轻松地为不同的用例创建多个定制视图。 例如，您可以创建一个视图，向您展示 Openshift 集群如何与基础架构的其他部分进行交互，也可以创建一个高级视图，让管理层了解 Openshift 安装所提供的业务价值。

在这个示例中，您将映射您的 Openshift 集群、它们的工作负载以及不同集群所暴露的 Openshift 路由。

:::tip 了解 Kubernetes 输出程序的基础知识[here!](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md)

:::

## 先决条件

<TemplatePrerequisites />

## 设置蓝图和资源映射

下文将指导您使用安装脚本设置蓝图和资源映射。您可以阅读有关安装脚本的更多信息[here](#how-does-the-installation-script-work) 。

### 创建蓝图

安装脚本提供了一种创建蓝图的便捷方法。 使用 `CUSTOM_BP_PATH` 环境变量，您可以获取预定义的 `blueprints.json` 来创建蓝图。 在本例中，您将使用[this file](https://github.com/port-labs/template-assets/blob/main/kubernetes/blueprints/openshift-blueprints.json) 来定义蓝图。请通过运行

```bash showLineNumbers
export CUSTOM_BP_PATH="https://github.com/port-labs/template-assets/blob/main/kubernetes/blueprints/openshift-blueprints.json"
```

该 `blueprints.json` 文件定义了以下蓝图: 

* 集群；
* namespace；
* 节点
* Pod
* ReplicaSet；
* 工作负载 *；
* 服务；
* Openshift 路由 *。

:::note 

* Workload "是创建和管理 pod 的 Kubernetes 对象的抽象。通过创建该蓝图，可以避免为每种工作负载类型创建一个专用蓝图，因为所有这些蓝图可能看起来都非常相似。
以下是 "Workload "将代表的 kubernetes 对象列表: 
    - 部署；
    - StatefulSet；
    - DaemonSet。
* `Openshift Route` 是 Openshift 最重要的资源之一，它赋予开发人员连接服务的能力，而整个网络层由 Openshift API 管理，并提供简单的 DNS 记录以实现可访问性。

:::

### 导出自定义资源映射

使用 `CONFIG_YAML_URL` 参数，可以定义自定义资源配置，以便在安装导出程序时使用。

在本例中，您将被用于 ** [this configuration file](https://github.com/port-labs/template-assets/blob/main/kubernetes/full-configs/openshift_usecase.yaml)**。为此，请运行

```bash showLineNumbers
export CONFIG_YAML_URL="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/full-configs/openshift_usecase.yaml"
```

现在，您可以使用以下代码片段运行安装脚本: 

```bash showLineNumbers
export CLUSTER_NAME="my-cluster"
export PORT_CLIENT_ID="my-port-client-id"
export PORT_CLIENT_SECRET="my-port-client-secret"
curl -s https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/install.sh | bash
```

现在，您可以浏览 Port 环境，查看蓝图是否已创建，Kubernetes 资源(包括 Openshift 路由)是否正在使用新安装的 k8s 输出程序向 Port 报告。

## 安装脚本如何工作？

<TemplateInstallation />