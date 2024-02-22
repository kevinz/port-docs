---
sidebar_position: 2
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";
import DeleteDependents from '../../../generalTemplates/_delete_dependents_api_explanation_template.md'
import CreateMissingRelatedEntities from '../../../generalTemplates/_create_missing_related_entities_api_explanation_template.md'

# 高级

可使用以下高级查询参数: 

<Tabs groupId="advanced" queryString="current-config-param" defaultValue="delete_dependents" values={[
{label: "Delete Dependents", value: "delete_dependents"},
{label: "Create Missing Related Entities", value: "create_missing_related_entities"},
]} >

<TabItem value="delete_dependents">

<DeleteDependents/>

* 在特定实体的 `DELETE` API 端点可用。
* 默认: `false`(禁用)
* 被引用: 删除从属 Port 实体。如果想在实体的蓝图有必要关系时删除目标实体(及其源实体)，则必须启用。

</TabItem>

<TabItem value="create_missing_related_entities">

<CreateMissingRelatedEntities/>

* 在特定实体的 `POST`、`PUT`、`PATCH` API 端点上可用。
* 默认: `false`(禁用)
* 被引用: 创建缺失的相关 Port 实体。如果想在目标实体不存在的情况下创建源实体(及其目标相关实体)，则必须启用。

</TabItem>

</Tabs>