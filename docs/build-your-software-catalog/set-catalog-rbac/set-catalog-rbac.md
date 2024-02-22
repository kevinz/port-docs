---

title: 设置目录 RBAC
sidebar_label: 设置目录 RBAC

---

import OwnershipTemplate from "./_ownership_explanation_template.mdx";
import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";
import styles from "./styles.module.css";

# 设置 RBAC 目录

<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/p-pNcvVfdUE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>

Provider 提供细粒度控制，确保每个用户只能看到目录中与他们相关的部分。

Port 目录 RBAC 功能由 Port 的[permissions controls](../../sso-rbac/rbac/rbac.md) 启用。

:::tip 

要管理谁可以查看 Port 中的哪些页面，请查看[page permissions](../../customize-pages-dashboards-and-plugins/page/page-permissions.md) 。

:::

## 💡 常用目录 RBAC Usage

例如，目录 RBAC 允许管理员精细控制哪些用户可以访问目录中的哪些信息: 

* 只向开发人员显示他们拥有的服务；
* 允许用户只编辑实体的特定属性；
* 为开发人员创建完全只读的视图；
* 等等。

## 设置目录数据的全局访问控制

创建时分配给每个蓝图的默认权限规定，具有管理员角色的用户和具有特定蓝图主持人角色的用户可以对蓝图执行任何操作。

还可以为**实体**分配全局权限控制: 

<Tabs groupId="permission" defaultValue="create">

<TabItem value="create" label="Create (register)">

要分配创建实体的权限，请在 "注册 "对象下赋予所需的角色权限，如下所示: 

<Tabs groupId="target" defaultValue="role">

<TabItem value="role" label="Role">

