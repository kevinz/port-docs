---

sidebar_position: 19
description: 将 Bitbucket Server 项目、软件仓库和拉取请求收录到你的目录中

---

import BitbucketProjectBlueprint from "./resources/bitbucket-server/_example_bitbucket_project_blueprint.mdx";
import BitbucketPullrequestBlueprint from "./resources/bitbucket-server/_example_bitbucket_pull_request_blueprint.mdx";
import BitbucketRepositoryBlueprint from "./resources/bitbucket-server/_example_bitbucket_repository_blueprint.mdx";
import BitbucketWebhookConfiguration from "./resources/bitbucket-server/_example_bitbucket_webhook_config.mdx";
import BitbucketServerPythonScript from "./resources/bitbucket-server/_example_bitbucket_python_script.mdx";

# Bitbucket(自行托管)

在本示例中，您将在 Bitbucket 服务器和 Port 之间创建 webhook 集成，该集成将有助于将 Bitbucket 项目、版本库和拉取请求实体导入 Port。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Bitbucket project blueprint</summary>

<BitbucketProjectBlueprint/>

</details>

<details>
<summary>Bitbucket repository blueprint</summary>

<BitbucketRepositoryBlueprint/>

</details>

<details>
<summary>Bitbucket pull request blueprint</summary>

<BitbucketPullrequestBlueprint/>

</details>

:::tip  蓝图属性 您可以根据 Bitbucket 账户中需要跟踪的内容，修改蓝图中的属性。

:::

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>Bitbucket webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: "Bitbucket 服务器映射器"；
    2.标识符 : `bitbucket_server_mapper`；
    3.Description : `将 Bitbucket 项目、版本库和拉取请求映射到 Port` 的 webhook 配置；
    4.图标 : `BitBucket`；
2. **集成配置**选项卡--填写以下JQ映射: 
   <BitbucketWebhookConfiguration/>
     ::注意
     注意并复制此选项卡中提供的 Webhook URL
     :::
3.点击页面底部的**保存**。

</details>

## 在 Bitbucket 中创建 webhook

