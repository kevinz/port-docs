---

sidebar_position: 2
description: 使用 webhook 保持合并请求的最新状态

---

import MergeRequestBlueprint from '../resources/gitlab/_example_gitlab_mr_blueprint.mdx'
import MergeRequestWebhookConfig from '../resources/gitlab/_example_gitlab_mr_configuration.mdx'

# GitLab

在本示例中，您将在[GitLab](https://about.gitlab.com/) 和 Port 之间创建一个 webhook 集成，用于接收合并请求实体。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Merge request blueprint</summary>

<MergeRequestBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints) : 

<details>
<summary>Pull request webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Gitlab mapper`；
    2.标识符 : `gitlab_mapper`；
    3.Description : `将 Gitlab 合并请求映射到 Port` 的 webhook 配置；
    4.图标 : `Gitlab`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <MergeRequestWebhookConfig/>
3.向下滚动到**高级设置**，输入以下详细信息: 
    1.请求标识符路径: `.headers.X-Gitlab-Event-Uuid`；
    2.点击页面底部的**保存**。

</details>

## 在 GitLab 中创建 webhook

:::tip 可以在组级和项目级创建网络钩子，本例重点介绍项目级网络钩子

:::

1. 在 GitLab 中转入所需的项目；
2. 在页面左侧的侧边栏选择**设置**，然后点击**Webhooks**；
3. 输入以下详细信息: 
    1. `URL` - 输入创建 webhook 配置后收到的 `url` 键值；
    2. 触发器"- 选择 "合并请求事件"；
    3.确保选中 "启用 SSL 验证 "复选框。
4.点击 ** 添加 webhook**

:::tip 为了查看 GitLab webhooks 中的不同有效载荷和事件，请使用以下命令、[look here](https://docs.gitlab.com/ee/user/project/integrations/webhook_events.html)

:::

完成！您对合并请求所做的任何更改(打开、关闭、编辑等)都会触发 webhook 事件，GitLab 会将该事件发送到 Port 提供的 webhook URL。 Port 会根据映射解析事件，并相应地更新目录实体。

## 让我们来测试一下

本节包括创建合并请求时 GitLab 发送的 webhook 事件示例，还包括根据上一节提供的 webhook 配置从事件中创建的实体。

### 有效载荷

下面是创建 GitLab 合并请求时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Webhook event payload</summary>

```json showLineNumbers
{
  "object_kind": "merge_request",
  "event_type": "merge_request",
  "user": {
    "id": 6152768,
    "name": "Your Name",
    "username": "username",
    "avatar_url": "https://secure.gravatar.com/avatar/9df2ac1caa70b0a67ff0561f7d0363e5?s=80&d=identicon",
    "email": "[REDACTED]"
  },
  "project": {
    "id": 46155864,
    "name": "Datadog Service Dependency Example",
    "web_url": "https://gitlab.com/getport-labs/datadog-service-dependency-example",
    "namespace": "port-labs",
    "default_branch": "main",
    "homepage": "https://gitlab.com/getport-labs/datadog-service-dependency-example",
    "url": "git@gitlab.com:getport-labs/datadog-service-dependency-example.git"
  },
  "object_attributes": {
    "assignee_id": 6152768,
    "author_id": 6152768,
    "created_at": "2023-06-16 14:56:31 UTC",
    "description": "Testing webhook event data",
    "id": 231009949,
    "iid": 1,
    "merge_status": "preparing",
    "merge_when_pipeline_succeeds": false,
    "milestone_id": "None",
    "source_branch": "webhook",
    "source_project_id": 46155864,
    "state_id": 1,
    "target_branch": "main",
    "target_project_id": 46155864,
    "title": "Test Webhook",
    "updated_at": "2023-06-16 14:56:31 UTC",
    "url": "https://gitlab.com/getport-labs/datadog-service-dependency-example/-/merge_requests/1",
    "source": {
      "id": 46155864,
      "name": "Datadog Service Dependency Example",
      "default_branch": "main",
      "homepage": "https://gitlab.com/getport-labs/datadog-service-dependency-example",
      "url": "git@gitlab.com:getport-labs/datadog-service-dependency-example.git"
    },
    "target": {
      "id": 46155864,
      "name": "Datadog Service Dependency Example"
    },
    "last_commit": {
      "id": "8bca1b72fc7d18d77d3e48f8d3b332165ff94898",
      "message": "finalize docs\n",
      "title": "finalize docs",
      "timestamp": "2023-05-22T17:27:13+00:00",
      "url": "https://gitlab.com/getport-labs/datadog-service-dependency-example/-/commit/8bca1b72fc7d18d77d3e48f8d3b332165ff94898",
      "author": {
        "name": "username",
        "email": "user@domain.com"
      }
    },
    "assignee_ids": [6152768],
    "reviewer_ids": [],
    "labels": [
      {
        "id": 30672355,
        "title": "Infra",
        "color": "#dc143c",
        "project_id": 46155864,
        "created_at": "2023-06-16 14:56:03 UTC",
        "type": "ProjectLabel"
      }
    ],
    "state": "opened",
    "first_contribution": true,
    "action": "open"
  },
  "labels": [
    {
      "id": 30672355,
      "title": "Infra",
      "color": "#dc143c",
      "project_id": 46155864,
      "created_at": "2023-06-16 14:56:03 UTC",
      "type": "ProjectLabel"
    }
  ],
  "changes": {},
  "repository": {
    "name": "Datadog Service Dependency Example",
    "url": "git@gitlab.com:getport-labs/datadog-service-dependency-example.git",
    "homepage": "https://gitlab.com/getport-labs/datadog-service-dependency-example"
  },
  "assignees": [
    {
      "id": 6152768,
      "name": "Your Name",
      "username": "username",
      "avatar_url": "https://secure.gravatar.com/avatar/9df2ac1caa70b0a67ff0561f7d0363e5?s=80&d=identicon",
      "email": "[REDACTED]"
    }
  ]
}
```

</details>

#### 映射结果

结合示例有效载荷和 webhook 配置可生成以下 Port 实体: 

```json showLineNumbers
{
  "identifier": "231009949",
  "title": "231009949 - Test Webhook",
  "blueprint": "gitlabMergeRequest",
  "team": [],
  "properties": {
    "state": "opened",
    "description": "Testing webhook event data",
    "mergeStatus": "preparing",
    "lastChangeType": "open",
    "repositoryUrl": "https://gitlab.com/getport-labs/datadog-service-dependency-example",
    "sourceBranch": "webhook",
    "targetBranch": "main",
    "mergeRequestUrl": "https://gitlab.com/getport-labs/datadog-service-dependency-example/-/merge_requests/1",
    "lastCommitUrl": "https://gitlab.com/getport-labs/datadog-service-dependency-example/-/commit/8bca1b72fc7d18d77d3e48f8d3b332165ff94898",
    "projectName": "Datadog Service Dependency Example",
    "projectUrl": "https://gitlab.com/getport-labs/datadog-service-dependency-example",
    "labels": [
      {
        "id": 30672355,
        "title": "Infra",
        "color": "#dc143c",
        "project_id": 46155864,
        "created_at": "2023-06-16 14:56:03 UTC",
        "type": "ProjectLabel"
      }
    ]
  },
  "relations": {}
}
```