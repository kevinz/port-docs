# 反映action进展

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

调用Port自助服务操作会在Port内创建一个 `actionRun` 对象。

:::tip 要了解有关配置自助服务操作的更多信息，请参阅[setup UI for actions](../setup-ui-for-action/setup-ui-for-action.md) 页面。配置自助服务操作后，调用该操作将生成 "actionRun"，您可以在本教程中了解更多信息。

:::

您可以通过以下方法之一查找所有现有的操作运行: 

1. 在 "审计日志 "页面上选择 "运行 "选项卡；
2. 在[specific entity page](../../customize-pages-dashboards-and-plugins/page/entity-page.md) 上选择特定实体的 "运行 "选项卡；
3. 从用户界面调用自助服务操作时，页面上会出现祝酒词，其中包含与自助服务操作的运行相对应的操作运行链接。

本教程将教您如何使用 Port 的 API 获取现有的操作运行，用附加元数据和有关所引用自助服务操作结果的信息更新它们，并将它们标记为已完成或失败，以保持所引用自助服务操作及其状态的历史记录一致。

## 设置

在本教程中，您将与从添加到 "微服务 "蓝图的基本 "创建微服务 "自服务操作中创建的操作运行进行交互。

本教程中我们将被引用的蓝图定义和自助服务操作详见下文: 

:::note 由于蓝图和自助服务操作不是本教程的重点，因此有意将其简化。 如有需要，可以很容易地扩展它们，以包含所需的额外属性。

:::

<details>
<summary>Microservice Blueprint</summary>

```json showLineNumbers
{
  "identifier": "microservice",
  "description": "This blueprint represents service in our software catalog",
  "title": "Service",
  "icon": "Microservice",
  "schema": {
    "properties": {
      "region": {
        "type": "string",
        "title": "Region"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</details>

<details>
<summary>Create microservice Self-Service Action</summary>

```json showLineNumbers
[
  {
    "identifier": "create_microservice",
    "title": "Create Microservice",
    "userInputs": {
      "properties": {
        "name": {
          "title": "Service name",
          "type": "string"
        },
        "region": {
          "title": "Cloud Region",
          "type": "string"
        }
      }
    },
    "invocationMethod": {
      "url": "https://getport.io",
      "agent": false,
      "type": "WEBHOOK"
    },
    "trigger": "CREATE",
    "description": "Create new microservice"
  }
]
```

</details>

<details>
<summary>2nd day operation microservice Self-Service Action</summary>

```json showLineNumbers
[
  {
    "identifier": "deploy_microservice",
    "title": "Deploy Microservice",
    "userInputs": {
      "properties": {
        "environment": {
          "title": "Environment",
          "type": "string"
        }
      }
    },
    "invocationMethod": {
      "url": "https://getport.io",
      "agent": false,
      "type": "WEBHOOK"
    },
    "trigger": "DAY-2",
    "description": "Deploy the microservice in a specified environment"
  }
]
```

</details>

## action运行结构

### `CREATE` 动作触发器

让我们使用以下参数调用一个 `CREATE` 自助服务操作: 

```json showLineNumbers
{
  "name": "my-microservice",
  "region": "eu-west-1"
}
```

通过调用自助服务操作，会发送以下操作调用正文: 

```json showLineNumbers
{
  "action": "create_microservice",
  "resourceType": "run",
  "status": "TRIGGERED",
  "trigger": {
    "by": {
      "orgId": "org_7SDeR821bunhS8es",
      "userId": "auth0|638879fa62c686d381b36ecb",
      "user": {
        "email": "test@test.com",
        "firstName": "test",
        "lastName": "test",
        "id": "auth0|638879fa62c686d381b36ecb"
      }
    },
    "origin": "UI",
    "at": "2022-12-07T12:53:52.916Z"
  },
  "context": {
    "entity": null,
    "blueprint": "microservice",
    // highlight-next-line
    "runId": "r_QOz6WoOB1Q2lmhZZ"
  },
  "payload": {
    "entity": null,
    "action": {
      "id": "action_ed2B0O9CbEYkuqvN",
      "identifier": "create_microservice",
      "title": "Create Microservice",
      "userInputs": {
        "properties": {
          "name": { "title": "Service name", "type": "string" },
          "region": { "title": "Cloud Region", "type": "string" }
        }
      },
      "invocationMethod": {
        "url": "https://getport.io",
        "agent": false,
        "type": "WEBHOOK"
      },
      "trigger": "CREATE",
      "description": "Create new microservice",
      "blueprint": "microservice",
      "createdAt": "2022-12-07T09:48:28.659Z",
      "createdBy": "auth0|638879fa62c686d381b36ecb",
      "updatedAt": "2022-12-07T09:48:28.659Z",
      "updatedBy": "auth0|638879fa62c686d381b36ecb"
    },
    "properties": { "name": "my-microservice", "region": "eu-west-1" }
  }
}
```

请注意，调用的自助服务操作的 `runId` 是:  `r_QOz6WoOB1Q2lmhZZ`。

#### 与运行互动

<Tabs groupId="interact" queryString="interact">

<TabItem value="info" label="Run info">

向 `https://api.getport.io/v1/actions/runs/{run_id}` 提出 GET 请求，其中 `run_id=r_QOz6WoOB1Q2lmhZZ` 会得到以下响应: 

