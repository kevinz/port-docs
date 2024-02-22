---

sidebar_position: 9
description: 将 Checkmarx KICS 扫描结果摄入目录

---

import PythonScript from './resources/checkmarx/_example_python_script.mdx'
import CheckmarxBlueprint from './resources/checkmarx/_example_checkmarx_blueprint.mdx'
import CheckmarxWebhookConfig from './resources/checkmarx/_example_checkmarx_webhook_config.mdx'

# Checkmarx KICS

在本例中，您将创建一个 `checkmarxScan` 蓝图，该蓝图将使用 Port's[API](../../../api/api.md) 和[webhook functionality](../../webhook.md) 的组合在 Checkmarx KICS 文件中引用所有扫描结果。

要将扫描结果引用到 Port，可使用脚本根据 webhook 配置发送扫描信息。

## 先决条件

创建以下蓝图定义和 webhook 配置: 

<details>
<summary>Checkmarx KICS blueprint</summary>
<CheckmarxBlueprint/>
</details>

<details>
<summary>Checkmarx KICS webhook configuration</summary>
<CheckmarxWebhookConfig/>

</details>

## 使用 Port 的 API 和 Python 脚本

下面的示例片段展示了如何使用 Python 将 Port 的 API 和 webhook 与现有的 Pipelines 集成: 

<details>
<summary>Python script example</summary>

<PythonScript/>

</details>