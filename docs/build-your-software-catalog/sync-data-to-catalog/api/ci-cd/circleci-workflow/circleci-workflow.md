---
sidebar_position: 1
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# CircleCI 工作流程

被用于 CircleCI 工作流后，您可以在 Port 中轻松创建/更新和查询实体。

<br></br>
<br></br>

![Github Illustration](/img/build-your-software-catalog/sync-data-to-catalog/circleci/circleci-illustration.jpg)

## 💡 常见的 CircleCI 工作流程 Usage

例如，Port 的应用程序接口(API)可以轻松实现 Port 与 CircleCI 工作流的集成: 

* 报告正在运行的**CI任务**的状态；
* 更新软件目录中有关**微服务**新**构建版本的信息；
* 获取现有**实体**。

## 设置

要使用 Circle CI 与 Port 交互，首先需要设置[CircleCI context](https://circleci.com/docs/contexts/) ，以便保存 Port 凭据，并将上下文传递给相关工作流。

```yaml showLineNumbers
workflows:
  deploy-service:
    jobs:
      # highlight-start
      - report-to-port:
          context:
            # the CircleCI context name of the credentials
            - port
            # highlight-end
```

请确保您的 Port 安装中已有[Blueprint](/build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md) ，以便创建/更新实体。

## 使用 Port 的 API

下面的示例片段展示了如何使用 Python 将被用于 Port API 的作业与现有的 CircleCI 管道集成: 

将以下任务和工作流程添加到 CircleCI Pipelines 中: 

<details>
  <summary> CircleCI Pipeline YAML </summary>

```yaml showLineNumbers
jobs:
  # ... other jobs
    report-to-port:
      docker:
        - image: cimg/python:3.11
      environment:
        API_URL: https://api.getport.io
      steps:
        - checkout
        - run: pip install -r port_requirements.txt
        - run: python get_port_entity.py

workflows:
  # ... other workflows
  deploy-production-service:
    jobs:
      # ... other jobs
      # highlight-start
      - report-to-port:
        context:
          - port
      # highlight-end
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

# These are the credentials passed by the 'port' context to your environment variables
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

# These are the credentials passed by the 'port' context to your environment variables
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

有关使用 CircleCI 处理 Port 的实用示例，请参阅[examples](./examples.md) 页面。