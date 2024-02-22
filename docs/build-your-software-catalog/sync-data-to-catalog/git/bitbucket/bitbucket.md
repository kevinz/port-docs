import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# Bitbucket

import BitbucketResources from './_bitbucket_exporter_supported_resources.mdx'

通过与 Bitbucket 的集成，您可以将 Bitbucket 对象作为现有蓝图的实体导出到 Port。 集成支持实时事件处理，因此 Port 可以始终准确地实时呈现您的 Bitbucket 资源。

## 💡 Bitbucket 集成常见用例

例如，我们的 Bitbucket 集成可以轻松地将 Bitbucket Workspace 中的数据直接填充到软件目录中: 

* 映射 Bitbucket Workspace 中的所有资源，包括**仓库**、**拉取请求**和其他 Bitbucket 对象；
* 实时关注 Bitbucket 对象的变更(创建/更新/删除)，并自动将变更应用到 Port 中的实体；
* 使用 GitOps 管理 Port 实体；
* 等等。

## 安装

要安装 Port 的 Bitbucket 应用程序，请遵循[installation](./installation.md) 指南。

## 接收 Git 对象

通过使用 Port 的 Bitbucket 应用程序，您可以根据实时事件自动将 Bitbucket 资源引用到 Port 中。

Provider 的 Bitbucket 应用程序允许您摄取 Bitbucket API 提供的各种对象资源，包括版本库、拉取请求等。 Bitbucket 应用程序允许您对来自 Bitbucket API 的数据执行提取、转换、加载(ETL)，将其转换为所需的软件目录数据模型。

Bitbucket 应用程序使用 YAML 配置文件来描述将数据加载到开发者门户的 ETL 流程。 这种方法体现了一个黄金分割点，即过于主观的 Git 可视化可能并不适用于每个人，而过于宽泛的方法可能会给开发者门户带来不必要的复杂性。

下面是`port-app-config.yml`文件中的一个示例片段，演示了从 Bitbucket Workspace 获取`pullRequest`数据并将其导入软件目录的 ETL 流程: 

```yaml showLineNumbers
resources:
  # Extract
  # highlight-start
  - kind: pull-request
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
      # highlight-end
    port:
      entity:
        mappings:
          # Transform & Load
          # highlight-start
          identifier: ".destination.repository.name + (.id|tostring)" # The Entity identifier will be the repository name + the pull request ID. After the Entity is created, the exporter will send `PATCH` requests to update this pull request within Port.
          title: ".title"
          blueprint: '"bitbucketPullRequest"'
          properties:
            creator: ".author.display_name"
            assignees: "[.participants[].user.display_name]"
            reviewers: "[.reviewers[].user.display_name]"
            status: ".state"
            createdAt: ".created_on"
            updatedAt: ".updated_on"
            link: ".links.html.href"
            # highlight-end
```

该应用程序利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对来自 Bitbucket API 事件的现有字段和值进行选择、修改、连接、转换和其他操作。

###`port-app-config.yml` 文件

Port-app-config.yml "文件用于指定要从 Bitbucket Workspace 中查询的确切资源，也用于指定要从 Bitbucket 中填充数据的实体和属性。

下面是一个 `port-app-config.yml` 块示例: 

```yaml showLineNumbers
resources:
  - kind: repository
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
    port:
      entity:
        mappings:
          identifier: ".name" # The Entity identifier will be the repository name.
          title: ".name"
          blueprint: '"bitbucketRepository"'
          properties:
            project: ".project.name"
            url: ".links.html.href"
            defaultBranch: ".main_branch"
```

###`port-app-config.yml` 结构

* Port-app-config.yml` 文件的根键是 `resources` 键: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: repository
      selector:
      ...
  ```


