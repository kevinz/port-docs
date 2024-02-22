import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import GitlabResources from './_gitlab_exporter_supported_resources.mdx'

# GitLab

通过与 GitLab 的集成，您可以将 GitLab 对象作为现有蓝图的实体导出到 Port。 集成支持实时事件处理，因此 Port 可以始终准确地实时呈现您的 GitLab 资源。

## 💡 GitLab 集成常见用例

例如，我们与 GitLab 的集成可让您轻松地直接从 GitLab 组织中获取数据来填充软件目录: 

* 映射 GitLab 组织中的所有资源，包括**组**、**项目**、**monorepos**、**合并请求**、**问题**、**Pipelines**和其他 GitLab 对象；
* 实时关注 GitLab 对象的更改(创建/更新/删除)，并自动将更改应用到 Port 中的实体；
* 使用 GitOps 管理 Port 实体；
* 等等。

## 安装

要安装 Port 的 GitLab 集成，请遵循[installation](./installation.md) 指南。

## 接收 Git 对象

通过被用于 Port 的 GitLab 集成，您可以根据实时事件自动将 GitLab 资源摄入 Port。

通过 Port 的 GitLab 集成，您可以摄取 GitLab API 提供的各种对象资源，包括组、项目、合并请求、管道等。 通过 GitLab 集成，您可以对来自 GitLab API 的数据执行提取、转换、加载(ETL)，将其转换为所需的软件目录数据模型。

GitLab 集成使用 YAML 配置来描述将数据加载到开发者门户的 ETL 流程。 这种方法体现了一个黄金分割点，即过于主观的 Git 可视化可能不适用于每个人，而过于宽泛的方法可能会给开发者门户带来不必要的复杂性。

下面是配置中的一个示例片段，演示了从 GitLab 获取 "合并请求 "数据并将其导入软件目录的 ETL 流程: 

```yaml showLineNumbers
resources:
  # Extract
  # highlight-start
  - kind: merge-request
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
    # highlight-end
    port:
      entity:
        mappings:
          # Transform & Load
          # highlight-start
          identifier: .id | tostring
          title: .title
          blueprint: '"gitlabMergeRequest"'
          properties:
            creator: .author.name
            status: .state
            createdAt: .created_at
            updatedAt: .updated_at
            link: .web_url
        # highlight-end
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 GitLab API 事件中的现有字段和值进行选择、修改、连接、转换和其他操作。

### 整合配置

集成配置是指定要从 GitLab 中查询的确切资源的方式，也是指定要从 GitLab 中填充数据的实体和属性的方式。

下面是集成配置块的示例: 

```yaml showLineNumbers
resources:
  - kind: project
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
    port:
    entity:
      mappings:
        identifier: .namespace.full_path | gsub("/";"-") # The Entity identifier will be the repository name.
        title: .name
        blueprint: '"gitlabRepository"'
        properties:
          url: .web_link
          description: .description
          namespace: .namespace.name
          fullPath: .namespace.full_path | split("/") | .[:-1] | join("/")
          defaultBranch: .default_branch
```

### 整合配置结构

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: project
      selector:
      ...
  ```


* 类型 "键是 GitLab API 中对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: project
        selector:
        ...
  ```


  <GitlabResources/>

#### 过滤不需要的对象

通过 "选择器 "和 "查询 "键，你可以准确地筛选出指定 "类型 "中的哪些对象将被录入软件目录


  ```yaml showLineNumbers
  resources:
    - kind: project
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

Port"、"实体 "和 "映射 "键打开了用于将 GitLab API 对象字段映射到Port实体的部分。 要创建多个同类映射，可以在 "资源 "数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: project
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one GitLab API object to a Port entity. Each value is a JQ query.
            identifier: .namespace.full_path | gsub("/";"-")
            title: .name
            blueprint: '"gitlabRepository"'
            properties:
              url: .web_link
              description: .description
              namespace: .namespace.name
              fullPath: .namespace.full_path | split("/") | .[:-1] | join("/")
              defaultBranch: .default_branch
        # highlight-end
    - kind: project # In this instance project is mapped again with a different filter
      selector:
        query: '.name == "MyRepositoryName"'
      port:
        entity:
          mappings: ...
  ```


:::提示 注意 `blueprint` 键的值，如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`)::: 

## 权限

Port 的 GitLab 集成需要一个范围为 `api` 的组访问令牌。

要创建组访问令牌，请按照[installation](./installation.md#creating-a-gitlab-group-access-token) 指南中的说明进行操作

## 示例

有关实用配置及其相应的蓝图定义，请参阅[examples](./examples.md) 页面。

## GitOps

Port 的 GitLab 集成还提供 GitOps 功能，请参阅[GitOps](./gitops/gitops.md) 页面了解更多信息。

## 高级

有关高级用例和示例，请参阅[advanced](./advanced.md) 页面。