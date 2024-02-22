---
sidebar_position: 1
title: 晋级记分卡

sidebar_label: 📊 晋级记分卡

---

import CombinatorIntro from "../search-and-query/_combinator_intro.md"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 📊 推广记分卡

## 什么是记分卡？

**记分卡**使我们能够根据 Port 实体的属性为其定义和跟踪指标/标准。 每个记分卡都由一组规则组成，每条规则定义一个或多个需要满足的条件。 每条规则都有一个 "级别 "属性，其值可以是 "金级"、"银级 "或 "铜级"。

**例如**，为了跟踪贵组织的 "服务 "成熟度，我们可以在 "服务 "[blueprint](../build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md) 上创建一组记分卡，以跟踪其进展情况。

## 💡 记分卡用例

例如，记分卡可用于评估软件目录中任何实体的成熟度、产品准备情况和工程质量: 

* 某项服务是否定义了 on-call？
* 版本库中是否存在 README.md 文件？
* 是否为 K8s 集群定义了 Grafana？
* 某个实体的关系是否为空？

在[live demo](https://demo.getport.io/serviceEntity?identifier=load-generator&amp;activeTab=8) 这个示例中，您可以看到在一项服务上定义的记分卡及其评价。 🎬

## 记分卡结构表

单张记分卡定义了一个类别，用于对不同的检查、验证和评估进行分组。 以下是单张记分卡的结构: 


| Field                        | Type     | Description                                                                                                                                         |
| ---------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `title`                      | `String` | Scorecard name that will be shown in the UI                                                                                                         |
| `identifier`                 | `String` | The unique identifier of the `Scorecard`. The identifier is used for API calls, programmatic access and distinguishing between different scorecards |
| [`filter`](#filter-elements) | `Object` | Optional set of [conditions](#conditions) to filter entities that will be evaluated by the scorecard                                                |
| [`rules`](#rule-elements)    | `Object` | The rules that we create for each scorecard to determine its level                                                                                 |


例如，服务成熟度记分卡可以包含 3 条规则，而生产准备就绪记分卡可以包含 2 条完全不同的规则。

## 规则要素

通过规则，您可以在记分卡内只针对实体和属性生成检查。

记分卡规则是由多项检查组成的单一评价，每项规则都有一个级别，直接表示检查通过的重要程度(检查越基本，级别越低): 


| Field         | Type     | Description                                                                                                                                                   |
| ------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `title`       | `String` | `Rule` name that will be shown in the UI                                                                                                                      |
| `description` | `String` | Description that will be shown in the UI when the rule is expanded. Value that contains markdown is also supported and will be displayed in a markdown format |
| `identifier`  | `String` | The unique identifier of the `Rule`                                                                                                                           |
| `level`       | `String` | one of `Gold` `Silver` `Bronze`                                                                                                                               |
| `query`       | `Object` | The query is built from an array of [`conditions`](#conditions) and a [`combinator`](#combinator) (or / and) that will define the                             |


#### Combinator

<CombinatorIntro />

<Tabs groupId="combinators" defaultValue="and" values={[
{label: "And", value: "and"},
{label: "Or", value: "or"}
]}>

<TabItem value="and">

```json showLineNumbers
{
  // highlight-next-line
  "combinator": "and",
  "conditions": [
    {
      "operator": "isNotEmpty",
      "property": "on_call"
    },
    {
      "operator": "<",
      "property": "open_incidents",
      "value": 5
    }
  ]
}
```

</TabItem>

<TabItem value="or">

```json showLineNumbers
{
  // highlight-next-line
  "combinator": "or",
  "conditions": [
    {
      "operator": "isNotEmpty",
      "property": "on_call"
    },
    {
      "operator": "<",
      "property": "open_incidents",
      "value": 5
    }
  ]
}
```

</TabItem>

</Tabs>

### 条件

条件是一些小的布尔检查，有助于根据指定的[`combinator`](#combinator) 确定 "查询 "的最终状态: 


| Field      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `operator` | Search operator to use when evaluating this rule, for example `=`, `!=`, `contains`, `doesNotContains`, `isEmpty`, `isNotEmpty` below                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `property` | Property to filter by according to its value. It can be a [meta-property](../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/meta-properties.md) such as `$identifier`, or any other standard entity property such as `slack_channel` including [Mirror Properties](../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/mirror-property/mirror-property.md) and [Calculation Properties](../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/calculation-property/calculation-property.md) |
| `value`    | Value to compare to (not required in isEmpty and isNotEmpty operators)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |


#### 可用操作员


| Operator           | Supported Types                                  | Description                                                           |
| ------------------ | ------------------------------------------------ | --------------------------------------------------------------------- |
| `=`                | `String`, `Number`, `Boolean`                    | checks if the rule value is equal to the entity value                 |
| `!=`               | `String`, `Number`, `Boolean`                    | checks if the rule value is not equal to the entity value             |
| `<=`               | `String`, `Number`                               | checks if the rule value is less than or equal to the entity value    |
| `>=`               | `String`, `Number`                               | checks if the rule value is greater than or equal to the entity value |
| `<`                | `String`, `Number`                               | checks if the rule value is less than the entity value                |
| `>`                | `String`, `Number`                               | checks if the rule value is greater than the entity value             |
| `contains`         | `String`, `Number`                               | checks if the rule value is contained within the entity value         |
| `doesNotContains`  | `String`, `Number`                               | checks if the rule value is not contained within the entity value     |
| `endsWith`         | `String`, `Number`                               | checks if the rule value ends with the entity value                   |
| `doesNotEndsWith`  | `String`, `Number`                               | checks if the rule value does not end with the entity value           |
| `beginsWith`       | `String`, `Number`                               | checks if the rule value begins with the entity value                 |
| `doesNotBeginWith` | `String`, `Number`                               | checks if the rule value does not begin with the entity value         |
| `isEmpty`          | `String`, `Number`, `Boolean`, `Array`, `Object` | checks if the rule value is an empty string, array, or object         |
| `isNotEmpty`       | `String`, `Number`, `Boolean`, `Array`, `Object` | checks if the rule value is not an empty string, array, or object     |


## 计分卡总水平计算

记分卡由多个规则组成，每个规则都有一个 `level` 属性。

可用的 "记分卡 "级别有

基本"->"铜色"->"银色"->"金色

实体***总是***从**`基本`**级开始。

规则的***最低等级是 "青铜"。

例如，一旦某个实体通过了某个级别的所有规则，它的级别就会相应改变: 

1. 实体从 "基本 "级开始；
2. 它有两个等级为`青铜`的规则；
3. 一旦实体通过了这两条规则，它的级别就是`青铜`；
4. 它有四条规则，级别为 "银色"；
5. 一旦实体通过了这四条规则(以及 "青铜 "级别的规则)，它的级别就是 "白银"；
6. 等等。

:::note  在上面列出的例子中，假设实体只通过了两条 "青铜 "规则中的一条，但它通过了所有的 "白银 "规则。 记分卡的 "级别 "仍然是 "基本"，因为没有满足所有的 "青铜 "规则。

:::

## 过滤器滤芯

通过过滤器，您可以只对您真正关心的实体和属性应用记分卡检查。

过滤器的查询结构与[rules](#rule-elements) 相同。

记分卡过滤器用于确保只对相关实体进行评估，只有过滤器评估为`true`的实体才会被检查指定的规则: 


| Field                       | Description                                               |
| --------------------------- | --------------------------------------------------------- |
| [`combinator`](#combinator) | Defines the logical operation to apply to the query rules |
| [`conditions`](#conditions) | An array of boolean conditions to filter entities with    |


## 记分卡用户界面指标

为蓝图配置记分卡后，在[entity page](/customize-pages-dashboards-and-plugins/page/entity-page) 中创建的每个实体都会有一个 "记分卡 "选项卡，详细说明不同的检查及其结果: 

![Developer Portal Scorecards Tab](../../static/img/software-catalog/scorecard/tutorial/ScorecardsTab.png)

此外，每个蓝图的[catalog page](/customize-pages-dashboards-and-plugins/page/catalog-page) 会自动为每个记分卡规则设置一列。例如，这个 `service` 蓝图配置了 4 个规则，我们可以在目录中看到每个规则都有一列: 

![catalogPageScorecardColumns](../../static/img/software-catalog/scorecard/catalogPageScorecardColumns.png)

### 定制视图

您可以使用表格操作(排序、编辑、分组等)为记分卡创建各种有用的视图。 例如，以下是按团队分组的某组织中所有 "服务 "的得分: 

![catalogViewScorecardsByTeam](../../static/img/software-catalog/scorecard/catalogViewScorecardsByTeam.png)

请注意，表格中的每一列(记分卡指标)底部都有一个汇总表，将鼠标悬停在该汇总表上，可查看表格中所有实体的该指标合规情况。

### 规则结果摘要

记分卡规则会自动添加到相关目录页面的栏目中，每个栏目都会在底部进行汇总。 例如，这些服务的记分卡中定义了一些规则，我们可以看到这一点: 

* 蝙蝠侠团队 "100% 的服务都定义了随叫随到，但只有 67% 的服务的公关周期短于 1500 分钟。
* 表格底部包含了所有服务(所有团队)的每条规则的结果汇总。在总共 18 项服务中，有 11 项服务的构建成功率高于 70%。

![catalogRuleSummaries](../../static/img/software-catalog/scorecard/catalogRuleSummaries.png)

## 下一步

[Explore How to Create, Edit, and Delete Scorecards with basic examples](/promote-scorecards/usage)

[Dive into advanced operations on Scorecards with our API ➡️ ](/api-reference/api-reference.mdx)
