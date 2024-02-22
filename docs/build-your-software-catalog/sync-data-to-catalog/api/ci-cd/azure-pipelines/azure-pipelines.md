---
sidebar_position: 1
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# Azure Pipelines

使用 Azure Pipelines，可以轻松创建/更新和查询 Port 中的实体。

<br></br>
<br></br>

![Github Illustration](/img/build-your-software-catalog/sync-data-to-catalog/azure-pipelines/azure-pipelines-illustration.jpg)

## 💡 常见的 Azure Pipelines Usage

例如，Port 的应用程序接口(API)可轻松实现 Port 与 Azure Pipelines 作业的集成: 

* 报告正在运行的**CI任务**的状态；
* 更新软件目录中有关**微服务**新**构建版本的信息；
* 获取现有**实体**。

## 设置

要使用 Azure Pipelines 与 Port 交互，首先需要将[define your Port credentials](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/set-secret-variables?view=azure-devops&amp;tabs=yaml%2Cbash#secret-variable-in-the-ui) 作为管道的变量。然后，将定义的变量传递给管道脚本，例如 `Python`: 

```yaml showLineNumbers
- task: PythonScript@0
  env:
    PORT_CLIENT_ID: $(PORT_CLIENT_ID) # The variable name for your Port clientId
    PORT_CLIENT_SECRET: $(PORT_CLIENT_SECRET) # The variable name for your Port clientSecret
  inputs:
    scriptSource: "filePath"
    scriptPath: "main.py"
```

请确保您的 Port 安装中已有[Blueprint](/build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md) ，以便创建/更新实体。

## 使用 Port 的 API

下面是一个示例片段，展示了如何使用 Python 将被用于 Port API 的作业与现有的 Azure 管道集成: 

在 Azure Pipelines 中添加以下任务: 

<details>
  <summary> Azure pipeline YAML </summary>

```yaml showLineNumbers
- script: |
    pip install -r port_requirements.txt
- task: PythonScript@0
  env:
    PORT_CLIENT_ID: $(PORT_CLIENT_ID)
    PORT_CLIENT_SECRET: $(PORT_CLIENT_SECRET)
  inputs:
    scriptSource: "filePath"
    scriptPath: "port.py"
```

</details>

<br></br>

:::note 在下面的示例中，我们使用了需要安装的 Python 模块。 你可以引用下面的 `requirements.txt`: 

<details>
  <summary> port_requirements.txt </summary>

```
requests>=2.28.2
```

</details>

:::

<Tabs groupId="usage" defaultValue="upsert" values={[
{label: "Create/Update", value: "upsert"},
{label: "Get", value: "get"}
]}>

<TabItem value="upsert">

在版本库中创建以下 Python 脚本，以创建或更新 Port 实体，作为管道的一部分: 

```python showLineNumbers
import os
import requests
import json

# These are the credentials passed by the variables of your pipeline to your tasks and in to your env
CLIENT_ID = os.environ['PORT_CLIENT_ID']
CLIENT_SECRET = os.environ['PORT_CLIENT_SECRET']

credentials = {
    'clientId': CLIENT_ID,
    'clientSecret': CLIENT_SECRET
}
token_response = requests.post(f"{API_URL}/auth/access_token", json=credentials)
access_token = token_response.json()['accessToken']

headers = {
    'Authorization': f'Bearer {access_token}'
}

entity_json = {
        "identifier": "example-entity",
        "properties": {
          "myStringProp": "My value",
          "myNumberProp": 1,
          "myBooleanProp": true,
          "myArrayProp": ["myVal1", "myVal2"],
          "myObjectProp": {"myKey": "myVal", "myExtraKey": "myExtraVal"}
      }
}

# request url : {API_URL}/blueprints/<blueprint_id>/entities
create_response = requests.post(f'{API_URL}/blueprints/test-blueprint/entities?upsert=true', json=entity_json, headers=headers)
print(json.dumps(get_response.json(), indent=4))
```

</TabItem>
<TabItem value="get">

在版本库中创建以下 Python 脚本，以获取 Port 实体作为管道的一部分: 

```python showLineNumbers
import os
import requests
import json

# These are the credentials passed by the variables of your pipeline to your tasks and in to your env
CLIENT_ID = os.environ['PORT_CLIENT_ID']
CLIENT_SECRET = os.environ['PORT_CLIENT_SECRET']

credentials = {
    'clientId': CLIENT_ID,
    'clientSecret': CLIENT_SECRET
}
token_response = requests.post(f"{API_URL}/auth/access_token", json=credentials)
access_token = token_response.json()['accessToken']

headers = {
    'Authorization': f'Bearer {access_token}'
}

# request url : {API_URL}/blueprints/<blueprint_id>/entities/<entity_id>
get_response = requests.get(f"{API_URL}/blueprints/test-blueprint/entities/test-entity", headers=headers)
print(json.dumps(get_response.json(), indent=4))
```

</TabItem>
</Tabs>

## 示例

有关使用 Azure Pipelines 处理 Port 的实用示例，请引用[examples](./examples.md) 页面。