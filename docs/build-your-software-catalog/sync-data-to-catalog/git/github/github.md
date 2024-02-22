import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import GitHubResources from './_github_exporter_supported_resources.mdx'

# GitHub

通过与 GitHub 的集成，您可以将 GitHub 对象作为现有蓝图的实体导出到 Port。 集成支持实时事件处理，因此 Port 可以始终准确地实时呈现您的 GitHub 资源。

## 💡 GitHub 集成常见用例

例如，我们与 GitHub 的集成可让您直接从 GitHub 组织中获取数据，轻松填充软件目录: 

* 映射 GitHub 组织中的所有资源，包括**仓库**、**拉取请求**、**工作流**、**工作流运行**、**团队**、**dependabot 警报**、**部署环境**和其他 GitHub 对象。
* 实时关注 GitHub 对象的变更(创建/更新/删除)，并自动将变更应用到 Port 中的实体。
* 使用 GitOps 管理 Port 实体。
* 直接从 Port 触发 GitHub 工作流。

## 安装

要安装 Port 的 GitHub 应用程序，请遵循[installation](./installation.md) 指南。

## 接收 Git 对象

通过被用于 Port 的 GitHub 应用程序，您可以根据实时事件自动将 GitHub 资源摄入 Port。

Provider 的 GitHub 应用程序允许您摄取 GitHub API 提供的各种对象资源，包括资源库、拉取请求、工作流等。 GitHub 应用程序允许您对来自 GitHub API 的数据执行提取、转换、加载(ETL)，将其转换为所需的软件目录数据模型。

GitHub 应用程序使用一个 YAML 配置文件来描述将数据加载到开发者门户的 ETL 流程。 这种方法体现了一个黄金分割点，即过于主观的 Git 可视化可能不适用于每个人，而过于宽泛的方法可能会给开发者门户带来不必要的复杂性。

下面是 `port-app-config.yml` 文件中的一个示例片段，演示了从 GitHub 组织获取 `githubPullRequest` 数据并将其导入软件目录的 ETL 流程: 

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
          identifier: ".head.repo.name + (.id|tostring)" # The Entity identifier will be the repository name + the pull request ID. After the Entity is created, the exporter will send `PATCH` requests to update this pull request within Port.
          title: ".title"
          blueprint: '"githubPullRequest"'
          properties:
            creator: ".user.login"
            assignees: "[.assignees[].login]"
            reviewers: "[.requested_reviewers[].login]"
            status: ".status" # merged, closed, open
            closedAt: ".closed_at"
            updatedAt: ".updated_at"
            mergedAt: ".merged_at"
            prNumber: ".id"
            link: ".html_url"
            # highlight-end
```

该应用程序利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 GitHub API 事件中的现有字段和值进行选择、修改、连接、转换和其他操作。

###`port-app-config.yml` 文件

Port-app-config.yml "文件用于指定要从 GitHub 组织中查询的确切资源，也用于指定要从 GitHub 中填充数据的实体和属性。

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
          blueprint: '"githubRepository"'
          properties:
            url: ".html_url"
            description: ".description"
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


* 类型 "键是 GitHub API 中对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: repository
        selector:
        ...
  ```


  <GitHubResources/>

#### 过滤不需要的对象

通过 "选择器 "和 "查询 "键，你可以准确筛选出哪些指定 "类型 "的对象将被录入软件目录: 


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

Port"、"实体 "和 "映射 "键打开了用于将 GitHub API 对象字段映射到Port实体的部分。 要创建多个同类映射，可在 "资源 "数组中添加另一项: 


  ```yaml showLineNumbers
  resources:
    - kind: repository
      selector:
        query: "true"
      # highlight-start
      port:
        entity:
          mappings: # Mappings between one GitHub API object to a Port entity. Each value is a JQ query.
            currentIdentifier: ".name" # OPTIONAL - keep it only in case you want to change the identifier of an existing entity from "currentIdentifier" to "identifier".
            identifier: ".name"
            title: ".name"
            blueprint: '"githubRepository"'
            properties:
              description: ".description"
              url: ".html_url"
              defaultBranch: ".default_branch"
      # highlight-end
    - kind: repository # In this instance repository is mapped again with a different filter
      selector:
        query: '.name == "MyRepositoryName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 请注意 `blueprint` 键的值，如果要使用硬编码字符串，则需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`)

