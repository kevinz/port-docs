---

sidebar_position: 1
description: 使用 webhook 保持拉取请求更新

---

import PullRequestBlueprint from '../resources/github/_example_github_pr_blueprint.mdx'
import PullRequestWebhookConfig from '../resources/github/_example_github_pr_configuration.mdx'

# GitHub

在本示例中，您将在[GitHub](https://github.com) 和 Port 之间创建一个 webhook 集成，用于接收拉取请求实体。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Pull request blueprint</summary>

<PullRequestBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints) : 

<details>
<summary>Pull request webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: "拉取请求映射器"；
    2.标识符: `pull_request_mapper`；
    3.Description : `来自 GitHub 的 pull-request 事件的 webhook 配置；
    4.图标 : `Github`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <PullRequestWebhookConfig/>
3.向下滚动到 **高级设置**，输入以下详细信息: 
    1.Secret: `WEBHOOK_SECRET`；
    2.签名头名称:  `X-Hub-Signature-256`；
    3.签名算法: 从下拉选项中选择 `sha256`；
    4.签名前缀 : `sha256=`；
    5.请求标识符路径 : `.headers.\"X-GitHub-Delivery\"`；
    6.点击页面底部的**保存**。

</details>

## 在 GitHub 中创建 webhook

1. 转到 GitHub 上所需的组织/仓库；
2. 选择 **设置**；
3. 选择 **Webhooks**；
4. 点击 **添加 webhook**；
5. 输入以下详细信息: 
    1. Payload URL` - 输入您在创建 webhook 配置后收到的 `url` 键的值；
    2. 内容类型` - `应用程序/json`；
    3. `Secret` - 输入您在创建 webhook 时指定的 secret 值；
    4.在 "您希望哪些事件触发此 webhook？ - 选择 "让我选择单个事件 "并选择 **拉取请求**；
    5.确保选中 "激活 "复选框。
6.点击**添加网络钩子**

:::tip 为了查看 GitHub webhooks 中可用的不同有效载荷和事件、[look here](https://docs.github.com/en/webhooks-and-events/webhooks/webhook-events-and-payloads)

:::

完成！您对拉取请求所做的任何更改(打开、关闭、编辑等)都会触发 webhook 事件，GitHub 会将该事件发送到 Port 提供的 webhook URL。 Port 会根据映射解析事件，并相应地更新目录实体。

## 让我们来测试一下

本节包括创建拉取请求时从 GitHub 发送的 webhook 事件示例。 此外，还包括根据上一节提供的 webhook 配置从事件中创建的实体。

### 有效载荷

下面是打开 GitHub 拉取请求时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Webhook event payload</summary>

```json showLineNumbers
{
  "action": "opened",
  "number": 15,
  "pull_request": {
    "url": "https://api.github.com/repos/username/My_Awesome_Python_App/pulls/15",
    "id": 1395792349,
    "node_id": "PR_kwDOEFWVvs5TMhnd",
    "html_url": "https://github.com/username/My_Awesome_Python_App/pull/15",
    "issue_url": "https://api.github.com/repos/username/My_Awesome_Python_App/issues/15",
    "number": 15,
    "state": "open",
    "locked": false,
    "title": "Update regex",
    "user": {
      "login": "username",
      "id": 15999660,
      "node_id": "MDQ6VXNlcjE1OTk5NjYw",
      "url": "https://api.github.com/users/username"
    },
    "body": "Modifying event header",
    "created_at": "2023-06-16T14:08:27Z",
    "updated_at": "2023-06-16T14:08:27Z",
    "closed_at": "None",
    "merged_at": "None",
    "assignees": [],
    "requested_reviewers": [],
    "requested_teams": [],
    "labels": [],
    "commits_url": "https://api.github.com/repos/username/My_Awesome_Python_App/pulls/15/commits",
    "head": {
      "label": "username:port",
      "ref": "port",
      "sha": "9bd151d8a6d6c3759e7fbdb5ba5ed82668021e77",
      "user": {
        "login": "username",
        "id": 15999660,
        "node_id": "MDQ6VXNlcjE1OTk5NjYw",
        "avatar_url": "https://avatars.githubusercontent.com/u/15999660?v=4",
        "html_url": "https://github.com/username",
        "type": "User"
      },
      "repo": {
        "id": 274044350,
        "node_id": "MDEwOlJlcG9zaXRvcnkyNzQwNDQzNTA=",
        "name": "My_Awesome_Python_App",
        "full_name": "username/My_Awesome_Python_App",
        "private": false,
        "owner": {
          "login": "username",
          "id": 15999660,
          "node_id": "MDQ6VXNlcjE1OTk5NjYw",
          "url": "https://api.github.com/users/username"
        },
        "html_url": "https://github.com/username/My_Awesome_Python_App",
        "description": "Repo description",
        "fork": false,
        "visibility": "public",
        "forks": 0,
        "open_issues": 11,
        "watchers": 1,
        "default_branch": "master"
      }
    }
  }
}
```

</details>

#### 映射结果

结合示例有效载荷和 webhook 配置可生成以下 Port 实体: 

```json showLineNumbers
{
  "identifier": "1395792349",
  "title": "Update regex",
  "blueprint": "pullRequest",
  "properties": {
    "author": "username",
    "url": "https://github.com/username/My_Awesome_Python_App/pull/15"
  },
  "relations": {}
}
```