要将 `create` 权限赋予另一个角色，请将其添加到 `roles` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "register": {
      // highlight-next-line
      "roles": ["my-blueprint-moderator", "Admin", "my-role"], // added my-role
      "users": [],
      "teams": [],
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="user" label="User">

要将 `create` 权限赋予另一个用户，请将其添加到 `users` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "register": {
      "roles": ["my-blueprint-moderator", "Admin"],
      // highlight-next-line
      "users": ["my-user@example.com"], // added my-user@example.com
      "teams": [],
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="team" label="Team">

要向另一个团队授予 "创建 "权限，请将其添加到 "团队 "数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "register": {
      "roles": ["my-blueprint-moderator", "Admin"],
      "users": [],
      // highlight-next-line
      "teams": ["my-team"], // added my-team
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="ownership" label="Ownership">

<OwnershipTemplate />

要向实体的拥有团队成员授予 `create` 权限，请更改 `ownedByTeam` 键: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "register": {
      "roles": ["my-blueprint-moderator", "Admin"],
      "users": [],
      "teams": [],
      // highlight-next-line
      "ownedByTeam": true // changed from false
    }
  }
}
```

:::tip 在 "创建 "权限的上下文中，"由团队拥有 "是指用户只有为一个新实体指定了他所属的团队，才能创建该实体。

:::

</TabItem>

</Tabs>

</TabItem>

<TabItem value="update" label="Update" attributes={{ className: styles.admonition_tip }}>

要分配更新实体的权限，请在 "更新 "对象下赋予所需的角色权限，如下所示: 

<Tabs groupId="target" defaultValue="role">

<TabItem value="role" label="Role" attributes={{ className: styles.admonition_tip }}>

要将 `update` 权限赋予另一个角色，请将其添加到 `roles` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "update": {
      // highlight-next-line
      "roles": ["my-blueprint-moderator", "Admin", "my-role"], // added my-role
      "users": [],
      "teams": [],
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="user" label="User" attributes={{ className: styles.admonition_tip }}>

要将 `update` 权限赋予另一个用户，请将其添加到 `users` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "update": {
      "roles": ["my-blueprint-moderator", "Admin"],
      // highlight-next-line
      "users": ["my-user@example.com"], // added my-user@example.com
      "teams": [],
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="team" label="Team" attributes={{ className: styles.admonition_tip }}>

要向另一个团队授予 "更新 "权限，请将其添加到 "团队 "数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "update": {
      "roles": ["my-blueprint-moderator", "Admin"],
      "users": [],
      // highlight-next-line
      "teams": ["my-team"], // added my-team
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="ownership" label="Ownership" attributes={{ className: styles.admonition_tip }}>

<OwnershipTemplate />

要向实体的拥有团队成员授予 `update` 权限，请更改 `ownedByTeam` 键: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "update": {
      "roles": ["my-blueprint-moderator", "Admin"],
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

</TabItem>

<TabItem value="delete" label="Delete (unregister)" attributes={{ className: styles.admonition_caution }}>

要分配删除实体的权限，请在 `delete` 对象下赋予所需的角色权限，如下所示: 

<Tabs groupId="target" default ="role">

<TabItem value="role" label="Role" attributes={{ className: styles.admonition_caution }}>

要将 `delete` 权限赋予另一个角色，请将其添加到 `roles` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "unregister": {
      // highlight-next-line
      "roles": ["my-blueprint-moderator", "Admin", "my-role"], // added my-role
      "users": [],
      "teams": [],
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="user" label="User" attributes={{ className: styles.admonition_caution }}>

要将 `delete` 权限赋予另一个用户，请将其添加到 `users` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "unregister": {
      "roles": ["my-blueprint-moderator", "Admin"],
      // highlight-next-line
      "users": ["my-user@example.com"], // added my-user@example.com
      "teams": [],
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="team" label="Team" attributes={{ className: styles.admonition_caution }}>

要向另一个团队授予 "删除 "权限，请将其添加到 "团队 "数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "unregister": {
      "roles": ["my-blueprint-moderator", "Admin"],
      "users": [],
      // highlight-next-line
      "teams": ["my-team"], // added my-team
      "ownedByTeam": false
    }
  }
}
```

</TabItem>

<TabItem value="ownership" label="Ownership" attributes={{ className: styles.admonition_caution }}>

<OwnershipTemplate />

要向实体的拥有团队成员授予 "删除 "权限，请更改 "ownedByTeam "键: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "unregister": {
      "roles": ["my-blueprint-moderator", "Admin"],
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

</TabItem>

</Tabs>

## 设置目录数据的细粒度访问控制

还可以为**实体**分配更细粒度的权限控制: 

<Tabs groupId="permission" defaultValue="updateProp">

<TabItem value="updateProp" label="Update specific property">

要分配更新特定实体属性的权限，请在 `updateProperties -> propertyName` 对象下赋予所需的角色权限，如下所示: 

<Tabs groupId="target" defaultValue="role">

<TabItem value="role" label="Role">

要将属性 `update` 权限赋予另一个角色，请将其添加到 `roles` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "updateProperties": {
      "myProperty": {
        // highlight-next-line
        "roles": ["my-blueprint-moderator", "Admin", "my-role"], // added my-role
        "users": [],
        "teams": [],
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

<TabItem value="user" label="User">

要将属性 `update` 权限赋予另一个用户，请将其添加到 `users` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "updateProperties": {
      "myProperty": {
        "roles": ["my-blueprint-moderator", "Admin"],
        // highlight-next-line
        "users": ["my-user@example.com"], // added my-user@example.com
        "teams": [],
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

<TabItem value="team" label="Team">

要将属性 `update` 权限赋予另一个团队，请将其添加到 `teams` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "updateProperties": {
      "myProperty": {
        "roles": ["my-blueprint-moderator", "Admin"],
        "users": [],
        // highlight-next-line
        "teams": ["my-team"], // added my-team
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

<TabItem value="ownership" label="Ownership">

<OwnershipTemplate />

要向实体的拥有团队成员授予属性 `update` 权限，请更改 `ownedByTeam` 键: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "updateProperties": {
      "myProperty": {
        "roles": ["my-blueprint-moderator", "Admin"],
        "users": [],
        "teams": [],
        // highlight-next-line
        "ownedByTeam": true // changed from false
      }
    }
  }
}
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="updateRel" label="更新特定关系" attributes={{ className: styles.admonition_tip }}>

要分配更新特定实体关系的权限，请在 `updateRelations -> relationName` 对象下赋予所需的角色权限，如下所示: 

<Tabs groupId="target" defaultValue="role">

<TabItem value="role" label="Role" attributes={{ className: styles.admonition_tip }}>

要将关系 `update` 权限赋予另一个角色，请将其添加到 `roles` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "updateRelations": {
      "myRelation": {
        // highlight-next-line
        "roles": ["my-blueprint-moderator", "Admin", "my-role"], // added my-role
        "users": [],
        "teams": [],
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

<TabItem value="user" label="User" attributes={{ className: styles.admonition_tip }}>

要将关系 `update` 权限赋予另一个用户，请将其添加到 `users` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "updateRelations": {
      "myRelation": {
        "roles": ["my-blueprint-moderator", "Admin"],
        // highlight-next-line
        "users": ["my-user@example.com"], // added my-user@example.com
        "teams": [],
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

<TabItem value="team" label="Team" attributes={{ className: styles.admonition_tip }}>

要将关系 `update` 权限赋予另一个团队，请将其添加到 `teams` 数组中: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "updateRelations": {
      "myRelation": {
        "roles": ["my-blueprint-moderator", "Admin"],
        "users": [],
        // highlight-next-line
        "teams": ["my-team"], // added my-team
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

<TabItem value="ownership" label="Ownership" attributes={{ className: styles.admonition_tip }}>

<OwnershipTemplate />

要向实体的拥有团队成员授予关系 `update` 权限，请更改 `ownedByTeam` 键: 

```json showLineNumbers
{
  "entities": {
    ... other permissions
    "updateRelations": {
      "myRelation": {
        "roles": ["my-blueprint-moderator", "Admin"],
        "users": [],
        "teams": [],
        // highlight-next-line
        "ownedByTeam": true // changed from false
      }
    }
  }
}
```

</TabItem>

</Tabs>

</TabItem>

</Tabs>

## 软件目录 RBAC 示例

有关 Port RBAC 的实际示例，请参阅[examples](./examples.md) 页面。

## FAQ

由于目录 RBAC 可以非常细化，在某些情况下，可能并不完全清楚由此分配的权限会做什么，本部分旨在提供一些实际例子以及在这些情况下 Provider 的 RBAC 的行为: 

### 如果用户没有权限编辑蓝图的必填属性，会发生什么情况？

在这种情况下，用户将无法注册或更新整个实体，因为他们无法为所需属性提供值；

### 如果实体注册启用了 `ownedByTeam` 设置，但用户无法编辑 `team` 属性，会发生什么情况？

在这种情况下，用户将无法注册新实体，因为他们无法为实体的团队字段选择一个值，并将其标记为自己团队所有。