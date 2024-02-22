import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 控制有效载荷

您可能希望集成的某些第三方应用程序可能不接受从 Port 自助服务操作传入的原始有效载荷。 Port 代理允许您控制发送到每个第三方应用程序的有效载荷。

您可以在部署 Port-agent 容器时提供有效载荷映射配置文件，从而更改发送到第三方应用程序的请求。

### 设置映射

设置映射取决于安装代理的方式。

<Tabs groupId="installationMethod" queryString defaultValue="helm" values={[
  {label: "Helm", value: "helm"},
  {label: "Argo", value:"argo"}
]}>

<TabItem value="helm">

为了向 Provider 提供映射配置，请再次运行安装命令，并添加以下参数: 

```bash showLineNumbers
--set-file controlThePayloadConfig=/PATH/TO/LOCAL/FILE.yml
```

</TabItem>

<TabItem value="argo">

为了向 Provider 提供映射，请将映射添加到安装[here](https://docs.getport.io/create-self-service-experiences/setup-backend/webhook/port-execution-agent/installation-methods/argocd#installation) 中创建的 `values.yaml` 文件中。需要将其添加为顶层字段。

下面是被引用的默认映射: 

```yaml showLineNumbers
controlThePayloadConfig: |
  [
    {
    "enabled": true,
    "url": ".payload.action.invocationMethod.url",
    "method": ".payload.action.invocationMethod.method // \"POST\""
    }
  ]
```

</TabItem>
</Tabs>

### 控制有效载荷映射

有效载荷映射文件是一个包含映射列表的 json 文件，每个映射都包含将被覆盖并发送到第三方应用程序的请求字段。

下面的示例展示了如何针对各种常见用例使用不同的映射配置来部署 Port 代理。

每个映射字段都可由 JQ 表达式构建，JQ 表达式将根据从 Port 发送给Port代理的原始有效载荷进行评估，评估结果将发送给第三方应用程序。

下面是映射文件模式: 

```showLineNumbers
[ # Can have multiple mappings. Will use the first one it will find with enabled = True (Allows you to apply mapping over multiple actions at once)
  {
      "enabled": bool || JQ,
      "url": JQ, # Optional. default is the incoming url from port
      "method": JQ, # Optional. default is POST. Should return one of the following string values POST / PUT / DELETE / GET
      "headers": dict[str, JQ], # Optional. default is {}
      "body": ".body", # Optional. default is the whole payload incoming from Port.
      "query": dict[str, JQ] # Optional. default is {},
      "report" { # Optional. Used to report the run status back to Port right after the request is sent to the 3rd party application
        "status": JQ, # Optional. Should return the wanted runs status
        "link": JQ, # Optional. Should return the wanted link or a list of links
        "summary": JQ, # Optional. Should return the wanted summary
        "externalRunId": JQ # Optional. Should return the wanted external run id
      }
  }
]
```

*** 主体可由 json 部分构建如下: **

```json showLineNumbers
{
  "body": {
    "key": 2,
    "key2": {
      "key3": ".im.a.jq.expression",
      "key4": "\"im a string\""
    }
  }
}
```

### 映射示例

下面是一些映射示例，说明如何使用 JQ 和从 Port 发送的操作有效载荷来更改代理向目标端点发送的有效载荷。 在每个映射中，我们将显示相关字段。

#### 应用映射过滤器

假设你的操作有几种不同的调用方法，你可以创建一个映射配置，只应用于特定类型的操作。

例如，要创建一个过滤器，只应用于使用 `GitLab` 方法的操作: 

```text showLineNumbers
"enabled": ".payload.invocationMethod.type == \"GITLAB\""
```

#### 根据属性创建 URL

假设配置了一个 `webhook` 调用操作，将请求转发到 URL `http://test.com/`，并且 Port 中的操作包含一个名为 `network_port` 的 `number` 类型输入，用于指定发送请求的网络Port，下面是如何使用 URL 和附加输入构建完整的 URL: 

```text showLineNumbers
"url": ".payload.invocationMethod.url + .payload.properties.network_port"
```

在属性 `network_port` 中输入 8080 后，调用该操作将导致代理向 `http://test.com/8080` 发送 webhook 请求。

###要以映射为基础的接收信息

```json showLineNumbers
{
  "action": "action_identifier",
  "resourceType": "run",
  "status": "TRIGGERED",
  "trigger": {
    "by": {
      "orgId": "org_XXX",
      "userId": "auth0|XXXXX",
      "user": {
        "email": "executor@mail.com",
        "firstName": "user",
        "lastName": "userLastName",
        "phoneNumber": "0909090909090909",
        "picture": "https://s.gravatar.com/avatar/dd1cf547c8b950ce6966c050234ac997?s=480&r=pg&d=https%3A%2F%2Fcdn.auth0.com%2Favatars%2Fga.png",
        "providers": ["port"],
        "status": "ACTIVE",
        "id": "auth0|XXXXX",
        "createdAt": "2022-12-08T16:34:20.735Z",
        "updatedAt": "2023-11-13T15:11:38.243Z"
      }
    },
    "origin": "UI",
    "at": "2023-11-13T15:20:16.641Z"
  },
  "context": {
    "entity": "e_iQfaF14FJln6GVBn",
    "blueprint": "kubecostCloudAllocation",
    "runId": "r_HardNzG6kzc9vWOQ"
  },
  "payload": {
    "entity": {
      "identifier": "e_iQfaF14FJln6GVBn",
      "title": "myEntity",
      "icon": "Port",
      "blueprint": "myBlueprint",
      "team": [],
      "properties": {},
      "relations": {},
      "createdAt": "2023-11-13T15:24:46.880Z",
      "createdBy": "auth0|XXXXX",
      "updatedAt": "2023-11-13T15:24:46.880Z",
      "updatedBy": "auth0|XXXXX"
    },
    "action": {
      "invocationMethod": {
        "type": "WEBHOOK",
        "agent": true,
        "synchronized": false,
        "method": "POST",
        "url": "https://myGitlabHost.com"
      },
      "trigger": "DAY-2"
    },
    "properties": {},
    "censoredProperties": []
  }
}
```

## 向 Port 报告action状态

代理可使用映射中的 "report "字段向 Port 报告操作状态。

向第三方应用程序发出请求后，报告请求将立即发送到 Port API，并更新 Port 中的运行状态。

代理被引用 "报告 "字段中的 JQ 来构建报告请求正文。

可用字段有

* `status` - 运行状态。可以是以下 Values 之一: 成功/失败
* `link` - 指向第三方应用程序中运行的链接。可以是字符串或字符串列表。
* `summary` - 运行的字符串摘要。
* `externalRunId` - 第三方应用程序中的外部运行 ID。外部运行 ID 用于被引用，以便通过外部运行 ID 搜索 Port 中的运行。

报告映射可被引用以下字段: 

`.body` - 如[above](#the-incoming-message-to-base-your-mapping-on)所述的传入信息 `.request` - 使用控制有效载荷映射计算并发送给第三方应用程序的请求 `.response` - 从第三方应用程序收到的响应

响应 "字段包含以下字段: 

* `statusCode` - 响应的状态代码
* `json` - 以 json 对象形式显示的响应正文
* `text` - 字符串形式的响应正文
* `headers` - json 对象的响应标题
