---

sidebar_position: 1

---

import InstallTerraform from "./_terraform_provider_base.mdx"

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# Terraform

通过我们与[Terraform](https://www.terraform.io/) 的集成，您可以将基础设施的状态与 Port 中代表它们的实体结合起来。

通过使用 Port 的 Terraform Provider，您可以轻松地将 Port 与现有的 IaC 定义集成，Terraform 提供的每个资源也可以使用相同的 `.tf` 定义文件报告到软件目录。

:::info  Port terraform Provider 您可以查看 Terraform Provider 的官方注册表页面[here](https://registry.terraform.io/providers/port-labs/port-labs/)

:::

## 💡 Terraform Provider 常见用例

例如，我们的 Terraform Providers 可让您直接从 IaC 定义中获取数据，轻松填充软件目录: 

* 报告**云账户**；
* 报告**数据库**；
* 报告**lambdas**和**托管 Kubernetes 服务**(EKS、AKS、GKE 等)；
* 等等。

## 安装

<InstallTerraform/>

## Terraform 定义结构

Port 的 Terraform Provider 支持以下资源将数据摄取到目录中: 

### `Port实体

[`port_entity`](https://registry.terraform.io/providers/port-labs/port-labs/latest/docs/resources/port_entity) 资源定义了一个基本实体: 

```hcl showLineNumbers
# highlight-start
resource "port_entity" "myEntity" {
  identifier = "myEntity" # Entity identifier
  title      = "My Entity" # Entity title
  blueprint  = "myBlueprint" # Identifier of the blueprint to create this entity from
 # highlight-end

  # Entity property values
  properties = {
    ...
  }
  ...
  # Entity relations
  ...
}
```

以下参数是**必需的**: 

* blueprint` - 据以创建此实体的蓝图的标识符；
* title: 实体的标题；
* 一个或多个[`properties`](#properties-schema) 模式定义。

还可以指定以下参数作为 `port_entity` 资源的一部分: 

* `identifier` - 实体的标识符；
    - 如果未提供`identifier`，则会自动生成一个标识符。
* `teams` - 拥有该实体的团队数组；
* `run_id` - 创建实体的操作的运行 ID。

### `属性`模式

[`properties`](https://registry.terraform.io/providers/port-labs/port-labs/latest/docs/resources/port_entity#nested-schema-for-properties) 模式会为实体的某个属性指定一个指定值。

```hcl showLineNumbers
resource "port_entity" "myEntity" {
  identifier = "myEntity" # Entity identifier
  title      = "My Entity" # Entity title
  blueprint  = "myBlueprint" # Identifier of the blueprint to create this entity from

# highlight-start
  properties = {
    string_props = {
      "myStringProp" = "My string"
      }

    number_props = {
      "myNumberProp" = 7
      }

    array_props = {
      string_items = {
        "myArrayProp" = ["a", "b", "c"]
        }
      }
 # highlight-end
  }
  # Entity relations
  ...
}
```

#### 定义

<Tabs groupId="properties" queryString="property" defaultValue="string" values={[
{label: "String", value: "string"},
{label: "Number", value: "number"},
{label: "Boolean", value: "boolean"},
{label: "Object", value: "object"},
{label: "Array", value: "array"},
{label: "URL", value: "url"},
{label: "Email", value: "email"},
{label: "User", value: "user"},
{label: "Team", value: "team"},
{label: "Datetime", value: "datetime"},
{label: "Timer", value: "timer"},
{label: "YAML", value: "yaml"}
]}>

<TabItem value="string">

```hcl showLineNumbers
properties = {
  string_props = {
   "myStringProp" = "My string"
  }
}
```

</TabItem>
<TabItem value="number">

```hcl showLineNumbers
properties = {
  number_props = {
    "myNumberProp" = 7
  }
}
```

</TabItem>
<TabItem value="boolean">

```hcl showLineNumbers
properties = {
  boolean_props = {
    "myBooleanProp" = true
  }
}
```

</TabItem>
<TabItem value="object">

```hcl showLineNumbers
properties = {
  object_props = {
    "myObjectProp" = jsonencode({ "my" : "object" })
  }
}
```

</TabItem>
<TabItem value="array">

```hcl showLineNumbers
properties = {
  array_props = {
    string_props = {
      "myArrayProp" = ["a", "b", "c"])
    }
  }
}
```

</TabItem>
<TabItem value="url">

```hcl showLineNumbers
properties = {
  string_props = {
    "myUrlProp" = "https://example.com"
  }
}
```

</TabItem>
<TabItem value="email">

```hcl showLineNumbers
properties = {
  string_props = {
    "myEmailProp" = "me@example.com"
  }
}
```

</TabItem>
<TabItem value="user">

```hcl showLineNumbers
properties = {
 string_props = {
   "myUserProp" = "argo-admin"
  }
}
```

</TabItem>
<TabItem value="team">

```hcl showLineNumbers
properties = {
  string_props = {
    "myTeamProp" = "argo-admins"
  }
}
```

</TabItem>
<TabItem value="datetime">

```hcl showLineNumbers
properties = {
  string_props = {
    "myDatetimeProp" = "2023-04-18T11:44:15.345Z"
  }
}
```

</TabItem>
<TabItem value="timer">

```hcl showLineNumbers
properties = {
  string_props = {
    "myUserProp" = "argo-admin"
  }
}
```

</TabItem>
<TabItem value="yaml">

```hcl showLineNumbers
properties = {
  string_props = {
    "myYamlProp" = "myKey: myValue"
  }
}
```

</TabItem>

</Tabs>

### `relations` 模式

[`relations`](https://registry.terraform.io/providers/port-labs/port-labs/latest/docs/resources/port_entity#relations) 模式将目标实体映射到源实体定义: 

```hcl showLineNumbers
resource "port_entity" "myEntity" {
  identifier = "myEntity" # Entity identifier
  title      = "My Entity" # Entity title
  blueprint  = "myBlueprint" # Identifier of the blueprint to create this entity from

  # Entity properties
  ...

# highlight-start
  relations = {
    single_relations = {
    "mySingleRelation" = "myTargetEntityIdentifier"
    }
  }

  relations = {
    many_relations = {
      "myManyRelation" = ["myTargetEntityIdentifier", "myTargetEntityIdentifier2"]
    }
  }
 # highlight-end
}
```

#### 定义

<Tabs groupId="relations" queryString="relations" defaultValue="single" values={[
{label: "Single", value: "single"},
{label: "Many", value: "many"},
]} >

<TabItem value="single">

模式如下

```hcl showLineNumbers
relations {
  single_relations = {
    # Key-value pair of the relation identifier and the target identifier
    "mySingleRelation" = "myTargetEntityIdentifier"
  }
}
```

</TabItem>

<TabItem value="many">

模式如下

```hcl showLineNumbers
relations {
  many_relations = {
    # Key-value pair of the relation identifier and the target identifiers
    "myManyRelation" = ["myTargetEntityIdentifier", "myTargetEntityIdentifier2"]
  }
}
```

</TabItem>

</Tabs>

## 使用 Terraform Provider 被引用数据

要使用 Terraform Provider 将数据引用到软件目录，需要在 Terraform 定义文件中定义[`port_entity`](https://registry.terraform.io/providers/port-labs/port-labs/latest/docs/resources/port_entity) 资源: 

<Tabs groupId="sync-data" queryString="current-scenario" defaultValue="create" values={[
{label: "Create", value: "create"},
{label: "Update", value: "update"},
{label: "Delete", value: "delete"},
]} >

<TabItem value="create">

要使用 Terraform 创建实体，请在`.tf`定义文件中添加`port_entity`资源: 

```hcl showLineNumbers
resource "port_entity" "myEntity" {
  identifier = "myEntity"
  title      = "My Entity"
  blueprint  = "myBlueprint"

  properties = {
    "string_props" = {
      "myStringProp" = "My string"
    }
    "number_props" = {
      "myNumberProp" = 7
    }
    "boolean_props" = {
      "myBooleanProp" = true
    }
    "object_props" = {
      "myObjectProp" = jsonencode({ "my" : "object" })
    }
    "array_props" = {
    "string_props" = {
        "myArrayProp" = ["a", "b", "c"]
      }
    }
  }
}
```

然后运行以下命令应用更改并更新目录: 

```shell showLineNumbers
# To view Terraform's planned changes based on your .tf definition file:
terraform plan
# To apply the changes and update the catalog
terraform apply
```

运行这些命令后，您将看到目录已更新为新实体。

</TabItem>

<TabItem value="update">

要使用 Terraform 更新实体，请更新`.tf`定义文件中现有的`port_entity`资源，然后运行`terraform apply`。

也可以使用 Port 的 Terraform Provider 开始管理现有实体，要开始管理现有实体，请在`.tf`定义文件中添加新的`port_entity`资源，并进行所需的更改: 

```hcl showLineNumbers
resource "port_entity" "myExistingEntity" {
  # highlight-next-line
  identifier = "myExistingEntity"
  title      = "My Entity"
  blueprint  = "myBlueprint"

  # Entity properties and relations
  ...
}
```

:::info 关于向 Terraform Provider 添加现有实体的重要说明: 

* 指定实体的 "标识符 "非常重要，否则 terraform 将创建一个带有自动生成的标识符的新实体。
* Port 的 Terraform Providers 使用[create/override](../../api/api.md?update-strategy=create-override#usage) 策略，这意味着对于现有实体，资源定义中未定义的任何属性都将被引用空值。

:::

</TabItem>

<TabItem value="delete">

要使用 Terraform 删除实体，只需删除在 `.tf` 定义文件中定义的 `port_entity` 资源，然后运行 `terraform apply` 即可。

</TabItem>

</Tabs>

## 将现有数据导入 Terraform 状态

<Tabs groupId="terraform-import" queryString="current-scenario" defaultValue="entity" values={[
{label: "Blueprint", value: "blueprint"},
{label: "Entity", value: "entity"},
]} >

<TabItem value="blueprint">

要将现有蓝图导入 Terraform 状态，请在`.tf`定义文件中添加`port_blueprint`资源: 

```hcl showLineNumbers
# highlight-start

resource "port_blueprint" "myBlueprint" {
  ...
}

# highlight-end
```

然后运行以下命令将蓝图导入 Terraform 状态: 

```shell showLineNumbers
terraform import port_blueprint.myBlueprint "{blueprintIdentifier}"
```

</TabItem>

<TabItem value="entity">

要将现有实体导入 Terraform 状态，请在`.tf`定义文件中添加`port_entity`资源: 

```hcl showLineNumbers
# highlight-start

resource "port_entity" "myEntity" {
  ...
}

# highlight-end
```

然后运行以下命令将实体导入 Terraform 状态: 

```shell showLineNumbers
terraform import port_entity.myEntity "{blueprintIdentifier}:{entityIdentifier}"
```

</TabItem>

</Tabs>

## 示例

有关实用配置及其相应的蓝图定义，请参阅[examples](./examples/examples.md) 页面。