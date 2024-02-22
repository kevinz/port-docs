---

sidebar_position: 20
description: 将 CodeCov 覆盖范围纳入您的目录

---

import CodecovCoverageBlueprint from "./resources/codecov/_example_coverage_blueprint.mdx";
import CodecovWebhookConfiguration from "./resources/codecov/_example_webhook_config.mdx";
import CodecovPythonScript from "./resources/codecov/_example_codecov_python_script.mdx";

# Codecov

在本示例中，您将在[Codecov](https://docs.codecov.com/docs/quick-start) 和 Port 之间创建一个 webhook 集成。该集成将有助于将覆盖实体导入 Port。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Codecov coverage blueprint</summary>

<CodecovCoverageBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints) : 

<details>
<summary>Codecov webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Codecov Mapper`；
    2.标识符 : `codecov_mapper`；
    3.Description : `将 Codecov 覆盖范围映射到 Port` 的 webhook 配置；
    4.图标 : `Git`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <CodecovWebhookConfiguration/>
     ::注意 Webhook URL
     注意并复制此选项卡中提供的 Webhook URL
     :::
3.点击页面底部的**保存**。

</details>

## 在 Codecov 中创建 webhook

1. 从您的 Codecov 账户，打开**设置**；
2. 点击左侧边栏菜单中的**Global YAML**选项卡；
3. 在 YAML 编辑器中添加以下 Codecov 配置，以便在代码库中发生事件时随时通知 Port: 


    ```yaml
    coverage:
    notify:
        webhook:
        default:
            only_pulls: false
            url: YOUR_PORT_WEBHOOK
    ```

    :::note Webhook URL replacement
    Remember to replace `YOUR_PORT_WEBOOK` with the value of the `URL` you received after creating the webhook configuration in Port.
    :::

```
:::tip notification service customization
For more information on customizing the notification service, follow this [guide](https://docs.codecov.com/docs/notifications#standard-notification-fields)
:::
```

3.单击**保存更改**保存 webhook 配置。

有关自定义通知服务的更多信息，请参阅[this documentation](https://docs.codecov.com/docs/notifications#standard-notification-fields)

一切就绪！当您的 Codecov 账户发生任何变化时，Port 提供的 URL 将触发一个 Webhook 事件。 Port 将根据映射解析事件，随后更新目录实体。

## 导入历史 Codecov 覆盖率

在本示例中，您将使用 Provider 提供的 Python 脚本从 Codecov REST API 获取覆盖率数据并将其引用到 Port。

### 先决条件

本示例使用的是上一节中的[blueprint and webhook](#port-configuration) 定义。

此外，请 Provider 下列环境变量: 

* `PORT_CLIENT_ID` - 您的 Port 客户端 ID
* `PORT_CLIENT_SECRET` - 您的 Port 客户端secret
* `CODECOV_TOKEN` - Codecov API 访问令牌
* `CODECOV_SERVICE_PROVIDER` - Git 托管服务提供商。可接受的 Values 包括 `github`、`github_enterprise`、`bitbucket`、`bitbucket_server`、`gitlab` 和 `gitlab_enterprise
* `CODECOV_SERVICE_PROVIDER_ACCOUNT_NAME` - Git 服务提供商的用户名

:::info  凭据 使用此参数查找您的 Port 凭据[guide](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)

使用以下命令查找您的 Codecov API 令牌[guide](https://docs.codecov.com/reference/overview)

:::

使用以下 Python 脚本将 Codecov 历史覆盖率被引用到 Port 中: 

<details>
<summary>Codecov Python script</summary>

<CodecovPythonScript/>

</details>

### 运行 python 脚本

要从您的 Codecov 账户向 Port 输入覆盖数据，请运行以下命令: 

```bash
export PORT_CLIENT_ID=<ENTER CLIENT ID>
export PORT_CLIENT_SECRET=<ENTER CLIENT SECRET>
export CODECOV_TOKEN=<ENTER CODECOV TOKEN>
export CODECOV_SERVICE_PROVIDER=<ENTER CODECOV SERVICE PROVIDER>
export CODECOV_SERVICE_PROVIDER_ACCOUNT_NAME=<ENTER CODECOV SERVICE PROVIDER ACCOUNT NAME>

git clone https://github.com/port-labs/example-codecov-test-coverage.git

cd example-codecov-test-coverage

pip install -r ./requirements.txt

python app.py
```

:::tip  Python 脚本信息 查找有关 python 脚本的更多信息[here](https://github.com/port-labs/example-codecov-test-coverage)

:::

完成！现在您可以将历史覆盖范围从 Codecov 导入 Port。 Port 将根据映射解析对象，并相应地更新目录实体。