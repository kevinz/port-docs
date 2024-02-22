---

sidebar_position: 8
description: 将 SOAP API 定义纳入目录

---

import PythonScript from './resources/soap/_example_python_script.mdx'
import SOAPBlueprint from './resources/soap/_example_soap_blueprint.mdx'
import SOAPWebhookConfig from './resources/soap/_example_soap_webhook_config.mdx'

# SOAP 应用程序接口

在本示例中，您将创建一个 `soapApi` 蓝图，该蓝图将使用 Port's[API](../../../api/api.md) 和[webhook functionality](../../webhook.md) 的组合来引用 SOAP API 定义文件中的所有路径。

要将 API 路径引用到 Port，需要使用一个脚本，根据 webhook 配置发送有关 API 定义的信息。

## 先决条件

创建以下蓝图定义和 webhook 配置: 

<details>
<summary>SOAP blueprint</summary>
<SOAPBlueprint/>
</details>

<details>
<summary>Package webhook configuration</summary>

<SOAPWebhookConfig/>

</details>

## 使用 Port 的 API 和 Python 脚本

下面的示例片段展示了如何使用 Python 将 Port 的 API 和 webhook 与现有的 Pipelines 集成: 

<details>
<summary>Python script example</summary>

<PythonScript/>

</details>