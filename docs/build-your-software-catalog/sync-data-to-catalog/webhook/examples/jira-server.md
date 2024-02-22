---
sidebar_position: 3
description: 将 Jira Server 项目和问题纳入目录

---

import JiraProjectBlueprint from "./resources/jira-server/_example_jira_project_blueprint.mdx";
import JiraIssueBlueprint from "./resources/jira-server/_example_jira_issue_blueprint.mdx";
import JiraWebhookConfiguration from "./resources/jira-server/_example_jira_webhook_configuration.mdx";
import JiraServerConfigurationPython from "./resources/jira-server/_example_jira_server_configuration_python.mdx";

# Jira(自行托管)

在本示例中，您将在 Jira 服务器和 Port 之间创建一个 webhook 集成，该集成将有助于将 Jira 项目和问题实体导入 Port。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Jira project blueprint</summary>

<JiraProjectBlueprint/>

</details>

<details>
<summary>Jira issue blueprint</summary>

<JiraIssueBlueprint/>

</details>

:::tip  蓝图属性 您可以根据希望在 Jira 项目和问题中跟踪的内容修改蓝图中的属性。

:::

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>Jira webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Jira mapper`；
    2.标识符 : `jira_mapper`；
    3.Description : `将 Jira 项目和问题映射到 Port` 的 webhook 配置；
    4.图标 : `Jira`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <JiraWebhookConfiguration/>
     ::注意
     注意并复制此选项卡中提供的 Webhook URL
     :::
3.点击页面底部的**保存**。

</details>

## 在 Jira 中创建 webhook

1. 以具有管理全局权限的用户身份登录 Jira；
2. 单击右上角的齿轮图标；
3. 选择 **系统**；
4. 在左侧边栏底部的**高级**下，选择**Webhooks**；
5. 点击**创建 Webhook**
6. 输入以下详细信息: 
    1. 名称"- 被用于一个有意义的名称，如 Port Webhook；
    2. 状态"--请确保网络钩子已**启用**；
    3. Webhook URL` - 输入在 Port 中创建 Webhook 配置后收到的 `url` 键的值；
    4. `Description` - 输入 webhook 的描述；
    5.问题相关事件"- 在此部分输入 JQL 查询，以筛选发送到 webhook 的问题(如果此字段为空，则所有问题都将触发 webhook 事件)；
    6.问题 "下 - 标记创建、更新和删除；
    7.在 "项目相关事件 "部分下，转到 "项目 "并标记创建、更新和删除；
7.单击页面底部的**创建**。