```json showLineNumbers
{
  "ok": true,
  "run": {
    "id": "r_QOz6WoOB1Q2lmhZZ",
    "status": "IN_PROGRESS",
    "blueprint": {
      "identifier": "microservice",
      "title": "Service"
    },
    "action": "create_microservice",
    "endedAt": null,
    "source": "UI",
    "relatedEntityExists": false,
    "relatedBlueprintExists": true,
    "properties": {
      "name": "my-microservice",
      "region": "eu-west-1"
    },
    "createdAt": "2022-12-07T12:53:52.916Z",
    "updatedAt": "2022-12-07T12:53:52.916Z",
    "createdBy": "auth0|638879fa62c686d381b36ecb",
    "updatedBy": "auth0|638879fa62c686d381b36ecb"
  }
}
```

:::info 在操作运行对象中，请注意以下几点: 

* `status` - 动作的当前状态。调用自助服务操作时，该值会自动设置为 `IN_PROGRESS`，但您可以根据运行进度将其更改为 `SUCCESS` 或 `FAILURE`；
* endedAt` - 显示 "null"，因为操作运行状态是 "IN_PROGRESS"，但当操作运行状态变为 "SUCCESS "或 "FAILURE "时，它将自动更新。

:::

</TabItem>

<TabItem value="logs" label="Run logs">

向 `https://api.getport.io/v1/actions/runs/{run_id}/logging` 提出 GET 请求，其中 `run_id=r_QOz6WoOB1Q2lmhZZ` 会得到以下响应: 

```json showLineNumbers
{
  "ok": true,
  "runLogs": []
}
```

</TabItem>

</Tabs>

### `DAY-2` action触发器

DAY-2 "自助服务操作的操作运行与 "创建 "自助服务操作的操作运行非常相似，主要区别在于操作运行对象中也提供了调用操作的实体。

例如，使用以下参数简单调用 "DAY-2 "自助服务操作后: 

```json showLineNumbers
{
  "environment": "production"
}
```

将发送以下操作调用正文(现有实体已突出显示): 

