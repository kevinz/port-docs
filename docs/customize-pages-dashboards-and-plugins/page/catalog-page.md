---
sidebar_position: 1
---

# 目录页面

目录页面显示从[blueprint](/build-your-software-catalog/define-your-data-model/setup-blueprint/#what-is-a-blueprint) 创建的所有现有[entities](/build-your-software-catalog/sync-data-to-catalog/#creating-entities) 的表格。在本例中，我们可以看到从 `K8s Cluster` 蓝图创建的所有集群实体: 

![Microservice blueprint page](/img/software-catalog/pages/catalogPage.png)

## 页面创建

创建蓝图时，软件目录中会自动生成一个目录页。 您还可以为任何现有蓝图手动创建额外的目录页，并根据需要对其进行自定义。 请继续阅读，了解可用的自定义选项。

要创建新的目录页面，请访问[Software Catalog](https://app.getport.io/services) ，单击左上角的 "+ 新建 "按钮，然后选择 "新建目录页面"。

### 初始过滤器

在某些情况下，实体表可能非常大，导致加载时间过长。 为防止出现这种情况，可以定义过滤器，在 Port 查询数据时(而不是在查询后)解决。 要定义这样的过滤器，请在创建页面时使用 "初始过滤器 "字段: 

<img src='/img/software-catalog/pages/initialFiltersForm.png' width='50%' />

<br/><br/>

您可以用 JSON 格式定义任何[supported rule](/search-and-query/#rules) 。 下面是一个仅显示上个月更新的 "部署 "的示例: 

```json showLineNumbers
[
  {
    "property": "$updatedAt",
    "operator": "between",
    "value": { "preset": "lastMonth" }
  }
]
```

#### 动态滤波器

创建目录页面时，可以使用登录用户的[dynamic properties](/search-and-query/#dynamic-properties) 。

### 不包括的属性

另一种减少加载时间的方法是在查询数据时从实体表中排除不需要的属性。 使用该选项时，新表将不包含排除属性的列。 要做到这一点，请在创建页面时使用 "排除属性 "字段: 

<img src='/img/software-catalog/pages/excludePropertiesForm.png' width='50%' />

## 定制

实体表可以自定义，这将定义用户对 Port 平台的看法。

:::tip 

我们强烈建议使用这些自定义功能，以便为您的开发人员提供简洁、准确的平台视图。

:::

所有表格自定义功能均可在表格顶部栏中使用: 

![Table operations bar](/img/software-catalog/pages/TableOperationsBar.png)

### 过滤器

您可以通过以下菜单对表格进行筛选: 

![Table filter menu marked](/img/software-catalog/pages/TableFilterMenu.png)

您可以定义任何带有合适值的过滤操作符。

您可以过滤一个或多个值，同时用 `And/Or` 设置每个字段之间的关系。

#### "我的团队 "过滤器

通过使用 "我的团队 "过滤器，您只能看到属于您的团队的实体。 这意味着您只能看到您是成员的团队的实体。

该过滤器适用于

* 字符串 "属性，格式为 "team"。
* [meta property](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/meta-properties) `Team`。

![My Teams Filter](/img/software-catalog/pages/MyTeamsFilter.png)

#### `我`过滤器

通过使用 "我 "过滤器，你只能看到属于登录用户的实体。

该过滤器适用于[`User`](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/user) 属性。

![Me Filter](../../../static/img/software-catalog/pages/meFilter.png)

### 排序

您可以通过以下菜单对表格进行排序: 

![Table sort menu marked](/img/software-catalog/pages/TableSortMenu.png)

您可以根据一个或多个任意类型的字段进行排序。

:::tip 要对特定列排序，请单击该列标题。

:::

#### 隐藏

您可以通过以下菜单隐藏表格列: 

![Table hide menu marked](/img/software-catalog/pages/TableHideMenu.png)

您可以决定用户是否可以查看每个字段。

:::tip 我们强烈建议向用户隐藏无关数据，为他们提供一个干净的工作环境，让他们不再分心。

:::

#### 组别

您可以使用以下菜单按实体分组: 

![Table group by menu marked](/img/software-catalog/pages/TableGroupByMenu.png)

您可以根据表格中的任何字段对结果进行分组。

:::tip 如果要为用户创建自定义视图(如 "按 Owner 分类的微服务")，建议使用 "按分组"。

只需创建 "按分组 "设置，必要时添加其他查看设置，然后从自定义视图[save a new page](#saving-new-pages) 。

:::

#### 搜索

Provider 为表格提供了自由文本搜索选项。 这将搜索实体的所有属性，并显示与查询匹配的实体。 如果查询包含多个单词，则将显示包含所有这些单词的实体，即使它们分布在不同的属性中。

![Table search bar marked](/img/software-catalog/pages/TableSearchBar.png)

->[Explore how to control page visibility and permissions](./page-permissions.md)

## 页面操作

页面有一组可以从用户界面执行的操作。

:::info  默认页面 创建新蓝图时会自动创建默认目录页面，该页面与其蓝图直接关联，这意味着如果删除蓝图，默认页面也将被删除。

如果愿意，您仍然可以编辑或删除默认页面。

可以进行筛选、排序、分组，还可以使用表格 widget 控件来更改默认页面的布局。

:::

### 保存视图

在特定页面上进行的每项更改(如筛选或排序)都会启用 "保存此视图 "按钮。 点击该按钮将为所有用户保存新视图。

![Page operations marked](/img/software-catalog/pages/PageOperationsMarked.png)

:::note 为所有用户保存视图的功能仅适用于[Admin role](../../sso-rbac/rbac/rbac.md#roles)

:::

### 保存新页面

每次在页面上进行更改并启用 "保存此视图 "时，都可以按其右侧的小箭头打开一个下拉菜单: 

<center>

![Save view menu button marked](/img/software-catalog/pages/SaveViewDropMenuButton.png)

</center>

点击 "另存为新页面 "按钮后，会弹出一个窗口: 

![Save as a new page popup](/img/software-catalog/pages/SaveAPageForm.png)

#### 选择页面图标

保存新页面或编辑现有页面时，您可以使用一组图标: 

<center>

![Page Icons dropdown menu](/img/software-catalog/pages/PageIcons.png)

</center>

### 编辑、锁定或删除页面

点击右上角的"... "按钮，可以编辑、锁定或删除页面: 

<center>

![Page menu](/img/software-catalog/pages/PageMenu.png)

</center>

#### 编辑页面

编辑页面可以更改页面名称和/或图标: 

![Edit Page popup window](/img/software-catalog/pages/EditPageForm.png)

#### 锁定页面

锁定目录页面会禁用隐藏列或应用筛选器修改显示数据的选项。

锁定页面可以让你根据开发人员的需求对页面进行特别策划，确保他们无法修改视图或查看与他们无关的数据。

要了解如何锁定页面，请参阅[page permissions](./page-permissions.md#lock-pages) 。

#### 删除网页

点击 "删除页面 "按钮可删除任何页面(无论是自动创建还是手动创建)。

:::warning  默认页面 从门户中删除蓝图时，与该蓝图关联的所有页面(包括为其创建的默认页 面)也将被删除。

:::
