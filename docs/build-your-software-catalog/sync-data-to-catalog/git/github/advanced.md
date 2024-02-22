---
sidebar_position: 4
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import DeleteDependents from '../../../../generalTemplates/_delete_dependents_git_explanation_template.md'

# 高级

GitHub 集成支持额外的 flag 以提供更多配置，从而更容易根据自己的喜好配置其行为。

要使用高级配置和附加 flag，请将它们作为根键添加到[`port-app-config.yml`](./github.md#port-app-configyml-file) 文件中，例如添加 `createMissingRelatedEntities` flag: 

```yaml showLineNumbers
# highlight-next-line
createMissingRelatedEntities: true
resources:
  - kind: pull-request
    selector:
      query: "true"
    port:
      entity:
        mappings:
        ... mappings configuration
```

## 使用高级配置

可将下列高级配置参数添加到[`port-app-config.yml`](./github.md#port-app-configyml-file) 文件中: 

<Tabs groupId="config" queryString="parameter">

<TabItem label="Spec path" value="specPath">

`specPath` 参数指定了一个[globPatterns](https://www.malikbrowne.com/blog/a-beginners-guide-glob-patterns)
 [] 列表，Port 的 GitHub 应用程序将在其中搜索 `port.yml` 文件。

* 默认值:  `**/port.yml
* 被引用: 
    - 如果希望应用程序扫描与 `port.yml` 不同的文件(例如，使用模式 `**/my-port-config.yml` 更改配置，使应用程序扫描名为 `my-port-config.yml` 的文件)；
    - 如果希望应用程序忽略某些路径下的 `port.yml` 文件。

</TabItem>

<TabItem label="Delete dependent entities" value="deleteDependent">

<DeleteDependents/>

* 默认:  `false`(禁用)
* 被引用: 删除从属 Port 实体。如果要删除必填关系中的目标实体(及其源实体)，则必须启用。

</TabItem>

<TabItem label="Enable merge entity" value="enableMergeEntity">

enableMergeEntity "参数用于指定在创建 `port.yml` 文件中所列实体时，是使用[create/update](../../api/api.md?operation=create-update#usage) 还是[create/override](../../api/api.md?operation=create-override#usage) 策略。

* 默认值: `true`(使用创建/更新)
* 用例: 如果希望 GitHub 成为目录实体的真实来源，请使用 `false`。如果希望将 GitHub 作为目录中实体的某些属性的来源，而将其他来源用于自动更改的属性，则使用 `true`。

</TabItem>

<TabItem value="createMissingRelatedEntities" label="Create missing related entities">

createMissingRelatedEntities"(创建缺失的相关实体)参数用于在软件目录中还不存在目标相关实体的情况下，自动创建缺失的相关 Port 实体。

* 默认值: `false`(不创建缺失的相关实体)
* 用例: 如果希望 GitHub 应用程序在软件目录中不存在相关实体的情况下创建裸相关实体，请使用 `true`。

</TabItem>

<TabItem value="enrichEntities" label="Enrich entities">

enrichEntitiesWithGitopsMetadata "参数用于启用附加元数据来丰富由 GitOps 管理的 Port 实体。

当该参数激活时，将 `port.yml` 文件中列出的实体摄取到 Port 时将包括附加信息，如规范文件路径(例如: `port.yml`、`/path/to/port.yml` 等)、最新提交信息等。

附加信息会以 JSON 对象属性的形式报告给 GitOps 托管实体。为了查看这些信息，您的[blueprint](../../../define-your-data-model/setup-blueprint/setup-blueprint.md) 需要包含一个[object property](../../../define-your-data-model/setup-blueprint/properties/object.md) 来存储元数据。该参数发送数据的默认标识符是 `gitopsMetadata`。

* 默认值: `true`(用 GitOps 元数据丰富实体)
* 用例: 如果希望 GitHub 应用程序用额外的 JSON 元数据来丰富由 GitOps 管理的 Port 实体，请使用 `true`。
    - 根据蓝图模式定义(默认属性标识符: `gitopsMetadata`)，被引用`gitopsMetadataProperty`可更改`object`属性的标识符。

**配置示例**

```yaml showLineNumbers
enrichEntitiesWithGitopsMetadata: true
gitopsMetadataProperty: myGitopsMetadata # the GitOps metadata will be sent to the "myGitopsMetadata" property of the blueprint's entities
```

</TabItem>

</Tabs>