```json showLineNumbers
{
  "payload": {
    "action": "deploy_microservice",
    "resourceType": "run",
    "status": "TRIGGERED",
    "trigger": {
      "by": {
        "orgId": "org_7SDeR821bunhS8es",
        "userId": "auth0|638879fa62c686d381b36ecb",
        "user": {
          "email": "test@test.com",
          "firstName": "test",
          "lastName": "test",
          "id": "auth0|638879fa62c686d381b36ecb"
        }
      },
      "origin": "UI",
      "at": "2022-12-08T10:07:09.886Z"
    },
    "context": {
      "entity": "my-microservice",
      "blueprint": "microservice",
      // highlight-next-line
      "runId": "r_z0nJYJv0wCm2ASTR"
    },
    "payload": {
      // highlight-start
      "entity": {
        "identifier": "my-microservice",
        "title": "my-microservice",
        "icon": null,
        "blueprint": "microservice",
        "properties": {
          "region": "eu-west-1"
        },

        "relations": {},
        "createdAt": "2022-12-07T15:25:28.677Z",
        "createdBy": "KZ5zDPudPshQMShUb4cLopBEE1fNSJGE",
        "updatedAt": "2022-12-07T15:30:24.660Z",
        "updatedBy": "KZ5zDPudPshQMShUb4cLopBEE1fNSJGE"
      },
      // highlight-end
      "action": {
        "id": "action_AOLZfMmE3YUeBlMt",
        "identifier": "deploy_microservice",
        "title": "Deploy Microservice",
        "userInputs": {
          "properties": {
            "environment": {
              "title": "Environment",
              "type": "string"
            }
          }
        },
        "invocationMethod": {
          "url": "https://getport.io",
          "agent": false,
          "type": "WEBHOOK"
        },
        "trigger": "DAY-2",
        "description": "Deploy the microservice in a specified environment",
        "blueprint": "microservice",
        "createdAt": "2022-12-08T10:05:54.935Z",
        "createdBy": "auth0|638879fa62c686d381b36ecb",
        "updatedAt": "2022-12-08T10:05:54.935Z",
        "updatedBy": "auth0|638879fa62c686d381b36ecb"
      },
      "properties": {
        "environment": "production"
      }
    }
  }
}
```

请注意，调用的自助服务操作的 `runId` 是:  `r_z0nJYJv0wCm2ASTR`。

#### 与运行互动

<Tabs groupId="interact" queryString="interact">

<TabItem value="info" label="Run info">

向 `https://api.getport.io/v1/actions/runs/{run_id}` 提出 `GET` 请求，其中 `run_id=r_z0nJYJv0wCm2ASTR` 会收到以下响应: 

```json showLineNumbers
{
  "ok": true,
  "run": {
    "id": "r_z0nJYJv0wCm2ASTR",
    "status": "IN_PROGRESS",
    "blueprint": {
      "identifier": "microservice",
      "title": "Service"
    },
    "entity": {
      "identifier": "my-microservice",
      "title": "my-microservice"
    },
    "action": "deploy_microservice",
    "endedAt": null,
    "source": "UI",
    "relatedEntityExists": true,
    "relatedBlueprintExists": true,
    "properties": {
      "environment": "production"
    },
    "createdAt": "2022-12-08T10:07:09.860Z",
    "updatedAt": "2022-12-08T10:07:09.860Z",
    "createdBy": "auth0|638879fa62c686d381b36ecb",
    "updatedBy": "auth0|638879fa62c686d381b36ecb"
  }
}
```

</TabItem>

<TabItem value="logs" label="Run Logs">

向`https://api.getport.io/v1/actions/runs/{run_id}/logging`(其中`run_id=r_z0nJYJv0wCm2ASTR`)发出`GET`请求，会收到以下响应: 

```json showLineNumbers
{
  "ok": true,
  "runLogs": []
}
```

</TabItem>

</Tabs>

## 更新action运行

:::info  Github 后端 使用 "Github 工作流 "作为动作后端时，将提供 "报告工作流状态 "选项，默认设置为 "是"。 使用该选项时，Port 将根据 Github 工作流的结果自动将动作运行状态更新为 "SUCCESS "或 "FAILURE"，因此无需手动更新。

:::

现在，让我们运行并更新一个操作。 可以执行以下更新: 

<Tabs groupId="interact" queryString="interact">

<TabItem value="info" label="Run info">

通过向 `https://api.getport.io/v1/actions/runs/{run_id}` 端点发送 `PATCH` 请求，可以更新运行的状态或链接列表。

不同的更新选项包括

* 通过`status`键设置作业运行状态--`SUCCESS`, `FAILURE`；
* 通过 `link` 键添加作业运行的外部 logging 链接 - AWS Cloudwatch logs、Github Workflow 作业、Jenkins 作业等。

