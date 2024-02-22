---

sidebar_position: 2

---

# 实体页面

每个[entity](../../build-your-software-catalog/sync-data-to-catalog/sync-data-to-catalog.md#entity-json-structure) 都有一个专门的页面，其中包含 3 个选项卡(默认情况下) : 

* * [`Overview`](#overview)
* [`Runs`](#runs)
* [`Audit log`](#audit-log)

## 概览

概览 "选项卡由两个部件组成: 

### 详情

在这里，您可以找到实体的属性及其 Values、记分卡及其 Values 以及其他元数据。

#### 相关实体

默认情况下，同一方向上所有直接相关的实体都会自动出现在此 widget 中。 向前相关和向后相关的实体都是如此。 间接相关的实体不会出现。

例如

工作流运行 "与 "工作流 "有正向关系，"工作流 "与 "Microservice "有正向关系，"Microservice "与 "拉取请求 "有**后向**关系。 由于我们中途改变了方向，所以这种关系是**间接**的: 

![builderRelationsExample](../../../static/img/software-catalog/pages/builderRelationsExample.png)

正如您所看到的，在查看某个 "工作流运行 "的实体页面时，"工作流 "和 "微服务 "会自动出现，但 "拉取请求 "不会，因为它的关系在另一个方向上: 

![entityRelationsExample](../../../static/img/software-catalog/pages/entityRelationsExample.png)

#### 新相关实体选项卡

点击 "+ 新标签 "按钮，可以向 "相关实体 "表中添加其他实体。 在对话框中，"相关蓝图 "下拉菜单将显示以任何方式与当前实体相关的所有实体。 在我们上面的 "工作流运行 "示例中，我们可以使用该按钮向我们的 widget 添加 "拉取请求 "标签。

![afterNewTab](../../../static/img/software-catalog/pages/afterNewTab.png)

在某些情况下，相关的蓝图可能可以通过不止一个关系到达，就像这样: 

![multipleRelations](../../../static/img/software-catalog/pages/multipleRelations.png)

假设我们想在 `ServiceInEnv` 的相关实体中添加一个 `集群 ` 标签。 在这种情况下， `相关属性 ` 下拉菜单将显示可能的关系供我们选择: 

![multiplePaths](../../../static/img/software-catalog/pages/multiplePaths.png)

#### 隐藏标签

右侧的 "隐藏选项卡 "按钮可让您控制该 widget 中哪些选项卡可见。

## Runs

如果实体的蓝图配置了任何[actions](/create-self-service-experiences/) ，"运行 "选项卡将显示其历史日志、结果、日志流等。

## 审计日志

此选项卡会显示导致实体配置发生任何更改的所有操作(包括 CRUD)。 对于每项更改，都会显示有用的元数据，如发起者、更改前后的 Diff、相关蓝图等。

## 仪表板小部件

[Visualization widgets](/customize-pages-dashboards-and-plugins/dashboards/) 可添加到实体页面，使用图形元素显示数据。

您可以通过 "添加可视化 "菜单向实体页面添加部件: 

![addVisualizations](../../../static/img/software-catalog/pages/addVisualizations.png)

让我们创建一个简单的数字图表，显示与此 "域 "相关的 "系统 "实体的数量: 

<img src='/img/software-catalog/pages/demoNumberChart.png' width='350rem' />

创建第一个部件后，实体页面中将创建一个名为 "仪表盘 "的新标签，显示新部件: 

![entityAfterVisualization](../../../static/img/software-catalog/pages/entityAfterVisualization.png)

每个附加的可视化都将作为部件添加到 "仪表盘 "选项卡中。

## 其他选项卡

[available property types](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/#supported-properties) 中的某些属性是可视化的。在蓝图中定义这些属性时，与该蓝图相关的每个实体页面都会自动创建一个额外的选项卡，以相关的可视化格式显示该属性的内容。

支持以下属性类型

* * [Markdown](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/markdown)
* [Embedded URL](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/embedded-url)
* [Swagger UI](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/swagger)

## 实体页面操作

每种页面类型都有一套可在用户界面上执行的操作。 下表总结了实体页面的可用操作: 


| Page type   | Save a view | Save view as<br /> a new page | Edit page | Delete page | Lock page |
| ----------- | :---------: | :---------------------------: | :-------: | :---------: | :-------: |
| Entity page |     ✅      |              ❌               |    ❌     |     ❌      |    ✅     |


有关各项操作的更多信息，请参见[page operations](/customize-pages-dashboards-and-plugins/page/catalog-page#page-operations) 。