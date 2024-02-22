---

sidebar_position: 6
description: Kyverno 快速启动

---

import TemplateInstallation from "./_template_installation.mdx";
import TemplatePrerequisites from "./_template_prerequisites.mdx";

# Kyverno

[Kyverno](https://kyverno.io/) Kyverno 是一个专为 Kubernetes 设计的策略引擎。 Kyverno 策略可以验证、mutation、生成和清理 Kubernetes 资源，允许集群管理员为其集群执行最佳配置实践。

使用 Port 的 Kubernetes 导出器，您可以跟踪不同集群中的所有 Kyverno 资源，并将所有策略和报告导出到 Port。 您将使用来自 kubernetes 资源和 CRD 的内置元数据在 Port 中创建实体，并跟踪其状态。

:::tip  我们的 Kubernetes 输出程序基础知识 了解我们的 Kubernetes 输出程序基础知识[here!](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md)

:::

## 先决条件

<TemplatePrerequisites />

## 设置蓝图和资源映射

下文将指导您使用安装脚本设置蓝图和资源映射。您可以阅读有关安装脚本的更多信息[here](#how-does-the-installation-script-work) 。

### 创建蓝图

安装脚本提供了一种创建蓝图的便捷方法。 使用 `CUSTOM_BP_PATH` 环境变量，您可以获取预定义的 `blueprints.json` 来创建蓝图。 在本例中，您将使用[this file](https://github.com/port-labs/template-assets/blob/main/kubernetes/blueprints/kyverno-blueprints.json) 来定义蓝图。请通过运行

```bash showLineNumbers
export CUSTOM_BP_PATH="https://github.com/port-labs/template-assets/blob/main/kubernetes/blueprints/kyverno-blueprints.json"
```

该 `blueprints.json` 文件定义了以下蓝图: 

* 集群
* namespace
* 节点
* 节点
* 复制集
* 工作量
* Kyverno 策略
* Kyverno 策略报告

:::note  蓝图信息

* Workload "是创建和管理 pod 的 Kubernetes 对象的抽象。
通过创建该蓝图，可以避免为每种工作负载类型创建专用蓝图，因为所有这些蓝图都可能
看起来非常相似。
以下是 "Workload "将代表的 kubernetes 对象列表: 
    - 部署
    - 状态集
    - 守护进程集
* Kyverno策略 "是最重要的Kyverno资源之一，让开发人员能够在Kubernetes集群中设置和执行策略规则。
* Kyverno策略报告 "是另一个重要的Kyverno资源，它包含将策略应用到Kubernetes集群的结果。

:::

### 导出自定义资源映射

使用 `CONFIG_YAML_URL` 参数，可以定义自定义资源映射，以便在安装导出程序时使用。

在本例中，您将被引用 ** [this configuration file](https://github.com/port-labs/template-assets/blob/main/kubernetes/templates/kyverno-kubernetes_v1_config.yaml)**。为此，请运行

```bash showLineNumbers
export CONFIG_YAML_URL="https://github.com/port-labs/template-assets/blob/main/kubernetes/templates/kyverno-kubernetes_v1_config.yaml"
```

现在，您可以使用以下代码片段运行安装脚本: 

```bash showLineNumbers
export CLUSTER_NAME="my-cluster"
export PORT_CLIENT_ID="my-port-client-id"
export PORT_CLIENT_SECRET="my-port-client-secret"
curl -s https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/install.sh | bash
```

现在您可以浏览您的 Port 环境，查看蓝图是否已创建，您的 k8s 和 Kyverno 资源是否正在使用新安装的 k8s 导出器向 Port 报告。

## 安装脚本如何工作？

<TemplateInstallation />