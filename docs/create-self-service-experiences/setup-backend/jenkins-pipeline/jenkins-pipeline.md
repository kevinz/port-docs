import Image from "@theme/IdealImage";
import DefineVars from "../../../../static/img/self-service-actions/setup-backend/jenkins-pipeline/define-variables.png";

# 使用 webhooks 触发 Jenkins

在本指南中，您将学习如何使用[Webhook Actions](../webhook/) 从 Port 触发您的[Jenkins](https://www.jenkins.io/) Pipelines。

![Illustration](../../../../static/img/self-service-actions/setup-backend/jenkins-pipeline/jenkins-illustration.png)

上图所示步骤如下: 

1. 在 Port 中调用一个操作；
2. Port 使用 SHA-1 对操作有效载荷进行签名，签名值为[`clientSecret`](../../../build-your-software-catalog/sync-data-to-catalog/api/api.md#find-your-port-credentials) ，并将其放入 `X-Port-Signature` 请求头中。
    信息
    使用请求标头验证 webhook 请求有以下好处: - 确保请求有效载荷未被篡改
    - 确保信息发送方是 Port
    - 确保收到的信息不是旧信息的重放
    :::
3.Port 通过向 `https://{JENKINS_URL}/generic-webhook-trigger/invoke` 发送 `POST` 请求来发布调用的 `WEBHOOK` 消息
    示例流程如下: 1.开发人员要求运行 Jenkins 管道；
    2.Port 向 Jenkins webhook `URL` 发送带有动作有效载荷的 `POST` 请求；
    3.Jenkins webhook 接收新的操作请求；
    4.Jenkins webhook 触发 Pipelines；

## 先决条件

Jenkins 所需插件

* [Generic webhook trigger](https://plugins.jenkins.io/generic-webhook-trigger/) - 允许使用 webhook 调用触发 Jenkins Pipelines。

## 设置 webhook

### 为 Pipelines 启用 webhook 触发器

要启用使用 webhook 引用触发 Jenkins 管道，需要在管道中添加 "通用 Webhook 触发器 "作为构建触发器。

在任务页面中，进入 "**配置**"选项卡，向下滚动到 "**构建触发器**"部分。 选中 "通用 Webhook 触发器 "复选框: 

![Enable generic webhook](../../../../static/img/self-service-actions/setup-backend/jenkins-pipeline/check-generic-webhook-option.png)

默认情况下，为作业启用 webhook 触发器时，可通过向 `http://JENKINS_URL/generic-webhook-trigger/invoke` 发送事件来触发作业。这意味着，如果未另行配置，向该路由发送事件时将触发所有作业。建议设置[job token](jenkins-pipeline.md#token-setup) ，以避免运行不需要的作业。

### 定义变量

选中复选框后，请查看 ** Pipelines 内容参数** 部分。 在这里您将定义传递给 Pipelines 运行的变量。

* Varues "字段的值应与作业配置中定义的、作业运行所期望的变量名称相匹配。
* 表达式 "字段应设置为 "JSONPath"，并指向 Port 操作发送的相关属性。

<br/>

<Image img={DefineVars} style={{ width: 550 }} />

<br/>

:::tip 下面是 Port 操作的部分 JSON 方案，其中显示了 Port 在触发操作时发送的输入: 

```json showLineNumber
[
  {
    "identifier": "runPipeline",
    "title": "Run Pipeline",
    "icon": "Jenkins",
    "userInputs": {
      "properties": {
        "input1": {
          "type": "string"
        }
      }
    }
    ... # Port Action configuration
]
```

以下是触发操作并发送到 Jenkins 时生成的有效载荷示例: 

```json showLineNumber
{
    ... # Event metadata
    "payload": {
        "properties": {
            "input1": "input1_value"
        }

    }

}
```

例如，"输入 1 "的 JSONPath 应为

```text
$.payload.properties.input1
```

**Port操作** - 完整的Port操作定义可在[here](./jenkins-pipeline.md#setting-up-the-port-action) 上找到。

:::

### 令牌设置

[token parameter](https://plugins.jenkins.io/generic-webhook-trigger/#plugin-content-token-parameter) 允许触发特定(或一组) 作业。

要为任务设置令牌，请向下滚动到**令牌**部分，然后提供任务令牌: 

![Configure Token](../../../../static/img/self-service-actions/setup-backend/jenkins-pipeline/configure-token.png)

保存后，您就可以通过以下 URL 具体触发该工作任务: 

```text showLineNumbers
http://JENKINS_URL/generic-webhook-trigger/invoke?token=<JOB_TOKEN>
```

:::tip 有关 Jenkins webhook 触发器的高级配置，请查看[Generic webhook trigger](https://plugins.jenkins.io/generic-webhook-trigger/) 插件页面！

:::

### 设置 Port 操作

要触发 Jenkins 管道，您需要设置一个 Port[Webhook Action](../webhook/) 。

下面是一个触发 webhook 的操作示例: 

```json showLineNumbers
[
  {
    "identifier": "runPipeline",
    "title": "Run Pipeline",
    "icon": "Jenkins",
    "userInputs": {
      "properties": {
        "input1": {
          "type": "string"
        }
      }
    },
    # highlight-start
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "http://JENKINS_URL/generic-webhook-trigger/invoke?token=<JOB_TOKEN>"
    },
    # highlight-end
    "trigger": "CREATE",
    "description": "Run Jenkins pipeline"
  }
]
```

### 确保 webhook 的安全

可以通过[configuring whitelisting](https://plugins.jenkins.io/generic-webhook-trigger/#plugin-content-whitelist-hosts) 为暴露在外的 Pipelines 添加保护层。

白名单为您提供了以下安全选项: 

* 限制可发送触发 Pipelines 请求的 IP 地址列表；
* 为 webhook 有效负载内容添加[validation](../webhook/signature-verification.md) ，以验证其是否真正来自 Port。

下面是所需配置的示例: 

![Webhook Validation](../../../../static/img/self-service-actions/setup-backend/jenkins-pipeline/validate-webhook.png)

:::note 

* IP 字段应设置为 "3.251.12.205"，这是我们托管的外呼 WEBHOOK Gateway；
    - 有关 Port 外呼的更多信息，请查看 Port 的[actions security](../../security/security.md) 页面。
* 在 **HMAC Secret** 字段中，选择一个包含您的 `port-client-secret` 的 secret。

如果该secret还不存在，请使用[this guide](https://www.jenkins.io/doc/book/using/using-credentials/) 创建一个 "secret文本 "类型的secret。该secret的值应该是您的 "Port 客户端secret"，可根据指南[here](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials) 找到。

:::

#### 向 Port 报告 Jenkins 操作运行状态

成功触发 Jenkins 管道后，必须在 Port 中更新运行操作的状态。

为了更新操作，需要创建 `RUN_ID` 变量，并将其设置为从[action payload](../../self-service-actions-deep-dive/self-service-actions-deep-dive.md#action-message-structure) 获取: 

![RUN_ID variable](../../../../static/img/self-service-actions/setup-backend/jenkins-pipeline/jenkins-runid-variable.png)

下面的代码片段演示了如何向 Port 报告 Pipelines 的进度: 

<details>
<summary> Click here to see the code</summary>

```groovy showLineNumbers
import groovy.json.JsonSlurper

pipeline {
    agent any

    environment {
        PORT_CLIENT_ID="YOUR_CLIENT_ID"
        PORT_CLIENT_SECRET="YOUR_CLIENT_SECRET"
        ACCESS_TOKEN = ""
    }

    stages {
        // highlight-next-line
        stage('Get access token') {
            steps {
                script {
                    // Execute the curl command and capture the output
                    def result = sh(returnStdout: true, script: """
                        accessTokenPayload=\$(curl -X POST \
                            -H "Content-Type: application/json" \
                            -d '{"clientId": "${PORT_CLIENT_ID}", "clientSecret": "${PORT_CLIENT_SECRET}"}' \
                            -s "https://api.getport.io/v1/auth/access_token")
                        echo \$accessTokenPayload
                    """)

                    // Parse the JSON response using JsonSlurper
                    def jsonSlurper = new JsonSlurper()
                    def payloadJson = jsonSlurper.parseText(result.trim())

                    // Access the desired data from the payload
                    ACCESS_TOKEN = payloadJson.accessToken
                }
            }
        }

        // highlight-next-line
        stage('Send logs example') {
            steps {
                script {
                    def logs_report_response = sh(script: """
                        curl -X POST \
                            -H "Content-Type: application/json" \
                            -H "Authorization: Bearer ${ACCESS_TOKEN}" \
                            -d '{"message": "this is a log test message example"}' \
                            "https://api.getport.io/v1/actions/runs/$RUN_ID/logs"
                    """, returnStdout: true)

                    println(logs_report_response)
                }
            }
        }

        // highlight-next-line
        stage('Update status example') {
            steps {
                script {
                    def status_report_response = sh(script: """
                        curl -X PATCH \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${ACCESS_TOKEN}" \
                          -d '{"status":"SUCCESS", "message": {"run_status": "Jenkins CI/CD Run completed successfully!"}}' \
                            "https://api.getport.io/v1/actions/runs/${RUN_ID}"
                    """, returnStdout: true)

                    println(status_report_response)
                }
            }
        }
    }
}
```

</details>