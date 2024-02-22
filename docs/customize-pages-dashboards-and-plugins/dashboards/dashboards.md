# 仪表板小部件

Port 支持小部件形式的各种可视化功能，让您可以使用图形元素显示软件目录中的数据，从而更轻松地理解大型数据集。

仪表板可在以下位置使用: 

1. Port 应用程序的[Home page](https://app.getport.io/organization/home) - 主页本身就是一个仪表盘，允许您添加和自定义本页所述的任何部件。
2. 每个[entity page](/customize-pages-dashboards-and-plugins/page/entity-page#dashboard-widgets) 都可以有一个 "仪表盘 "标签，上面有自己的部件。
3. [software catalog](https://app.getport.io/services) 允许您创建可定制的[dashboard pages](/customize-pages-dashboards-and-plugins/page/dashboard-page) 。

## 小工具类型

### 饼图

您可以创建一个饼图，按照特定实体页面内的类别和实体属性来说明软件目录中的实体数据[**specific entity page**](../page/entity-page.md) 。

![Pie Chart](/img/software-catalog/widgets/pieChartExample.png)

#### 可视化属性


| Field                   | Type     | Description                                                                                                                  | Default | Required |
| ----------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------- | ------- | -------- |
| `Title`                 | `String` | Pie chart title                                                                                                              | `null`  | `true`   |
| `Icon`                  | `String` | Pie chart Icon                                                                                                               | `null`  | `false`  |
| `Description`           | `String` | Pie chart description                                                                                                        | `null`  | `false`  |
| `Blueprint`             | `String` | The chosen blueprint from which related entities data is visualized                                                          | `null`  | `true`   |
| `Breakdown by property` | `String` | Group your chart by a specific property                                                                                      | `null`  | `true`   |
| `Filters`               | `Array`  | Filters to include or exclude specific data based on Port's [Search Rules](../../search-and-query/search-and-query.md#rules) | []      | `false`  |


### Number chart

您可以从[**specific entity page**](../page/entity-page.md) 中的相关实体创建数字图表可视化。您既可以对实体进行计数，也可以对数字属性执行聚合函数。 您还可以过滤实体，使聚合数字图表只应用于有限的实体集，并使用 Port 的[Search Rules](/search-and-query/search-and-query.md#rules)

![Number Chart](../../../static/img/software-catalog/widgets/numberChartExample.png)

#### 数字图表属性


| Field             | Type     | Description                                                                                                                                                                                                                                 | Default    | Required |
| ----------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | -------- |
| `Title`           | `String` | Number Chart title                                                                                                                                                                                                                          | `null`     | `true`   |
| `Icon`            | `String` | Number Chart Icon                                                                                                                                                                                                                           | `null`     | `false`  |
| `Description`     | `String` | Number Chart description                                                                                                                                                                                                                    | `null`     | `false`  |
| `Blueprint`       | `String` | The chosen blueprint from which related entities data is visualized from                                                                                                                                                                    | `null`     | `true`   |
| `Calculate By`    | `String` | Aggregate by either counting the entities or perform a function on a property. Possible values: `entities`, `property`                                                                                                                      | `entities` | `true`   |
| `Property`        | `String` | The number chart value will be the selected property's aggregated value (according to the chosen function). The `property` key is only available when `Calculate By` is equal to `property`                                                 | `null`     | `true`   |
| `Function`        | `String` | In case `Calculate By` is equal to `property` the options are: sum, min, max, average and median. <br/> In case `Calculate By` is equal to `entities` the options are: count and average.                                                   | `null`     | `true`   |
| `Average of`      | `String` | In case `Calculate By` is equal to `entities` the options are: hour, day ,week and month. <br/> In case `Calculate By` is equal to `property` the options are: hour, day, week, month and total (divide the sum by the number of entities). | `null`     | `true`   |
| `Measure time by` | `String` | Used to specify an alternative property to use as the time property for the average calculation instead of the default field which is `createdAt`.                                                                                          | `null`     | `false`  |
| `Filters`         | `Array`  | Filters to include or exclude specific data based on Port's [search rules](../../search-and-query/search-and-query.md#rules)                                                                                                                | []         | `false`  |
| `unit`            | `String` | The unit of the number chart. Possible Values: `%`, `$`, `£`, `€`, `none`, `custom`                                                                                                                                                         | `null`     | `true`   |
| `unitCustom`      | `String` | Text to display below the number value. The `unitCustom` key is only available when `unit` equals to `custom`                                                                                                                               | `null`     | `true`   |
| `unitAlignment`   | `String` | `left`, `right`, `bottom`.                                                                                                                                                                                                                  | `null`     | `true`   |


:::note 在计算平均时间间隔(如按小时、天、周或月)时，必须注意任何部分时间间隔都被视为一个完整的时间间隔。 这种方法可确保不同时间单位的一致性。

例如，如果数据集包含跨越 2 小时 20 分钟的信息，但选择的平均时间范围是 "小时"，那么求和值将除以 3 小时。

:::

### Markdown

通过该 widget，您可以以格式化的形式显示任何标记符内容: 

<img src='/img/software-catalog/widgets/markdownWidget.png' width='500rem' />

#### Markdown widget 属性


| Field      | Type     | Description           | Default | Required |
| ---------- | -------- | --------------------- | ------- | -------- |
| `Title`    | `String` | Markdown widget title | `null`  | `true`   |
| `Icon`     | `String` | Markdown widget Icon  | `null`  | `false`  |
| `markdown` | `String` | Markdown content      | `null`  | `false`  |


### Iframe 可视化

您可以创建 iframe widget，以便在仪表盘中显示嵌入的 url。 iframe widget 对于显示外部仪表盘或其他外部内容非常有用。 它还会在 iframe URL 查询参数中附加实体标识符和蓝图标识符，以便嵌入的页面可以将其用于各种用途。

实体标识符将连接到`entity`查询参数下，蓝图标识符将连接到`blueprint`查询参数下。 例如: `https://some-iframe-url.com?entity=entity_identifier&amp;blueprint=blueprint_identifier`。

![iFrame](../../../static/img/software-catalog/widgets/iframeWidget.png)

#### Widget 属性


| Field               | Type           | Description                                                                                                                                            | Default | Required |
| ------------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------- | -------- |
| `Title`             | `String`       | Iframe widget title                                                                                                                                    | `null`  | `true`   |
| `Icon`              | `String`       | Iframe widget Icon                                                                                                                                     | `null`  | `false`  |
| `Description`       | `String`       | Iframe widget description                                                                                                                              | `null`  | `false`  |
| `URL`               | `String`       | Iframe widget url                                                                                                                                      | `null`  | `false`  |
| `URL type`          | `String`       | `public` or `protect`                                                                                                                                  | `null`  | `false`  |
| `Authorization Url` | `URL String`   | If the `URL type` is `protected` this will be required. Read more about it [here](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/embedded-url/authentication/#authentication-code-flow--pkce) | `null`  | `false`  |
| `clientId`          | `String`       | If the `URL type` is `protected` this will be required. Read more about it [here](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/embedded-url/authentication/#authentication-code-flow--pkce) | `null`  | `false`  |
| `Scopes`            | `String Array` | If the `URL type` is `protected` this will be required. Read more about it [here](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/embedded-url/authentication/#authentication-code-flow--pkce) | `null`  | `false`  |
| `Token URL`         | `URL String`   | If the `URL type` is `protected` this will be required. Read more about it [here](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/embedded-url/authentication/#authentication-code-flow--pkce) | `null`  | `false`  |


#### 表

使用该窗口小部件，您可以根据选定的蓝图创建显示所有实体的表格。您可以使用窗口小部件中的相应按钮，随心所欲地创建表格[searched, filtered and customized](/customize-pages-dashboards-and-plugins/page/catalog-page#customization) 。

<img src='/img/software-catalog/widgets/tableExample.png' width='400rem' />

#### 定制

与目录页面一样，表格也支持以下自定义选项: 

* * [Initial filters](/customize-pages-dashboards-and-plugins/page/catalog-page/#initial-filters)
* [Excluded properties](/customize-pages-dashboards-and-plugins/page/catalog-page/#excluded-properties)

#### 行动运行

通过该小工具，您可以在门户网站上创建一个表格，显示[self-service action](/create-self-service-experiences) 过去的所有运行情况。该表格将自动显示每次运行的相关数据，包括状态、输入参数、执行用户等。

<img src='/img/software-catalog/widgets/actionRunsTableExample.png' width='100%' />

## 图表过滤器

[Pie charts](#pie-chart)、[number charts](#number-chart) 和[tables](#table) 支持筛选器，可从中包含或排除特定数据。筛选器基于 Port 的[Search Rules](../../search-and-query/search-and-query.md#rules) ，并在创建 widget 时设置: 

<img src='/img/software-catalog/widgets/widgetFilterForm.png' width='400rem' />

### 过滤器示例: 只显示上周的部署实体

假设我们有一个名为 "Service "的[blueprint](/build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md) ，它与另一个名为 "Deployment "的蓝图相关，我们希望在此服务上周部署的基础上创建可视化。

为了实现这种理想状态，我们可以进入 "服务 "的某个配置文件页面并创建一个新的可视化。 在下拉菜单中选择 "部署 "蓝图后，我们可以在 "过滤器 "数组中添加以下过滤器: 

```json showLineNumbers
[
  {
    "property": "$createdAt",
    "operator": "between",
    "value": {
      "preset": "lastWeek"
    }
  }
]
```

### 动态过滤器

筛选小工具时，可以使用登录用户的[dynamic properties](/search-and-query/#dynamic-properties) 。