:::tip 您不必在一个请求中提供所有不同的更新，您可以根据需要多次向端点发出 `PATCH` 请求，直到动作运行结束。

请注意，每次补丁请求都会覆盖特定键之前的信息。 例如，当多次更新 `link` 键时，只有最新更新中提供的值才会显示在动作运行对象上。

:::

让我们用下面的 `PATCH` 请求正文更新我们的操作运行: 

```json showLineNumbers
{
  "status": "SUCCESS",
  "link": [
    "https://github.com/actions/toolkit/actions/runs/3617893813",
    "https://github.com/actions/toolkit/actions/runs/4165617487"
  ],
  "message": {
    "run_status": "Run completed successfully!"
  }
}
```

API 返回以下响应: 

```json showLineNumbers
{
  "ok": true,
  "run": {
    "id": "r_QOz6WoOB1Q2lmhZZ",
    "status": "SUCCESS",
    "blueprint": {
      "identifier": "microservice",
      "title": "Service"
    },
    "action": "create_microservice",
    // highlight-next-line
    "endedAt": "2022-12-07T14:51:52.796Z",
    "source": "UI",
    // highlight-start
    "link": [
      "https://github.com/actions/toolkit/actions/runs/3617893813",
      "https://github.com/actions/toolkit/actions/runs/4165617487"
    ],
    "message": {
      "run_status": "Run completed successfully!"
    },
    // highlight-end
    "relatedEntityExists": false,
    "relatedBlueprintExists": true,
    "properties": {
      "name": "my-microservice",
      "region": "eu-west-1"
    },
    "createdAt": "2022-12-07T12:53:52.916Z",
    "updatedAt": "2022-12-07T14:51:52.796Z",
    "createdBy": "auth0|638879fa62c686d381b36ecb",
    "updatedBy": "KZ5zDPudPshQMShUb4cLopBEE1fNSJGE"
  }
}
```

:::info 请注意我们的action运行是如何更新的: 

* `status` - 已更新为 `SUCCESS`；
* `endedAt` - 现在能正确显示操作运行的更新时间；
* `link` - 现在包括我们提供的链接，这些链接也将出现在与 Port 中的操作运行相匹配的页面中；
* `message` - 现在包括我们提供的附加信息，这些信息也将显示在与 Port 中运行的操作相匹配的页面中。

:::

</TabItem>

<TabItem value="logs" label="Run logs">

通过向 `https://api.getport.io/v1/actions/runs/{run_id}/logs` 端点发送 `POST` 请求，可以在运行日志中添加新的日志信息。

不同的更新选项包括

* 通过 `terminationStatus` 键设置运行状态--`SUCCESS`, `FAILURE`；
* 在运行日志中添加额外的日志条目。

让我们用下面的 `POST` 请求体更新我们的操作运行日志: 

```json showLineNumbers
{
  "message": "my new log message"
}
```

API 返回以下响应: 

```json showLineNumbers
{
  "ok": true,
  "runLog": {
    "id": "log_Wo7cIcCftqhj4lNy",
    "runId": "r_z0nJYJv0wCm2ASTR",
    "message": "my new log message",
    "createdAt": "2023-03-12T15:27:25.394Z",
    "createdBy": "KZ5zDPudPshQMShUb4cLopBEE1fNSJGE"
  }
}
```

如果我们向 `https://api.getport.io/v1/actions/runs/{run_id}/logs` 端点发送 `GET` 请求，就会返回整个操作的运行日志: 

```json showLineNumbers
{
  "ok": true,
  "runLogs": [
    {
      "id": "log_Wo7cIcCftqhj4lNy",
      "runId": "r_z0nJYJv0wCm2ASTR",
      "message": "my new log message",
      "createdAt": "2023-03-12T15:27:25.394Z",
      "createdBy": "KZ5zDPudPshQMShUb4cLopBEE1fNSJGE"
    }
  ]
}
```

如果我们想添加最后的 logging 条目，同时将操作运行标记为成功，可以使用下面的请求体: 

```json showLineNumbers
{
  "message": "my new log message with final status",
  "terminationStatus": "SUCCESS"
}
```

