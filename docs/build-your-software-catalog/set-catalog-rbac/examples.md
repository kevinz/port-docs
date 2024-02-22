import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 示例

在本节中，我们将举例说明在组织中使用目录权限的方法，以及如何应用这些权限。

## 用例 💡

在使用目录权限管理时，除其他配置外，还可以使用以下配置: 

1. 可以为特定用户/角色设置实体不可变/部分不可变(只能创建/删除/修改)。例如
    1. 部署 "实体对所有角色都是不可变的，而 "集群 "实体只有蓝图的**管理者**才能编辑。
    2. **成员**可以创建新的`Microservice`实体，但不允许删除`Microservice`实体。
2.对于特定用户/角色，每个实体的属性/关系都可以是不可变的。例如，"repository_link "属性对所有角色(除**Admin**外)都是不可变的。
3.只允许特定用户/角色修改实体[owned by their team](#setting-permissions-by-team-ownership) 。例如，**Members** 只能编辑属于其团队的 `Microservices`。
4.可向特定用户或角色授予操作执行权限。例如，您可以允许每个**成员**创建新的 "部署 "实体，但只有 "部署 "**管理员**才能执行 "添加资源 "的第 2 天操作。

## 设置蓝图权限

要为蓝图设置权限，请单击 DevPortal Builder 页面上所需蓝图的权限按钮。 这将打开一个包含权限 JSON 的模态，允许您控制可对蓝图或其实体执行的所有操作。

### 角色示例

<Tabs groupId="blueprint-permissions" defaultValue="register" values={[
{label: "Let member register entity", value: "register"},
{label: "Only let admin update property", value: "only-admin"},
]}>

<TabItem value="register">

如果要启用 **Members** 来注册蓝图的实体，可以按如下方式更改 JSON: 

```json showLineNumbers
{
  "entities": {
    "register": {
      // highlight-next-line
      "roles": ["Admin", "Env-moderator", "Member"], // changed from ["Admin", "Env-moderator"]
      "users": [],
      "teams": [],
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="only-admin">

要只允许 **Admins** 更改属性 `slackChannelUrl` ，请删除版主角色: 

```json showLineNumbers
{
  "entities": {
    "updateProperties": {
      "slackChannelUrl": {
        // highlight-next-line
        "roles": ["Admin"], // changed from ["Admin", "Env-moderator"]
        "users": [],
        "teams": [],
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

</Tabs>

### 用户示例

<Tabs groupId="user-permissions" defaultValue="let-user-relation" values={[
{label: "Let user edit relation", value: "let-user-relation"}
]}>

<TabItem value="let-user-relation">

要授予特定用户编辑 `deployedAt` 关系的权限，请将其添加到用户数组中: 

```json showLineNumbers
{
  "entities": {
    "updateRelations": {
      "deployedAt": {
        "roles": ["Admin", "Env-moderator"],
        // highlight-next-line
        "users": ["some-user@myorg.com"], // changed from []
        "teams": [],
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

</Tabs>

### 团队范例

<Tabs groupId="team-permissions" defaultValue="let-team-relation" values={[
{label: "Let team edit relation", value: "let-team-relation"}
]}>

<TabItem value="let-team-relation">

要授予特定团队编辑 `deployedAt` 关系的权限，请将其添加到团队数组中: 

```json showLineNumbers
{
  "entities": {
    "updateRelations": {
      "deployedAt": {
        "roles": ["Admin", "Env-moderator"],
        "users": [],
        // highlight-next-line
        "teams": ["ServiceMaintainers"], // changed from []
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

</Tabs>

### 团队所有权范例

有些操作有 "ownedByTeam"(团队所有)flag，这样就可以按团队所有设置权限，而不是按角色或直接分配给用户。

例如，以下 JSON 将允许**每个用户**(无论其角色如何)对属于其所属团队的`Env`实体(设置了`team`属性的实体)执行`delete_env`操作: 

```json showLineNumbers
{
  "actions": {
    "delete_env": {
      "execute": {
        "roles": ["Admin", "Env-moderator"],
        "users": [],
        "teams": [],
        // highlight-next-line
        "ownedByTeam": true // changed from false
      }
    }
  }
}
```

### 团队继承

团队继承允许您利用关系自动分配和管理实体所有权。

通过使用 "团队继承"，您可以配置实体自动从与之相关的实体继承其 "团队"。

团队继承可以通过在[blueprint structure](../define-your-data-model/relate-blueprints/relate-blueprints.md#structure-table) 中添加`teamInheritance`键来配置。`teamInheritance`对象有一个`path`键，表示我们要继承的实体团队蓝图的关系路径。

:::info  路径

* 路径 "键的作用与[mirror properties](../define-your-data-model/setup-blueprint/properties/mirror-property/mirror-property.md#api-definition) 中的 "路径 "键类似；
* 路径 "不需要以"$team "元属性结尾，因为它会被推断出来；
* 团队继承只能使用[`single`](../define-your-data-model/relate-blueprints/relate-blueprints.md#single-relation-structure) 类型关系的路径来配置。

:::

例如，下面的 JSON(添加到**蓝图定义**)将配置`myBlueprint`蓝图的实体，使其从`myExtraRelatedBlueprint`蓝图的实体继承其团队: 

```json showLineNumbers
{
  "identifier": "myBlueprint",
  // ...blueprint properties
  "teamInheritance": {
    "path": "myRelatedBlueprint.myExtraRelatedBlueprint"
  }
}
```

:::note 在上面的示例中，关系链是: myBlueprint -> myRelatedBlueprint -> myExtraRelatedBlueprint

:::

### Global VS granular permissions

为蓝图实体授予写入权限时，有两个控制级别: 

1. global 权限 - 创建/更新整个实体。例如，允许 **Member** 用户更新 `Env` 实体(所有属性和关系)。
2. 细粒度权限--控制用户/角色在创建或更新实体时可以更新哪些属性和关系。例如，只允许**成员**用户更新`Env`实体的`slackChannelUrl`属性。

要为蓝图引用细粒度权限，可使用 JSON 中的 "updateProperties "和 "updateRelations "字段，请参阅下面的示例: 

<Tabs groupId="global-granular-permissions" defaultValue="let-user-property" values={[
{label: "Let user edit property", value: "let-user-property"},
{label: "Let user edit entity", value: "let-user-entity"}
]}>

<TabItem value="let-user-property">

以下更改将允许 **Member** 用户仅更新 `Env` 实体的 `slackChannelUrl` 属性: 

```json showLineNumbers
{
  "entities": {
    "updateProperties": {
      "slackChannelUrl": {
        // highlight-next-line
        "roles": ["Admin", "Env-moderator", "Member"], // changed from ["Admin", "Env-moderator"]
        "users": [],
        "teams": [],
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

<TabItem value="let-user-entity">

如果要引用全局权限，请使用 JSON 中的 `update` 字段。

以下更改将允许 **Member** 用户更新其团队拥有的 `Env` 实体的每一个属性/关系: 

```json showLineNumbers
{
  "entities": {
    "update": {
      "roles": ["Admin", "Env-moderator"],
      "users": [],
      "teams": [],
      // highlight-next-line
      "ownedByTeam": true // changed from false
    }
  }
}
```

</TabItem>

</Tabs>

:::warning 使用 global 权限会覆盖任何已设置的细粒度权限。

如果同时设置了两种权限类型，则在评估权限时将引用 global 设置。

:::

:::info 在注册新实体时，"更新"、"更新属性 "和 "更新关系 "设置也同样适用。 这意味着用户不能用他没有权限编辑的属性(或关系)注册新实体。

:::