:::tip 为了查看 Jira webhooks 中可用的不同有效载荷和事件、[look here](https://developer.atlassian.com/server/jira/platform/webhooks/)

:::

完成！您对项目或问题所做的任何更改(打开、关闭、编辑等)都会触发 webhook 事件，Jira 会将该事件发送到 Port 提供的 webhook URL。 Port 会根据映射解析事件，并相应地更新目录实体。

## 让我们来测试一下

本节包括创建或更新问题时从 Jira 发送的 webhook 事件示例。 此外，还包括根据上一节提供的 webhook 配置从事件中创建的实体。

### 有效载荷

下面是创建 Jira 问题时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Webhook event payload</summary>

```json showLineNumbers
{
  "timestamp": 1702992455854,
  "webhookEvent": "jira:issue_updated",
  "issue_event_type_name": "issue_updated",
  "user": {
    "self": "https://jira.yourdomain.com/rest/api/2/user?username=youruser",
    "name": "youruser",
    "key": "JIRAUSER10000",
    "emailAddress": "youruser@email.com",
    "avatarUrls": {
      "48x48": "https://www.gravatar.com/avatar/73b83eb1f16580bfe2bfccf81fcb1870?d=mm&s=48",
      "24x24": "https://www.gravatar.com/avatar/73b83eb1f16580bfe2bfccf81fcb1870?d=mm&s=24",
      "16x16": "https://www.gravatar.com/avatar/73b83eb1f16580bfe2bfccf81fcb1870?d=mm&s=16",
      "32x32": "https://www.gravatar.com/avatar/73b83eb1f16580bfe2bfccf81fcb1870?d=mm&s=32"
    },
    "displayName": "My User",
    "active": true,
    "timeZone": "Etc/UTC"
  },
  "issue": {
    "id": "10303",
    "self": "https://jira.yourdomain.com/rest/api/2/issue/10303",
    "key": "BSD-6",
    "fields": {
      "issuetype": {
        "self": "https://jira.yourdomain.com/rest/api/2/issuetype/10002",
        "id": "10002",
        "description": "Created by Jira Software - do not edit or delete. Issue type for a user story.",
        "iconUrl": "https://jira.yourdomain.com/images/icons/issuetypes/story.svg",
        "name": "Story",
        "subtask": false
      },
      "timespent": null,
      "project": {
        "self": "https://jira.yourdomain.com/rest/api/2/project/10001",
        "id": "10001",
        "key": "BSD",
        "name": "Basic Soft Dev",
        "projectTypeKey": "software",
        "avatarUrls": {
          "48x48": "https://jira.yourdomain.com/secure/projectavatar?avatarId=10324",
          "24x24": "https://jira.yourdomain.com/secure/projectavatar?size=small&avatarId=10324",
          "16x16": "https://jira.yourdomain.com/secure/projectavatar?size=xsmall&avatarId=10324",
          "32x32": "https://jira.yourdomain.com/secure/projectavatar?size=medium&avatarId=10324"
        }
      },
      "fixVersions": [],
      "customfield_10110": null,
      "customfield_10111": null,
      "aggregatetimespent": null,
      "resolution": null,
      "customfield_10106": null,
      "customfield_10107": null,
      "customfield_10108": null,
      "customfield_10109": null,
      "resolutiondate": null,
      "workratio": -1,
      "lastViewed": "2023-12-19T13:27:14.538+0000",
      "watches": {
        "self": "https://jira.yourdomain.com/rest/api/2/issue/BSD-6/watchers",
        "watchCount": 1,
        "isWatching": true
      },
      "created": "2023-12-19T12:14:34.524+0000",
      "priority": {
        "self": "https://jira.yourdomain.com/rest/api/2/priority/3",
        "iconUrl": "https://jira.yourdomain.com/images/icons/priorities/medium.svg",
        "name": "Medium",
        "id": "3"
      },
      "customfield_10100": "0|i001av:",
      "customfield_10101": null,
      "customfield_10102": null,
      "labels": [],
      "timeestimate": null,
      "aggregatetimeoriginalestimate": null,
      "versions": [],
      "issuelinks": [],
      "assignee": {
        "self": "https://jira.yourdomain.com/rest/api/2/user?username=janedoe",
        "name": "janedoe",
        "key": "JIRAUSER10001",
        "emailAddress": "noreplay@example.com",
        "avatarUrls": {
          "48x48": "https://www.gravatar.com/avatar/c567e3a76e53c7bd7d2dda08af2122e3?d=mm&s=48",
          "24x24": "https://www.gravatar.com/avatar/c567e3a76e53c7bd7d2dda08af2122e3?d=mm&s=24",
          "16x16": "https://www.gravatar.com/avatar/c567e3a76e53c7bd7d2dda08af2122e3?d=mm&s=16",
          "32x32": "https://www.gravatar.com/avatar/c567e3a76e53c7bd7d2dda08af2122e3?d=mm&s=32"
        },
        "displayName": "Jane Doe",
        "active": true,
        "timeZone": "Etc/UTC"
      },
      "updated": "2023-12-19T13:27:35.853+0000",
      "status": {
        "self": "https://jira.yourdomain.com/rest/api/2/status/10003",
        "description": "",
        "iconUrl": "https://jira.yourdomain.com/",
        "name": "To Do",
        "id": "10003",
        "statusCategory": {
          "self": "https://jira.yourdomain.com/rest/api/2/statuscategory/2",
          "id": 2,
          "key": "new",
          "colorName": "default",
          "name": "To Do"
        }
      },
      "components": [],
      "timeoriginalestimate": null,
      "description": "Be able to login on the app",
      "timetracking": {},
      "archiveddate": null,
      "attachment": [],
      "aggregatetimeestimate": null,
      "summary": "As a user, I want to login",
      "creator": {
        "self": "https://jira.yourdomain.com/rest/api/2/user?username=youruser",
        "name": "youruser",
        "key": "JIRAUSER10000",
        "emailAddress": "youruser@email.com",
        "avatarUrls": {
          "48x48": "https://www.gravatar.com/avatar/73b83eb1f16580bfe2bfccf81fcb1870?d=mm&s=48",
          "24x24": "https://www.gravatar.com/avatar/73b83eb1f16580bfe2bfccf81fcb1870?d=mm&s=24",
          "16x16": "https://www.gravatar.com/avatar/73b83eb1f16580bfe2bfccf81fcb1870?d=mm&s=16",
          "32x32": "https://www.gravatar.com/avatar/73b83eb1f16580bfe2bfccf81fcb1870?d=mm&s=32"
        },
        "displayName": "My User",
        "active": true,
        "timeZone": "Etc/UTC"
      },
      "subtasks": [],
      "reporter": {
        "self": "https://jira.yourdomain.com/rest/api/2/user?username=johndoe",
        "name": "johndoe",
        "key": "JIRAUSER10002",
        "emailAddress": "noreplay@example.com",
        "avatarUrls": {
          "48x48": "https://www.gravatar.com/avatar/c567e3a76e53c7bd7d2dda08af2122e3?d=mm&s=48",
          "24x24": "https://www.gravatar.com/avatar/c567e3a76e53c7bd7d2dda08af2122e3?d=mm&s=24",
          "16x16": "https://www.gravatar.com/avatar/c567e3a76e53c7bd7d2dda08af2122e3?d=mm&s=16",
          "32x32": "https://www.gravatar.com/avatar/c567e3a76e53c7bd7d2dda08af2122e3?d=mm&s=32"
        },
        "displayName": "John Doe",
        "active": true,
        "timeZone": "Etc/UTC"
      },
      "customfield_10000": "{summaryBean=com.atlassian.jira.plugin.devstatus.rest.SummaryBean@5bedd466[summary={pullrequest=com.atlassian.jira.plugin.devstatus.rest.SummaryItemBean@49d7365c[overall=PullRequestOverallBean{stateCount=0, state='OPEN', details=PullRequestOverallDetails{openCount=0, mergedCount=0, declinedCount=0}},byInstanceType={}], build=com.atlassian.jira.plugin.devstatus.rest.SummaryItemBean@fb742a[overall=com.atlassian.jira.plugin.devstatus.summary.beans.BuildOverallBean@706ec7bf[failedBuildCount=0,successfulBuildCount=0,unknownBuildCount=0,count=0,lastUpdated=<null>,lastUpdatedTimestamp=<null>],byInstanceType={}], review=com.atlassian.jira.plugin.devstatus.rest.SummaryItemBean@1bf5dc7[overall=com.atlassian.jira.plugin.devstatus.summary.beans.ReviewsOverallBean@324c9570[stateCount=0,state=<null>,dueDate=<null>,overDue=false,count=0,lastUpdated=<null>,lastUpdatedTimestamp=<null>],byInstanceType={}], deployment-environment=com.atlassian.jira.plugin.devstatus.rest.SummaryItemBean@69cdfd37[overall=com.atlassian.jira.plugin.devstatus.summary.beans.DeploymentOverallBean@6f189c8e[topEnvironments=[],showProjects=false,successfulCount=0,count=0,lastUpdated=<null>,lastUpdatedTimestamp=<null>],byInstanceType={}], repository=com.atlassian.jira.plugin.devstatus.rest.SummaryItemBean@14b2b5cf[overall=com.atlassian.jira.plugin.devstatus.summary.beans.CommitOverallBean@4283453c[count=0,lastUpdated=<null>,lastUpdatedTimestamp=<null>],byInstanceType={}], branch=com.atlassian.jira.plugin.devstatus.rest.SummaryItemBean@44a13c1e[overall=com.atlassian.jira.plugin.devstatus.summary.beans.BranchOverallBean@6f7634e8[count=0,lastUpdated=<null>,lastUpdatedTimestamp=<null>],byInstanceType={}]},errors=[],configErrors=[]], devSummaryJson={\"cachedValue\":{\"errors\":[],\"configErrors\":[],\"summary\":{\"pullrequest\":{\"overall\":{\"count\":0,\"lastUpdated\":null,\"stateCount\":0,\"state\":\"OPEN\",\"details\":{\"openCount\":0,\"mergedCount\":0,\"declinedCount\":0,\"total\":0},\"open\":true},\"byInstanceType\":{}},\"build\":{\"overall\":{\"count\":0,\"lastUpdated\":null,\"failedBuildCount\":0,\"successfulBuildCount\":0,\"unknownBuildCount\":0},\"byInstanceType\":{}},\"review\":{\"overall\":{\"count\":0,\"lastUpdated\":null,\"stateCount\":0,\"state\":null,\"dueDate\":null,\"overDue\":false,\"completed\":false},\"byInstanceType\":{}},\"deployment-environment\":{\"overall\":{\"count\":0,\"lastUpdated\":null,\"topEnvironments\":[],\"showProjects\":false,\"successfulCount\":0},\"byInstanceType\":{}},\"repository\":{\"overall\":{\"count\":0,\"lastUpdated\":null},\"byInstanceType\":{}},\"branch\":{\"overall\":{\"count\":0,\"lastUpdated\":null},\"byInstanceType\":{}}}},\"isStale\":false}}",
      "aggregateprogress": {
        "progress": 0,
        "total": 0
      },
      "environment": null,
      "duedate": null,
      "progress": {
        "progress": 0,
        "total": 0
      },
      "comment": {
        "comments": [],
        "maxResults": 0,
        "total": 0,
        "startAt": 0
      },
      "votes": {
        "self": "https://jira.yourdomain.com/rest/api/2/issue/BSD-6/votes",
        "votes": 0,
        "hasVoted": false
      },
      "worklog": {
        "startAt": 0,
        "maxResults": 20,
        "total": 0,
        "worklogs": []
      },
      "archivedby": null
    }
  },
  "changelog": {
    "id": "10407",
    "items": [
      {
        "field": "description",
        "fieldtype": "jira",
        "from": null,
        "fromString": null,
        "to": null,
        "toString": "Be able to login on the app"
      }
    ]
  }
}
```

</details>

#### 映射结果

```json showLineNumbers
{
  "identifier": "BSD-6",
  "title": "As a user, I want to login",
  "blueprint": "jiraIssue",
  "properties": {
    "url": "https://jira.yourdomain.com/rest/api/2/issue/10303",
    "status": "To Do",
    "assignee": "janedoe",
    "issueType": "Story",
    "reporter": "johndoe",
    "priority": "Medium",
    "creator": "youruser"
  },
  "relations": {
    "project": "BSD",
    "parentIssue": null,
    "subtasks": []
  },
  "filter": true
}
```

## 导入 Jira 历史问题

在本例中，您将使用 Provider 提供的 Python 脚本从 Jira Server API 获取数据并将其引用到 Port。

### 先决条件

本示例使用的是上一节中的[blueprint and webhook](#port-configuration) 定义。

此外，您还需要以下环境变量: 

* `PORT_CLIENT_ID` - 您的 Port 客户端 ID
* `PORT_CLIENT_SECRET` - 您的 Port 客户端secret
* `JIRA_API_URL` - 您的 Jira 服务器主机，例如`https://jira.yourdomain.com`。
* `JIRA_USERNAME` - 访问 Jira 软件(服务器)资源时被用于的 Jira 用户名
* `JIRA_PASSWORD` - 访问 Jira 资源时要使用的 Jira 帐户密码或令牌

:::info 使用以下命令查找您的 Port 凭据[guide](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)

:::

使用以下 Python 脚本将历史 Jira 问题引用到 Port: 

<details>
<summary>Jira Python script for historical issues</summary>

<JiraServerConfigurationPython/>

</details>

完成！现在您可以将历史问题从 Jira 导入 Port。 Port 将根据映射解析问题，并相应地更新目录实体。