1. 从 Bitbucket 账户打开要添加 webhook 的项目；
2. 点击**项目设置**或左侧边栏的齿轮图标；
3. 在工作流程部分，选择左侧边栏上的**Webhooks**；
4. 单击**添加网络钩子**按钮，为版本库创建网络钩子；
5. 输入以下详细信息: 
    1. title` - 使用一个有意义的名称，如 Port Webhook；
    2. `URL` - 输入在 Port 中创建 webhook 配置后收到的 webhook `URL` 的值；
    3. `Secret` - 输入您在 Port 中配置 webhook 时提供的secret值；
    4. 触发器
        1.在 **项目**下选择 `modified`；
        2.在 **Repository** 下选择 `modified`；
        3.在**拉取请求**下，根据用例选择任意事件。
6.单击**保存**保存 webhook；

:::tip 请访问[this documentation](https://confluence.atlassian.com/bitbucketserver/event-payload-938025882.html) 了解有关 Bitbucket 中 webhook 事件有效载荷的更多信息。

:::

完成！您在 Bitbucket 中的项目、版本库或拉取请求发生的任何更改都会触发 webhook 事件，并发送到 Port 提供的 webhook URL。 Port 会根据映射解析事件，并相应地更新目录实体。

## 让我们来测试一下

本节包括一个当拉取请求被合并时从 Bitbucket 发送的 webhook 事件示例。 此外，本节还包括根据上一节提供的 webhook 配置从该事件创建的实体。

### 有效载荷

以下是 Bitbucket 拉取请求合并时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Webhook event payload</summary>

```json showLineNumbers
{
  "body": {
    "eventKey": "pr:merged",
    "date": "2023-11-16T11:03:42+0000",
    "actor": {
      "name": "admin",
      "emailAddress": "username@gmail.com",
      "active": true,
      "displayName": "Test User",
      "id": 2,
      "slug": "admin",
      "type": "NORMAL",
      "links": {
        "self": [
          {
            "href": "http://myhost:7990/users/admin"
          }
        ]
      }
    },
    "pullRequest": {
      "id": 2,
      "version": 2,
      "title": "lint code",
      "description": "here is the description",
      "state": "MERGED",
      "open": false,
      "closed": true,
      "createdDate": 1700132280533,
      "updatedDate": 1700132622026,
      "closedDate": 1700132622026,
      "fromRef": {
        "id": "refs/heads/dev",
        "displayId": "dev",
        "latestCommit": "9e08604e14fa72265d65696608725c2b8f7850f2",
        "type": "BRANCH",
        "repository": {
          "slug": "data-analyses",
          "id": 1,
          "name": "data analyses",
          "description": "This is for my repository and all the blah blah blah",
          "hierarchyId": "24cfae4b0dd7bade7edc",
          "scmId": "git",
          "state": "AVAILABLE",
          "statusMessage": "Available",
          "forkable": true,
          "project": {
            "key": "MOPP",
            "id": 1,
            "name": "My On Prem Project",
            "description": "On premise test project is sent to us for us",
            "public": false,
            "type": "NORMAL",
            "links": {
              "self": [
                {
                  "href": "http://myhost:7990/projects/MOPP"
                }
              ]
            }
          },
          "public": false,
          "archived": false,
          "links": {
            "clone": [
              {
                "href": "ssh://git@myhost:7999/mopp/data-analyses.git",
                "name": "ssh"
              },
              {
                "href": "http://myhost:7990/scm/mopp/data-analyses.git",
                "name": "http"
              }
            ],
            "self": [
              {
                "href": "http://myhost:7990/projects/MOPP/repos/data-analyses/browse"
              }
            ]
          }
        }
      },
      "toRef": {
        "id": "refs/heads/main",
        "displayId": "main",
        "latestCommit": "e461aae894b6dc951f405dca027a3f5567ea6bee",
        "type": "BRANCH",
        "repository": {
          "slug": "data-analyses",
          "id": 1,
          "name": "data analyses",
          "description": "This is for my repository and all the blah blah blah",
          "hierarchyId": "24cfae4b0dd7bade7edc",
          "scmId": "git",
          "state": "AVAILABLE",
          "statusMessage": "Available",
          "forkable": true,
          "project": {
            "key": "MOPP",
            "id": 1,
            "name": "My On Prem Project",
            "description": "On premise test project is sent to us for us",
            "public": false,
            "type": "NORMAL",
            "links": {
              "self": [
                {
                  "href": "http://myhost:7990/projects/MOPP"
                }
              ]
            }
          },
          "public": false,
          "archived": false,
          "links": {
            "clone": [
              {
                "href": "ssh://git@myhost:7999/mopp/data-analyses.git",
                "name": "ssh"
              },
              {
                "href": "http://myhost:7990/scm/mopp/data-analyses.git",
                "name": "http"
              }
            ],
            "self": [
              {
                "href": "http://myhost:7990/projects/MOPP/repos/data-analyses/browse"
              }
            ]
          }
        }
      },
      "locked": false,
      "author": {
        "user": {
          "name": "admin",
          "emailAddress": "username@gmail.com",
          "active": true,
          "displayName": "Test User",
          "id": 2,
          "slug": "admin",
          "type": "NORMAL",
          "links": {
            "self": [
              {
                "href": "http://myhost:7990/users/admin"
              }
            ]
          }
        },
        "role": "AUTHOR",
        "approved": false,
        "status": "UNAPPROVED"
      },
      "reviewers": [],
      "participants": [],
      "properties": {
        "mergeCommit": {
          "displayId": "1cbccf99220",
          "id": "1cbccf99220b23f89624c7c604f630663a1aaf8e"
        }
      },
      "links": {
        "self": [
          {
            "href": "http://myhost:7990/projects/MOPP/repos/data-analyses/pull-requests/2"
          }
        ]
      }
    }
  },
  "headers": {
    "X-Forwarded-For": "10.0.148.57",
    "X-Forwarded-Proto": "https",
    "X-Forwarded-Port": "443",
    "Host": "ingest.getport.io",
    "X-Amzn-Trace-Id": "Self=1-6555f719-267a0fce1e7a4d8815de94f7;Root=1-6555f719-1906872f41621b17250bb83a",
    "Content-Length": "2784",
    "User-Agent": "Atlassian HttpClient 3.0.4 / Bitbucket-8.15.1 (8015001) / Default",
    "Content-Type": "application/json; charset=UTF-8",
    "accept": "*/*",
    "X-Event-Key": "pr:merged",
    "X-Hub-Signature": "sha256=bf366faf8d8c41a4af21d25d922b87c3d1d127b5685238b099d2f311ad46e978",
    "X-Request-Id": "d5fa6a16-bb6c-40d6-9c50-bc4363e79632",
    "via": "HTTP/1.1 AmazonAPIGateway",
    "forwarded": "for=154.160.30.235;host=ingest.getport.io;proto=https"
  },
  "queryParams": {}
}
```

</details>

#### 映射结果

```json showLineNumbers
{
   "identifier":"2",
   "title":"lint code",
   "blueprint":"bitbucketPullrequest",
   "properties":{
      "created_on":"2023-11-16T10:58:00Z",
      "updated_on":"2023-11-16T11:03:42Z",
      "merge_commit":"9e08604e14fa72265d65696608725c2b8f7850f2",
      "state":"MERGED",
      "owner":"Test User",
      "link":"http://myhost:7990/projects/MOPP/repos/data-analyses/pull-requests/2",
      "destination":"main",
      "source":"dev",
      "participants":[],
      "reviewers":[]
   },
   "relations":{
      "repository":"data-analyses"
   },
   "filter":true
}
```

## 导入 Bitbucket 历史问题

在本示例中，您将使用 Provider 提供的 Python 脚本从 Bitbucket 服务器 API 获取数据并将其引用到 Port。

### 先决条件

本示例使用的是上一节中的[blueprint and webhook](#port-configuration) 定义。

此外，请 Provider 下列环境变量: 

* `PORT_CLIENT_ID` - 您的 Port 客户端 ID
* `PORT_CLIENT_SECRET` - 您的 Port 客户端secret
* `BITBUCKET_HOST` - Bitbucket 服务器主机，例如 `http://localhost:7990`.
* `BITBUCKET_USERNAME` - 访问 Bitbucket 资源时被引用的 Bitbucket 用户名
* `BITBUCKET_PASSWORD` - Bitbucket 账号密码

:::info 使用以下命令查找您的 Port 凭据[guide](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)

:::

使用以下 Python 脚本将历史 Bitbucket 项目、源和拉取请求引用到 Port 中: 

<details>
<summary>Bitbucket Python script</summary>

<BitbucketServerPythonScript/>

</details>

完成！现在您可以将历史项目、版本库和拉取请求从 Bitbucket 服务器导入 Port。 Port 将根据映射解析对象，并相应地更新目录实体。