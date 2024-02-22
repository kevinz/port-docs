---
title: 将数据输入软件目录

sidebar_label: 将数据纳入软件目录

---

# 将数据输入软件目录

<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/pSS37tvvEtM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>

Port 提供多种集成功能，让您可以利用基础设施中已在使用的工具轻松引用和管理数据。

![Catalog Architecture](../../../static/img/sync-data-to-catalog/catalog-arch.jpg)

## 简介

Port 的集成方法允许您同时采集[blueprints](../define-your-data-model/setup-blueprint/setup-blueprint.md#blueprint-structure) 和[entities](#entity-json-structure) 。

通过使用 Port 的集成，您可以确保软件目录始终是最新的，实时数据直接从您的系统中引用，这对您的环境来说是最可靠的真实来源。

## 创建实体

实体是与蓝图定义的类型相匹配的对象，它代表软件组件的数据，这些数据由蓝图属性定义。

## 实体 JSON 结构

这是实体的基本结构: 

```json showLineNumbers
{
  "identifier": "unique-ID",
  "title": "Title",
  "team": [],
  "blueprint": "blueprintName",
  "properties": {
    "property1": "",
    "property2": ""
  },
  "relations": {}
}
```

#### 结构表


| Field        | Type     | Description                                                                                                                                                                                                                                                            |
| ------------ | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `identifier` | `String` | Unique identifier. <br /> Note that while the identifier is unique, it can be changed after creation.                                                                                                                                                                  |
| `title`      | `String` | Entity name that will be shown in the UI.                                                                                                                                                                                                                              |
| `team`       | `Array`  | **Optional Field.** An array of the associated teams. <br /> Note that group permissions are handled according to this array, see [Teams and ownership](#teams-and-ownership).                                                                                         |
| `blueprint`  | `String` | The name of the [blueprint](../define-your-data-model/setup-blueprint/setup-blueprint.md) that this entity is based on.                                                                                                                                                |
| `properties` | `Object` | An object containing key-value pairs, where each key is a property **as defined in the blueprint definition**, and each value applies the `type` of the property.                                                                                                      |
| `relations`  | `object` | An object containing key-value pairs.<br /> Each key is the identifier of the [relation](../define-your-data-model/relate-blueprints/relate-blueprints.md) that is defined on the blueprint.<br /><br />See more in the [related entities](#related-entities) section. |


#### 团队和所有权

:::info  团队和所有权 `团队`键定义实体的所有权，并控制谁可以修改或删除现有实体。

如需进一步了解 Port 的所有权，请参阅我们的[permissions](../../sso-rbac/rbac/rbac.md) 部分。

:::

#### 相关实体

当两个蓝图连接时，创建 "源 "蓝图的实体将显示一个附加选项--"相关性"。

该选项在 "关系 "部分显示如下: 

#### 单一关系示例

当使用 `many = false` 配置蓝图之间的关系时，可以通过添加 `relationIdentifier` 作为键，并添加 `relatedEntityIdentifier` 作为值，为实体添加关系: 

```json showLineNumbers
"relations": {
    "relation-identifier": "relatedEntityIdentifier"
}
```

#### 许多关系示例

当使用 `many = true` 配置蓝图之间的关系时，您可以通过添加 `relationIdentifier` 作为键，以及 `relatedEntityIdentifier`(s) 作为值的数组，为实体添加关系: 

```json showLineNumbers
"relations": {
    "relation-identifier": ["relatedEntityIdentifier1", "relatedEntityIdentifier2"]
}
```

:::tip 点击[**relations**](../define-your-data-model/relate-blueprints/relate-blueprints.md) 了解更多详情。

:::

## 输入集成方法

Port 提供各种数据摄取集成和方法，这些方法可以轻松地将数据摄取到目录中并保持更新。

请使用以下链接了解 Port 提供的不同数据引用方法: 

* * [REST](../../api-reference/api-reference.mdx);
* [CI/CD](./ci-cd/ci-cd.md);
* [Kubernetes & ArgoCD & K8s CRDs](./kubernetes/kubernetes.md);
* [IaC](./iac/iac.md);
* [Git providers](./git/git.md);
* [AWS](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/aws.md),[Azure](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/azure/azure.md),[GCP](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/gcp/gcp.md) ；
* [Cloud Cost](./cloud-cost/opencost.md);
* [Event Processing](./event-processing/kafka.md);
* 事件管理 -[PagerDuty](./incident-management/pagerduty.md),[Opsgenie](./incident-management/opsgenie.md),[FireHydrant](./incident-management/firehydrant.md) ；
