---

sidebar_position: 4
sidebar_label: 页面权限

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 页面权限

页面有 4 种常规的 CRUD 权限: "创建"、"读取"、"更新 "和 "删除"。

目前只能修改 `读取`权限。

## 获取页面权限

任何用户都可以获取特定页面的权限: 

<Tabs groupId="view-permissions" queryString values={[
{label: "From the UI", value: "ui"},
{label: "From the API", value: "api"}
]}>

<TabItem value="ui">

在软件目录中，选择要查看权限的页面，然后单击 "权限": 

<img src='/img/software-catalog/pages/viewPagePermissions.gif' width='70%' />

</TabItem>

<TabItem value="api">

:::info 

* 请记住，API 请求需要访问令牌，如果需要生成一个新的访问令牌，请参阅[Getting an API token](/build-your-software-catalog/sync-data-to-catalog/api/api.md#get-api-token) 。
* 目前，要查看页面标识符，可以通过向  
GET 请求到 `https://api.getport.io/v1/pages`

:::

向 URL 发送 **HTTP GET** 请求: `https://api.getport.io/v1/pages/{page_identifier}/permissions`。

响应将包含允许读取(查看)所请求页面的角色和用户: 

```json showLineNumbers
{
  "read": {
    "roles": ["Admin", "Member"],
    "users": ["exampleUser1@example.com", "exampleUser2@example.com"],
    "teams": ["Team 1", "Team 2"]
  }
}
```

该响应体表明这些角色、用户和团队拥有阅读该页面的权限。 此外，未出现在该请求体中的每个角色、用户和团队都没有查看该页面的权限。

</TabItem>

</Tabs>

:::note 只能申请软件目录页面的页面权限。 例如，不能更改 "生成器 "页面和 "审计日志 "页面的权限。

:::

## 更新页面权限

只有具有 "admin "角色的用户才能更新目录页面的权限: 

<Tabs groupId="edit-permissions" queryString values={[
{label: "From the UI", value: "ui"},
{label: "From the API", value: "api"}
]}>

<TabItem value="ui">

在软件目录中，选择要编辑权限的页面，然后点击 "权限"。 选择要授予权限的用户或团队，然后点击 "完成"。

<img src='/img/software-catalog/pages/editPagePermissions.gif' width='70%' />

</TabItem>

<TabItem value="api">

要更新页面权限，需要指定对页面拥有权限的角色、团队或用户。

要执行更新，请向以下 URL 发送 **HTTP PATCH** 请求: `https://api.getport.io/v1/pages/{page_identifier}/permissions`。

下面是一个请求正文示例: 

```json showLineNumbers
{
  "read": {
    "roles": ["Admin", "Member"]
  }
}
```

:::tip 

PATCH` API 只对请求正文中指定的密钥执行更新。 请确保在请求正文中只包含相关密钥(用户、角色或团队)。

如果不指定特定密钥(例如请求中的 `users`，用户对特定页面的权限将保持不变)。

更改权限时，任何未出现在请求正文相应键值中的角色、用户或团队都将失去对页面的权限(这就是删除页面权限的方法)。

:::

</TabItem>

</Tabs>

### 示例

让我们介绍一组页面权限，然后探讨不同的 `PATCH` 请求体如何改变页面的有效权限。

给定页面的权限如下: 

```json showLineNumbers
{
  "read": {
    "roles": ["Admin", "Member"],
    "users": [],
    "teams": []
  }
}
```

#### 为角色添加权限

发送包含以下正文的 **HTTP PATCH** 请求，将赋予 `Services-Moderator` 角色查看页面的权限(不会删除任何现有角色的权限): 

```json showLineNumbers
{
  "read": {
    "roles": ["Admin", "Member", "Services-Moderator"]
  }
}
```

#### 删除角色权限

发送包含以下正文的 **HTTP PATCH** 请求，将移除 `Member` 角色查看页面的权限: 

```json showLineNumbers
{
  "read": {
    "roles": ["Admin"]
  }
}
```

#### 为用户添加权限

使用以下正文发出 **HTTP PATCH** 请求，将赋予指定用户查看页面的权限(不改变现有`角色`的权限): 

```json showLineNumbers
{
  "read": {
    "users": ["exampleUser1@example.com", "exampleUser2@example.com"]
  }
}
```

#### 为团队添加权限

发送包含以下正文的 **HTTP PATCH** 请求，将赋予指定团队查看页面的权限(无需更改现有 "角色 "的权限): 

```json showLineNumbers
{
  "read": {
    "teams": ["Team 1", "Team 2"]
  }
}
```

:::info 可以在单个`PATCH`请求中更新多个权限键(`角色`、`团队`和/或`用户`)，但请注意，任何未指定的`角色`、`团队`或`用户`，如果之前拥有页面权限，则将失去这些权限。

:::

## 锁定页面

锁定页面会影响具有过滤和/或隐藏功能的部件。

有关锁定页面的不同方法，请参阅下面的章节: 

<Tabs values={[
{label: "API", value: "api"},
{label: "UI", value: "ui"}
]}>

<TabItem value="api">

要锁定页面，请向以下 URL 发送 **HTTP PATCH** 请求: `https://api.getport.io/v1/pages/{page_identifier}`。

正文如下

```json showLineNumbers
{
  "locked": true
}
```

</TabItem>

<TabItem value="ui">

拥有更新页面权限的用户(通常是具有管理员角色的用户)可以锁定页面的部件。

1. 单击 "保存页面 "按钮，在所需视图中保存页面；
2. 打开页面菜单，点击 "锁定页面"。

</TabItem>

</Tabs>

:::note 锁定的页面标题旁边会有 "锁定 "图标。

<center>

![Locked Page](../../../static/img/software-catalog/pages/LockedPage.png)

</center>

:::