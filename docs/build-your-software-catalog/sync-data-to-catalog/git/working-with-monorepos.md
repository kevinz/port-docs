---
sidebar_position: 4
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# 使用 monorepos

[connecting your Git provider](/build-your-software-catalog/sync-data-to-catalog/git/) 到 Port 后，Port 会自动为安装集成的组织中的每个存储库创建一个<PortTooltip id="entity">实体</PortTooltip>。

如果你使用的是**monorepo**，并希望在单个仓库中为每个微服务创建一个实体，你可以通过调整 Git 集成的映射来实现这一点: 

1. 访问门户网站[data-sources page](https://app.getport.io/dev-portal/data-sources) 。
2. 在 "Exporters"(出口商)下，点击要编辑的 Git Provider，例如

<img src='/img/sync-data-to-catalog/monorepoDataSourcesExample.png' width='50%' />

<br/><br/>

3.将打开一个窗口，其中包含集成的 YAML 映射。

根据被用于的情况使用以下代码段(在 `resources` 数组中添加 `folder` 条目，或者用它替换整个 YAML): 

:::info  选择要包含的 repos 和文件夹 在下面的代码段中，复制前将 `path` 和 `repos` 字段更改为所需数值。

:::

<Tabs groupId="git-provider" queryString values={[
  { value: 'github', label: 'GitHub' },
  { value: 'gitlab', label: 'GitLab' },
  { value: 'bitbucket', label: 'BitBucket' }]}>

<TabItem value="github">

```yaml showLineNumbers
resources:
  - kind: folder
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
      folders: # Specify the repositories and folders to include under this relative path.
        #highlight-start
        - path: apps/* # Relative path to the folders within the repositories.
          repos: # List of repositories to include folders from.
            - backend-service
            - frontend-service
        #highlight-end
    port:
      entity:
        mappings:
          identifier: ".folder.name"
          title: ".folder.name"
          blueprint: '"service"'
          properties:
            url: .repo.html_url + "/tree/" + .repo.default_branch  + "/" + .folder.path
            readme: file://README.md
```

</TabItem>

<TabItem value="gitlab">

```yaml showLineNumbers
resources:
  - kind: folder
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
      folders: # Specify the repositories and folders to include under this relative path.
        #highlight-start
        - path: "apps/" # Relative path to the folders within the repositories.
          repos: # List of repositories to include folders from.
            - backend-service
            - frontend-service
        #highlight-end
    port:
      entity:
        mappings:
          identifier: .folder.name
          title: .folder.name
          blueprint: '"service"'
          properties:
            url: >-
              .repo.web_url + "/tree/" + .repo.default_branch  + "/" +
              .folder.path
            language: .repo.__languages | to_entries | max_by(.value) | .key
            readme: file://README.md
```

</TabItem>

<TabItem value="bitbucket">

```yaml showLineNumbers
resources:
  - kind: folder
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
      folders: # Specify the repositories and folders to include under this relative path.
        #highlight-start
        - path: apps/* # Relative path to the folders within the repositories.
          repos: # List of repositories to include folders from.
            - backend-service
            - frontend-service
        #highlight-end
    port:
      entity:
        mappings:
          identifier: .folder.name
          blueprint: '"service"'
          properties:
            url: .repo.links.html.href + "/src/" + .repo.mainbranch.name + "/" + .folder.path
            readme: file://README.md
```

</TabItem>

</Tabs>

4.点击 "Resync "应用更改。
5.返回[catalog](https://app.getport.io/services) ，可以看到 Port 已为指定存储库中的每个文件夹创建了一个实体，而不是为每个存储库创建一个实体。