带有 `terminationStatus` 键的日志信息只能为一个操作运行发送一次。 `terminationStatus` 发送后，运行状态会被自标记，且不能再修改该运行。

</TabItem>

</Tabs>

## 将实体绑定到action运行中

您还可以通过在每个创建或更改实体的 API 路由(即对 `https://api.getport.io/v1/blueprints/{blueprint_id}/entities/{entity_id}` 路由的 `POST`、`PUT`、`PATCH` 和 `DELETE` 请求)中附加 `run_id` 查询参数，为动作运行添加额外的上下文和元数据。通过添加 `run_id` 参数，您可以将对实体所做的更改反映为动作运行在其运行期间所执行的步骤集的一部分。

:::tip 只有当action运行处于 "IN_PROGRESS "状态时，才能将实体与action运行绑定。

:::

例如，让我们调用另一个动作运行，并使用下面的 python 代码段创建一个新的微服务实体，该实体与我们触发的自助服务动作相匹配，并添加 `run_id` 查询参数来标记自助服务动作负责创建新的微服务: 

<details>
<summary>Click here to see the Python code</summary>

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests
from pprint import pprint
import requests

CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'

API_URL = 'https://api.getport.io/v1'

RUN_ID = 'YOUR_RUN_ID'

TARGET_BLUEPRINT_ID = 'microservice'

def get_auth_token():
    credentials = {'clientId': CLIENT_ID, 'clientSecret': CLIENT_SECRET}
    token_response = requests.post(f'{API_URL}/auth/access_token', json=credentials)
    return token_response.json()['accessToken']

def get_run_id(access_token, run_id):
    headers = {
        'Authorization': f'Bearer {access_token}'
    }

    run_id_resp = requests.get(f'{API_URL}/actions/runs/{run_id}', headers=headers)
    return run_id_resp.json()['run']

def create_entity(access_token, run_id, properties):
    headers = {
        'Authorization': f'Bearer {access_token}'
    }

    query = {
        "run_id": run_id,
        "upsert": "true"
    }

    body = {
        "identifier": f"{properties['name'].replace(' ','_')}",
        "title": f"{properties['name']}",
        "properties": {
            "region": f"{properties['region']}"
        }
    }

    entity_resp = requests.post(f"{API_URL}/blueprints/{TARGET_BLUEPRINT_ID}/entities", headers=headers, params=query, json=body)
    pprint(entity_resp.json()['entity'])
    return entity_resp.json()['entity']['identifier']

def add_action_run_log_entry(access_token, run_id, message):
    headers = {
        'Authorization': f'Bearer {access_token}'
    }

    body = {
        "message": message
    }

    action_update_resp = requests.post(f'{API_URL}/actions/runs/{run_id}/logs', headers=headers, json=body)

def mark_action_run_as_successful(access_token, run_id):
    headers = {
        'Authorization': f'Bearer {access_token}'
    }

    body = {
        "status": "SUCCESS",
        "link": ["https://github.com/actions/toolkit/actions/runs/3617893813"]
    }

    action_update_resp = requests.patch(f'{API_URL}/actions/runs/{run_id}', headers=headers, json=body)
    pprint(action_update_resp.json()['run'])

def main():
    access_token = get_auth_token()

    run = get_run_id(access_token, RUN_ID)
    props = run['properties']
    entity_id = create_entity(access_token, RUN_ID, props)
    add_action_run_log_entry(access_token, RUN_ID, f'new entity: {entity_id}')
    add_action_run_log_entry(access_token, RUN_ID, f'New entity created!')
    mark_action_run_as_successful(access_token, RUN_ID)

if __name__ == '__main__':
    main()
```

</details>

现在，当你查看动作运行的 run logging 时，就会看到新创建实体的信息: 

![Developer portal action run log](../../../static/img/self-service-actions/action_run_log.png)

:::tip 在上面的示例中，我们只创建了一个实体，但作为单个操作运行步骤的一部分，可以创建、更新或删除多个实体，所有这些更改都将反映在操作运行日志中。

:::
