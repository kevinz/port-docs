---
sidebar_position: 4
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import DeleteDependents from '../../../../generalTemplates/_delete_dependents_git_explanation_template.md'

# 高级

Bitbucket 集成支持额外的 flag 以提供更多配置，从而更容易根据自己的喜好配置其行为。

## 使用高级配置

可使用以下高级配置参数: 

<Tabs groupId="config" queryString="parameter">

<TabItem label="Spec path" value="specPath">

`specPath` 参数指定了一个字符串，Port 的 Bitbucket 应用程序在构建通向 `yml` 文件的搜索路径时将使用该字符串，版本库中以 `specPath` 值结尾的每个路径都将被扫描。

* 默认值:  `port.yml
* 被用于: 
    - 如果希望应用程序扫描与 `port.yml` 不同的文件(例如，使用模式 `my-port-config.yml` 更改配置应用程序扫描名为 `my-port-config.yml` 的文件)；
    - 如果希望应用程序忽略某些路径下的 `port.yml` 文件。

</TabItem>

<TabItem label="Delete dependent entities" value="deleteDependent">

<DeleteDependents/>

* 默认:  `false`(禁用)
* 被用于: 删除从属 Port 实体。如果要删除必填关系中的目标实体(及其源实体)，则必须启用。

</TabItem>

<TabItem label="Enable merge entity" value="enableMergeEntity">

enableMergeEntity "参数用于指定在创建 `port.yml` 文件中所列实体时，是使用[create/update](../../api/api.md?operation=create-update#usage) 还是[create/override](../../api/api.md?operation=create-override#usage) 策略。

* 默认值: `true`(使用创建/更新)
* 用例: 如果希望 Bitbucket 成为目录实体的真实来源，请使用 `false`。如果你想把 Bitbucket 作为目录中实体的某些属性的来源，而把其他来源用于自动更改的属性，则使用 `true`。

</TabItem>

</Tabs>

下面列出的所有高级配置都可以添加到[`port-app-config.yml`](./bitbucket.md#port-app-configyml-file) 文件中。