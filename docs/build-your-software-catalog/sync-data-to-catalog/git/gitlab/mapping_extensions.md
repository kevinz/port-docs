---

sidebar_position: 5

---

import RepositoryBlueprint from './_gitlab_exporter_example_repository_blueprint.mdx'

# 映射扩展

## 简介

将数据映射到 Port 的默认方式是使用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将数据映射和转换为 Port 实体。

不过，在某些情况下，您可能希望将数据映射到 Port，而默认的 JQ 映射是不够的。

可能的被引用案例: 

* 将版本库 README.md 文件内容映射到 Port；
* 检查版本库中是否存在特定文件；
* 检查版本库中是否存在特定字符串；
* 检查版本库中是否被引用了特定版本的 packages；
* 检查版本库中是否配置了 CI/CD Pipelines；

## 将文件内容映射到Port中

在下面的示例中，您将定义 GitLab 项目并将其 **README.md** 文件内容导出到 Port: 

<RepositoryBlueprint/>

我们可以看到，其中一个属性的类型是 markdown，这意味着我们需要将 **README.md** 文件的内容映射到 Port 中。

为此，我们将使用带有文件路径的 `file://` 前缀来告诉 GitLab 输出程序，我们要将文件内容映射到 Port 中。

```yaml showLineNumbers
- kind: project
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .path_with_namespace | gsub(" "; "")
          title: .name
          blueprint: '"gitlabRepository"'
          properties:
            url: .web_url
            // highlight-next-line
            readme: file://README.md
            description: .description
            namespace: .namespace.name
            fullPath: .namespace.full_path
            defaultBranch: .default_branch
```

## 搜索检查

我们可以被引用 GitLab 输出程序对我们的资源库进行搜索检查，以确保它们符合我们组织的政策和标准。

利用 Port 记分卡和搜索检查功能，组织可以定义对每个存储库执行的一系列检查，检查结果将反映在 Port 中存储库的记分卡中，这将有助于组织识别不符合组织政策和标准的存储库。

### 工作原理

搜索检查使用[GitLab Advanced Search API](https://docs.gitlab.com/ee/api/search.html) 。

这意味着 GitLab Search API 支持的任何搜索查询都可以被 GitLab 输出程序引用。

#### 语法

* `search://scope=<scope>&&query=<query>`
    - `search://` - 这是表示映射是搜索检查的前缀；
    - `scope` - 搜索范围，目前只支持 `blobs` (在资源库文件中搜索) (更多详情请参见[GitLab Search API](https://docs.gitlab.com/ee/api/search.html#scope) ) ；
    - `query` - 要搜索的查询，应是有效的 GitLab 高级搜索查询(详情请参见[GitLab Advanced Search syntax](https://docs.gitlab.com/ee/user/search/advanced_search.html#syntax) ) 。

#### 搜索检查示例

* `search://scope=blobs&amp;&amp;query=filename:README.md` - 检查项目是否包含 `README.md` 文件；
* `search://scope=blobs&amp;&amp;query=filename:test_* extension:py` - 检查项目是否包含测试；
* `search://scope=blobs&amp;&amp;query=filename:".gitlab-ci.yml` - 检查项目是否配置了 gitlab CI；
* `search://scope=blobs&amp;&amp;query=fastapi filename:requirements.txt` - 检查项目是否在`requirements.txt`文件中引用了 fastapi；
    在下面的截图中，我们可以看到我们使用了与 gitlab 高级搜索完全相同的查询语句
    ![GitLab Search Query Syntax](../../../../../static/img/integrations/gitlab/GitlabSearchQueryExample.png)

### 集成配置示例

```yaml showLineNumbers
- kind: project
    selector:
      query: 'true'
    port:
      entity:
        mappings:
          identifier: .path_with_namespace | gsub(" "; "")
          title: .name
          blueprint: '"gitlabRepository"'
          properties:
            url: .web_link
            description: .description
            namespace: .namespace.name
            fullPath: .namespace.full_path
            defaultBranch: .default_branch
            readme: file://README.md
            // highlight-start
            hasLicense: search://scope=blobs&&query=filename:"LICENSE"
            usingFastapiPackage: search://scope=blobs&&query=fastapi filename:requirements.txt
            hasCI: search://scope=blobs&&query=filename:.gitlab-ci.yml
            usingOldLoggingPackage: search://scope=blobs&&query=logging extension:py
            hasTests: search://scope=blobs&&query=filename:test_* extension:py
            // highlight-end
```

反映搜索检查结果的记分卡示例: 

![GitLab Search Checks Scorecard](../../../../../static/img/integrations/gitlab/GitlabSearchScorecardExample.png)