:::

#### 设置

要使用[`port-app-config.yml`](#port-app-configyml-file) 文件引用 GitHub 对象，可以使用以下方法之一: 

<Tabs queryString="method">

<TabItem label="Using Port" value="port">

使用 Port 管理 GitHub 集成配置: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 GitHub 引用的蓝图。
3. 从菜单中选择**采集数据**选项。
4. 在 Git Provider 类别下选择 GitHub。
5. 在编辑器中添加 `port-app-config.yml` 文件的内容。
6. 点击保存配置。

使用这种方法会将配置引用到 GitHub apply 有权限访问的所有版本库。

在使用 Port***配置集成时，在摄取数据窗口中指定的配置是全局的，允许您在编辑器中为多个 Port 蓝图指定映射，而不管您选择的是哪个蓝图。

</TabItem>

<TabItem label="Using GitHub" value="github">

要使用 GitHub 管理 GitHub 集成配置，可以选择全局配置或细粒度配置: 

* **全球配置: ** 在您的组织中创建一个 `.github-private` 仓库，并将 `port-app-config.yml` 文件添加到该仓库。
    - 使用这种方法会将配置引用到 GitHub 应用程序有权限访问的所有版本库(除非该配置被版本库中的细粒度`port-app-config.yml`覆盖)。
* **粒度配置: ** 将 `port-app-config.yml` 文件添加到所需版本库的 `.github` 目录中。
    - 使用此方法只将配置引用到存在 `port-app-config.yml` 文件的版本库。

当**使用 GitHub**进行全局配置时，只有当`port-app-config.yml`文件位于版本库的**默认分支**(通常为`main`)中时，才会引用`port-app-config.yml`文件中指定的配置。

</TabItem>

</Tabs>

:::info  重要事项 当**使用 Port** 时，指定的配置将优先于任何其他配置源(包括使用 GitHub 的全局配置和使用 GitHub 的细粒度配置)。

如果要删除 Port 中指定的配置并改用 Github，只需将 Port 中的映射内容替换为`null`，然后点击`保存并重新同步`。

:::

## 权限

Port 的 GitHub 集成需要以下权限: 

* 版本库权限: 
    - **操作: ** 读取和写入(用于使用 GitHub 工作流执行自助操作)。
    - **管理: ** 只读(用于导出版本库团队)
    - **检查: ** 读取和写入(用于验证 `port.yml`)。
    - **内容: ** 只读。
    - **元数据: ** 只读。
    - **问题: ** 只读。
    - **拉取请求: ** 读取和写入。
    - **依赖机器人警报: ** 只读。
    - **部署: ** 只读。
    - **环境: ** 只读。
    - **代码扫描警报: ** 只读。
    - 
* 组织权限: 
    - **成员: ** 只读(用于导出组织团队)。
    - **管理: ** 只读(用于导出组织用户)。
* 版本库事件(需要通过 Webhook 接收来自 GitHub 的更改并对其应用 `port-app-config.yml` 配置): 
    - 问题
    - 拉取请求
    - 推送
    - 工作流运行
    - 团队
    - Dependabot 警报
    - 部署
    - 分支保护规则
    - 代码扫描警报
    - 成员
    - 成员

:::note 首次安装应用程序时，系统会提示您确认这些权限。

您可以随时重新配置应用程序，让它访问新的资源库，或取消访问权限。

:::

## 示例

有关实用配置及其相应的蓝图定义，请参阅[examples](./examples.md) 页面。

## GitOps

Port 的 GitHub 应用程序也提供 GitOps 功能，请参考[GitOps](./gitops/gitops.md) 页面了解更多信息。

## 高级

有关高级用例和示例，请参阅[advanced](./advanced.md) 页面。

## 自行安装

Port 的 GitHub 应用程序还支持自托管安装，请参考[self-hosted installation](./self-hosted-installation.md) 页面了解更多信息。