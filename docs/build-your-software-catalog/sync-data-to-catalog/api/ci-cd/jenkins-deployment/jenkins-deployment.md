---
sidebar_position: 1
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# Jenkins 部署

:::tip  可用的 Ocean 集成 Port 为 Jenkins 提供了一个[Ocean integration](/build-your-software-catalog/sync-data-to-catalog/ci-cd/jenkins.md) ，可让您自动将 Jenkins 资源与 Port 同步，并提供更多配置选项。 这是**推荐的**将 Port 与 Jenkins 集成的方式。如果您仍希望使用 Port 的 API，请关注此页面。

:::

被用于 Jenkins 构建后，您可以在 Port 中轻松创建/更新和查询实体。

<br></br>
<br></br>

![Github Illustration](/img/build-your-software-catalog/sync-data-to-catalog/jenkins/jenkins-pipeline-illustration.jpg)

## 💡 Jenkins 的常见构建 Usage

例如，Port 的应用程序接口(API)可轻松实现 Port 与 Jenkins 构建的集成: 

* 报告正在运行的**CI任务**的状态；
* 更新软件目录中有关**微服务**新**构建版本的信息；
* 获取现有**实体**。

## 先决条件

1. 本例被用于了以下 Jenkins 插件: 

* [Plain Credentials](https://plugins.jenkins.io/credentials-binding/) (>=143.v1b_df8b_d3b_e48)
* [HTTP Request](https://plugins.jenkins.io/http_request/) (>=1.16)

2.示例中被用于的方法如下，这些签名需要经过批准: 

```
new groovy.json.JsonSlurperClassic
method groovy.json.JsonSlurperClassic parseText java.lang.String
```

3.将您的 `PORT_CLIENT_ID` 和 `PORT_CLIENT_SECRET` 添加为[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/) ，以便在您的 Pipelines 中被用于。
4.确保您的 Port 安装中已有[Blueprint](/build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md) ，以便创建/更新实体。

## 设置

:::tip 本指南中被用于的所有 Port API 路由都可以在 Port 的[API documentation](/api-reference/api-reference.mdx) 中找到。

:::

要在 Jenkins 构建中与 Port 交互，请按照以下步骤操作: 

### 获取您的应用程序接口令牌

下面的代码段向您展示了如何使用 `withCredentials` 将 `PORT_CLIENT_ID` 和 `PORT_CLIENT_SECRET` 凭据传递给联编，它利用 Plain Credentials 插件将凭据绑定到变量。

Provider 提供的代码段还包括将 Port 的 API URL 保存为环境变量，以便在未来阶段使用，并使用凭据向 Port 的 API 发出请求，以获取访问令牌: 

<details>
  <summary> Get API token </summary>

```js showLineNumbers
pipeline {
  agent any
  environment {
    API_URL = "https://api.getport.io"
  }
...
    withCredentials([
        string(credentialsId: 'port-client-id', variable: 'PORT_CLIENT_ID'),
        string(credentialsId: 'port-client-secret', variable: 'PORT_CLIENT_SECRET')
        ]){
            // Token request body
            auth_body = """
                {
                    "clientId": "${PORT_CLIENT_ID}",
                    "clientSecret": "${PORT_CLIENT_SECRET}"
                }
                """

            // Make a request to fetch Port API's token
            token_response = httpRequest contentType: 'APPLICATION_JSON',
                httpMode: "POST",
                requestBody: auth_body,
                url: "${API_URL}/v1/auth/access_token"

            // Parse the response to get the accessToken
            def slurped_response = new JsonSlurperClassic().parseText(token_response.content)
            def token = slurped_response.accessToken // Use this token for authentication with Port
            ...
        }
```

</details>

### 使用 Port 的 API

在 Jenkins 构建中添加以下代码，以创建/更新实体或获取现有实体: 

<Tabs groupId="usage" defaultValue="upsert" values={[
{label: "Create/Update", value: "upsert"},
{label: "Get", value: "get"}
]}>

<TabItem value="upsert">

```js showLineNumbers
import groovy.json.JsonSlurperClassic
...
    auth_body = """
        {
            "clientId": "${PORT_CLIENT_ID}",
            "clientSecret": "${PORT_CLIENT_SECRET}"
        }
        """
    token_response = httpRequest contentType: 'APPLICATION_JSON',
        httpMode: "POST",
        requestBody: auth_body,
        url: "${API_URL}/v1/auth/access_token"
    def slurped_response = new JsonSlurperClassic().parseText(token_response.content)
    def token = slurped_response.accessToken // Port's access token

    entity_body = """
        {
            "identifier": "example-entity",
            "properties": {
              "myStringProp": "My value",
              "myNumberProp": 1,
              "myBooleanProp": true,
              "myArrayProp": ["myVal1", "myVal2"],
              "myObjectProp": {"myKey": "myVal", "myExtraKey": "myExtraVal"}
            }
        }
    """
    // request url : {API_URL}/blueprints/<blueprint_name>/entities/<entity_name>
    response = httpRequest contentType: "APPLICATION_JSON", httpMode: "POST",
            url: "${API_URL}/v1/blueprints/blueprint/entities?upsert=true&merge=true",
            requestBody: entity_body,
            customHeaders: [
                [name: "Authorization", value: "Bearer ${token}"],
            ]
    println(response.content)
```

</TabItem>
<TabItem value="get">

```js showLineNumbers
import groovy.json.JsonSlurperClassic
...
    auth_body = """
        {
            "clientId": "${PORT_CLIENT_ID}",
            "clientSecret": "${PORT_CLIENT_SECRET}"
        }
        """
    token_response = httpRequest contentType: 'APPLICATION_JSON',
        httpMode: "POST",
        requestBody: auth_body,
        url: "${API_URL}/v1/auth/access_token"
    def slurped_response = new JsonSlurperClassic().parseText(token_response.content)
    def token = slurped_response.accessToken // Port's access token

    response = httpRequest contentType: 'APPLICATION_JSON', httpMode: "GET",
            url: "${API_URL}/v1/blueprints/blueprint/entities/entity-example",
            customHeaders: [
                [name: "Authorization", value: "Bearer ${token}"],
            ]
    println(response.content)
```

</TabItem>
</Tabs>

## 示例

有关与 Jenkins 一起使用 Port 的实用示例，请参阅[examples](./examples.md) 页面。