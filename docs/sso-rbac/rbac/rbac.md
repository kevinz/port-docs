# 角色、团队和所有权

Port 的 RBAC 机制可以为特定用户和团队分配权限，还可以根据 Port 中不同角色的需要配置自定义角色。

## 分配权限

在 Port 中，可以通过[roles](#roles) 、[team ownership](../../build-your-software-catalog/set-catalog-rbac/examples.md#team-ownership-examples) 和[users](../../build-your-software-catalog/set-catalog-rbac/examples.md#user-examples) 来分配权限。

### 角色

共有 3 种角色，以下是其开箱即用的权限: 


| Role                         | Description                                                                                                      |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Admin**                    | Perform any operation on the platform                                                                            |
| **Moderator** of a Blueprint | Perform any operation on a specific blueprint and its entities. A user can be a moderator of multiple blueprints |
| **Member**                   | Read-only permissions + permissions to execute actions                                                           |


:::info **Moderator** 角色会在创建蓝图时自动创建。 例如，创建 "Env "蓝图会生成一个名为 "Env-moderator "的角色，该角色可以对 "Env "蓝图及其实体执行任何操作。

:::

#### 层次结构

除了为每个角色指定权限外，还可根据以下层次结构继承权限: 

**管理员** > **版主** > **成员**

例如，如果允许**会员**编辑 "集群 "实体，那么 "微服务 "**管理员**也可以编辑它们(**管理员**可以编辑所有蓝图下的所有实体)。

您可以在用户表中查看(和编辑)每个用户的角色: 

![Users page](../../../static/img/software-catalog/role-based-access-control/permissions/usersPageRolesHightlight.png)

有关用户页面的更多信息，请参阅[Users and Teams](#users-and-teams-management) 部分

## 用户和团队管理

在 Port 中，您可以在一个地方控制和管理所有用户和团队。

这样，管理员就可以

* 邀请用户加入他们的组织，并为他们分配特定的角色和团队。
* 管理团队及其成员。
* 在组织内推广资产所有权(团队分配)。
* 在门户网站上设置细粒度权限(权限管理)。

这也将使开发商受益，他们可以

* 了解自己拥有并负责哪些软件资产。
* 根据自己的角色和团队，查看自己的资产并对其执行操作。

每个用户都由以下属性定义: 

1. 基本信息--镜像、姓名和电子邮件。
2. 角色 - 用户的权限级别(请参阅[set catalog RBAC](../../build-your-software-catalog/set-catalog-rbac/set-catalog-rbac.md) 部分) ；
3. 团队--"团队 "是拥有实体的一组用户(请参阅[team](#team-meta-property) 部分) 。

可通过以下方式管理用户和团队

* Port[Users & Teams page](#users--teams-page)
* Port's[Terraform provider](#terraform-provider)
* Port的[Port API](#port-api)

### 用户和团队页面

![Teams and Users page](../../../static/img/software-catalog/role-based-access-control/users-and-teams/usersAndTeams.png)

#### 用户选项卡

在用户选项卡中，您可以

* 查看所有用户；
* 邀请新用户
* 编辑用户
* 删除用户；
* 等等。

#### 团队选项卡

在团队选项卡中，您可以

* 查看所有团队；
* 创建新团队
* 编辑团队
* 删除团队；
* 等等。

:::tip  为用户和团队使用 SSO

**注意: ** 下列限制不适用于在 Port 内手动创建的团队。

启用单点登录(SSO)后，用户和团队信息(包括团队成员资格)将直接来自身份 Providers (IdP)。

由于这些团队是通过 IdP 同步的，因此无法对其执行以下操作: 

* 编辑 SSO 团队成员；
* 删除 SSO 团队。

如果您尝试执行其中一项禁用的操作，界面将显示解释: 

![Managed by SSO notice](../../../static/img/software-catalog/role-based-access-control/users-and-teams/createTeamNoticeWithSSO.png)

:::

### Terraform Provider

您可以通过[Terraform provider](https://registry.terraform.io/providers/port-labs/port-labs/latest) 执行上述操作。下面是一个`main.tf`文件的基本示例，该文件定义了一个有 3 个用户的团队: 

```bash showLineNumbers
resource "port_team" "example" {
  name        = "example"
  description = "example"
  users = [
    "user1@test.com",
    "user2@test.com",
    "user3@test.com",
  ]
}
```

您可以浏览 "团队 "模式[here](https://registry.terraform.io/providers/port-labs/port-labs/latest/docs/resources/port_team) 。

### Port API

应用程序接口允许您管理[teams](https://api.getport.io/static/index.html#/Teams) 和[users](https://api.getport.io/static/index.html#/Users) 。

### `团队'元属性

每个实体都有一个名为 "team "的[meta-property](../../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/meta-properties.md) ，允许您设置哪个团队拥有该实体。作为管理员，您还可以根据该字段设置蓝图权限。

包含`team`字段的实体 JSON 示例: 

```json showLineNumbers
{
  "identifier": "unique-ID",
  "title": "Entity Title",
  "team": [],
  "blueprint": "testBlueprint",
  "properties": {
    "prop1": "value"
  },
  "relations": {}
}
```

实体创建/编辑页面中的团队下拉选择器: 

![Team property](../../../static/img/software-catalog/role-based-access-control/users-and-teams/teamPropertyMarkedInUIForm.png)


| Field | Type | Description                                            | Default      |
| ----- | ---- | ------------------------------------------------------ | ------------ |
| team  | List | System field that defines the team that owns an Entity | `"team": []` |


* 我们支持在 Port 上手动创建团队，也支持与身份 Provider(如[Okta](../sso-providers/okta.md) 和[AzureAD](../sso-providers/azure-ad.md) )集成以导入现有团队。
* 当用户登录 Port 时，他们的群组将自动从其身份 Providers 中提取，允许的团队值也会相应更新。
* 还可以配置[team inheritance](../../build-your-software-catalog/set-catalog-rbac/examples.md#team-inheritance) 并利用关系自动填充实体的 "团队 "键。

:::info Okta 和 AzureAD 集成只有在从相关身份提供程序配置 SSO 后才能使用，详情请参考[Single Sign-On (SSO)](../sso-providers/) 部分。

:::