---

sidebar_position: 5
description: Trivy 快速入门

---

import TemplateInstallation from "./_template_installation.mdx";
import TemplatePrerequisites from "./_template_prerequisites.mdx";

# 特里维操作员

[Trivy Operator](https://github.com/aquasecurity/trivy-operator) 是一款开源安全扫描仪，它利用[Trivy](https://github.com/aquasecurity/trivy) 持续扫描 Kubernetes 集群，查找安全问题。

使用 Port 的 Kubernetes Exporter，您可以跟踪不同集群中的所有 Trivy 资源，并将所有安全问题导出到 Port。 您将使用 kubernetes 资源和 CRD 的内置元数据在 Port 中创建实体，并跟踪其状态。

:::tip 了解 Kubernetes 输出程序的基础知识[here!](/build-your-software-catalog/sync-data-to-catalog/kubernetes/kubernetes.md)

:::

## 先决条件

<TemplatePrerequisites />

## 设置蓝图和资源映射

下文将指导您使用安装脚本设置蓝图和资源映射。您可以阅读有关安装脚本的更多信息[here](#how-does-the-installation-script-work) 。

### 创建蓝图

安装脚本提供了一种创建蓝图的便捷方法。 使用 `CUSTOM_BP_PATH` 环境变量，您可以获取预定义的 `blueprints.json` 来创建蓝图。 在本例中，您将使用[this file](https://github.com/port-labs/template-assets/blob/main/kubernetes/blueprints/trivy-blueprints.json) 来定义蓝图。请通过运行

```bash showLineNumbers
export CUSTOM_BP_PATH="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/blueprints/trivy-blueprints.json"
```

该 `blueprints.json` 文件定义了以下蓝图: 

* 集群；
* namespace；
* 节点
* Pod
* ReplicaSet；
* 工作负载 *；
* Trivy 漏洞。

:::note 

* Workload "是创建和管理 pod 的 Kubernetes 对象的抽象。
通过创建该蓝图，可以避免为每种工作负载类型创建专用蓝图，因为所有这些蓝图都可能
看起来非常相似。
以下是 "Workload "将代表的 kubernetes 对象列表: 
    - 部署；
    - StatefulSet；
    - DaemonSet。
* `Trivy Vulnerabilities` 是最重要的 Trivy 资源之一，让开发人员能够查找和查看与 Kubernetes 集群中不同资源相关的风险。

:::

### 导出自定义资源映射

使用 `CONFIG_YAML_URL` 参数，可以定义自定义资源映射，以便在安装导出程序时使用。

在本例中，您将被引用 ** [this configuration file](https://github.com/port-labs/template-assets/blob/main/kubernetes/templates/trivy-kubernetes_v1_config.yaml)**。为此，请运行

```bash showLineNumbers
export CONFIG_YAML_URL="https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/templates/trivy-kubernetes_v1_config.yaml"
```

现在，您可以使用以下代码片段运行安装脚本: 

```bash showLineNumbers
export CLUSTER_NAME="my-cluster"
export PORT_CLIENT_ID="my-port-client-id"
export PORT_CLIENT_SECRET="my-port-client-secret"
curl -s https://raw.githubusercontent.com/port-labs/template-assets/main/kubernetes/install.sh | bash
```

现在您可以浏览您的 Port 环境，查看蓝图是否已创建，您的 k8s 和 Trivy 资源是否正在使用新安装的 k8s 导出器向 Port 报告。

## 安装脚本如何工作？

<TemplateInstallation />

## 使用脚本的另一种整合方式

虽然上述 Trivy Kubernetes 输出程序是推荐的安装方法，但您可能更喜欢使用 webhook 和脚本来[ingest your Trivy scan results to Port](/build-your-software-catalog/sync-data-to-catalog/webhook/examples/packages/trivy) 。