---
sidebar_position: 4
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import DeleteDependents from '../../../../generalTemplates/_delete_dependents_git_explanation_template.md'

# 高级

GitLab 集成支持额外的 flag 以提供更多配置，从而更容易根据自己的喜好配置其行为。

要使用高级配置和附加 flag，请将它们作为根键添加到[integration configuration](./gitlab.md#the-integration-configuration) ，例如添加 `createMissingRelatedEntities` flag: 

```yaml showLineNumbers
# highlight-next-line
createMissingRelatedEntities: true
resources:
  - kind: merge-request
    selector:
      query: "true"
    port:
      entity:
        mappings:
        ... mappings configuration
```

## 使用高级配置

以下是可用的高级配置参数，可添加到[integration configuration](./gitlab.md#the-integration-configuration) 中: 

<Tabs groupId="config" queryString="parameter">

<TabItem label="Spec path" value="specPath">

`specPath` 参数指定了一个[globPatterns](https://www.malikbrowne.com/blog/a-beginners-guide-glob-patterns)
 [] 列表，Port 的 GitLab 应用程序将在其中搜索 `port.yml` 文件。

* 默认值:  `**/port.yml
* 被用于: 
    - 如果希望应用程序扫描与 `port.yml` 不同的文件(例如，使用模式 `**/my-port-config.yml` 更改配置，使应用程序扫描名为 `my-port-config.yml` 的文件)；
    - 如果希望应用程序忽略某些路径下的 `port.yml` 文件。

</TabItem>

<TabItem label="Delete dependent entities" value="deleteDependent">

<DeleteDependents/>

* 默认:  `false`(禁用)
* 被用于: 删除从属 Port 实体。如果要删除必填关系中的目标实体(及其源实体)，则必须启用。

</TabItem>

<TabItem label="Enable merge entity" value="enableMergeEntity">

enableMergeEntity "参数用于指定在创建 `port.yml` 文件中所列实体时，是使用[create/update](../../api/api.md?operation=create-update#usage) 还是[create/override](../../api/api.md?operation=create-override#usage) 策略。

* 默认值: `false`(使用创建/覆盖)
* 用例: 如果希望 GitLab 作为目录实体的真实来源，请使用 `false`。如果希望将 GitLab 作为目录中实体的某些属性的来源，而将其他来源用于自动更改的属性，则使用 `true`。

</TabItem>

<TabItem value="createMissingRelatedEntities" label="Create missing related entities">

createMissingRelatedEntities"(创建缺失的相关实体)参数用于在软件目录中还不存在目标相关实体的情况下，自动创建缺失的相关 Port 实体。

* 默认值: `false`(不创建缺失的相关实体)
* 用例: 如果希望 GitLab 应用程序在软件目录中不存在相关实体的情况下创建裸相关实体，请使用 `true`。

</TabItem>

</Tabs>