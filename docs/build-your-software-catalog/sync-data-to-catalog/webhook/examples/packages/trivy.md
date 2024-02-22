---

sidebar_position: 10
description: 将 Trivy 漏洞纳入您的目录

---

import PythonScript from './resources/trivy/_example_python_script.mdx'
import TrivyBlueprint from './resources/trivy/_example_trivy_blueprint.mdx'
import TrivyWebhookConfig from './resources/trivy/_example_trivy_webhook_config.mdx'

# Trivy

在本示例中，您将创建一个 `trivyVulnerability` 蓝图，使用 Port's[API](../../../api/api.md) 和[webhook functionality](../../webhook.md) 的组合来引用 Trivy 结果文件中的所有漏洞。

:::info  推荐安装选项 虽然本示例中提供的脚本便于按计划将 Trivy 扫描结果摄取到 Port，但我们强烈建议您[use our Trivy Kubernetes exporter](/build-your-software-catalog/sync-data-to-catalog/kubernetes/templates/trivy) ，以持续扫描 kubernetes 集群并实时将漏洞摄取到 Port。

:::

要将扫描结果引用到 Port，可使用一个脚本，根据 webhook 配置发送有关漏洞的信息。

## 先决条件

创建以下蓝图定义和 webhook 配置: 

<details>
<summary>Trivy vulnerability blueprint</summary>
<TrivyBlueprint/>
</details>

<details>
<summary>Trivy webhook configuration</summary>
<TrivyWebhookConfig/>

</details>

## 使用 Port 的 API 和 Python 脚本

下面的示例片段展示了如何使用 Python 将 Port 的 API 和 webhook 与现有的 Pipelines 集成: 

<details>
<summary>Python script example</summary>

<PythonScript/>

</details>