* 类型 "键是 Bitbucket API 中对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: repository
        selector:
        ...
  ```


  <BitbucketResources/>

#### 过滤不需要的对象

通过 "选择器 "和 "查询 "键，你可以准确地筛选出指定 "类型 "中的哪些对象将被录入软件目录


  ```yaml showLineNumbers
  resources:
    - kind: repository
      # highlight-start
      selector:
        query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
      # highlight-end
      port:
  ```


例如，要只引用名称以`"service"`开头的存储库，可以这样使用`query`键: 

```yaml showLineNumbers
query: .name | startswith("service")
```

<br/>

Port"、"实体 "和 "映射 "键打开了用于将 Bitbucket API 对象字段映射到Port实体的部分。 映射 "键可以是一个对象，也可以是一个对象数组，该数组应与Port实体的结构相匹配。[entity](../../../sync-data-to-catalog/sync-data-to-catalog.md#entity-json-structure)


  ```yaml showLineNumbers
  resources:
    - kind: repository
      selector:
        query: "true"
      # highlight-start
      port:
        entity:
          mappings: # Mappings between one Bitbucket API object to a Port entity. Each value is a JQ query.
            identifier: ".name"
            title: ".name"
            blueprint: '"bitbucketRepository"'
            properties:
              project: ".project.name"
              url: ".links.html.href"
              defaultBranch: ".main_branch"
      # highlight-end
  ```


:::提示 注意 `blueprint` 键的值，如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`)::: 

#### 设置

要使用[`port-app-config.yml`](#port-app-configyml-file) 文件引用 Bitbucket 对象，可以使用以下方法之一: 

<Tabs queryString="method">

<TabItem label="Using Port" value="port">

使用 Port 管理 Bitbucket 集成配置: 

1. 转到 DevPortal Builder 页面；
2. 选择要被 Bitbucket 引用的蓝图；
3. 从菜单中选择**录入数据**选项；
4. 在 Git Provider 类别下选择 Bitbucket；
5. 在编辑器中添加 `port-app-config.yml` 文件的内容；
6. 点击保存配置。

使用这种方法会将配置应用到 Bitbucket Workspace 中的所有版本库。

在使用 Port***配置集成时，在摄取数据窗口中指定的配置是全局的，允许您在编辑器中为多个 Port 蓝图指定映射，而不管您选择的是哪个蓝图。

</TabItem>

<TabItem label="Using Bitbucket" value="bitbucket">

要使用 Bitbucket 管理 Bitbucket 集成配置，可以选择全局配置或细粒度配置: 

* **全局配置: ** 在您的 Workspace 中创建一个 `.bitbucket-private` 仓库，并将 `port-app-config.yml` 文件添加到该仓库；
    - 使用该方法会将配置引用到 Bitbucket Workspace 中的所有版本库(除非该配置被版本库中的细粒度 `port-app-config.yml` 覆盖)；
* **粒度配置: ** 将 `port-app-config.yml` 文件添加到所需版本库的根目录；
    - 使用这种方法，配置只被引用到存在 `port-app-config.yml` 文件的版本库。

当**使用 Bitbucket**进行全局配置时，只有当`port-app-config.yml`文件位于版本库的**默认分支**(通常为`main`)中时，才会引用该文件中指定的配置。

</TabItem>

</Tabs>

:::info  重要

使用全局配置**使用 Port**时，指定的配置将覆盖任何其他配置源(包括使用 Bitbucket 的全局配置和使用 Bitbucket 的细粒度配置)；

:::

## 权限

Port 的 Bitbucket 集成需要以下范围: 

* `账户`；
* 存储库
* 拉取请求

:::note 首次安装应用程序时，系统会提示您确认这些权限。

:::

## 示例

有关实用配置及其相应的蓝图定义，请参阅[examples](./examples.md) 页面。

## GitOps

Port 的 Bitbucket 应用程序也提供 GitOps 功能，请参考[GitOps](./gitops/gitops.md) 页面了解更多信息。

## 高级

有关高级用例和示例，请参阅[advanced](./advanced.md) 页面。