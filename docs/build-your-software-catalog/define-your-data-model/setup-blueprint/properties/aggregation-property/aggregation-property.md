---

sidebar_position: 17
description: 聚合属性聚合属性允许你根据目录中的关系计算指标

---

import ApiRef from "../../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 🧬 聚合特性

聚合属性可让您根据目录中的[relations](/build-your-software-catalog/define-your-data-model/relate-blueprints/#what-is-a-relation) 计算指标。

使用聚合属性可以查看相关实体的相关指标，而无需手动计算。

聚合可在以任何方式(直接、间接、上游或下游)与当前蓝图相关的任何蓝图上执行。

## 何时使用聚合属性？

聚合属性最好定义在目录中抽象层**较高的蓝图上。

这些蓝图通常与目录中的许多其他蓝图**相关，因此最能从聚合属性中受益。

## 💡 常用聚合 Usage

例如，如果您有一个微服务蓝图，您可以对其定义聚合属性，以便根据相关实体计算指标，如

* 与微服务相关的开放式 jira 问题数量。
* 与微服务相关的未解决的 "危 "和 "高 "漏洞数量。
* 上周的平均部署频率。
* 上个月的构建成功率。

聚合属性可让您根据相关实体的指标指定记分卡和倡议规则。

:::tip 例如，如果您有一个微服务蓝图，并有相关的警报蓝图，您可以定义一条规则，检查与每个微服务相关的打开的 CRITICAL 和 HIGH 警报的数量是否大于 0。

:::

## 规格

aggregationProperties "键是实体 JSON 中的顶级键(类似于 "identifier"、"title"、"properties "等)。

聚合属性支持按实体或按属性进行计算。

* 按**实体**计算是对符合查询条件的实体进行计算(例如，计算符合查询条件的实体数量)。
* 按**属性**计算是对符合查询条件的实体的属性进行计算(例如，对符合查询条件的实体的属性值求和)。

## 定义

<Tabs groupId="api-definition" queryString defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "tf"},
{label: "Pulumi", value: "pulumi"}
]}>

<TabItem value="api">

### 按实体计算

被引用的实体计算用于根据与查询匹配的实体计算指标。

支持的方法

* `count` - 计算与查询匹配的实体数量。例如，计算与微服务相关的打开 Jira 问题的数量。
* `average` - 计算每个定义时间段内实体的平均值。例如，计算每周的平均部署频率。

<Tabs groupId="CalculateByEntities" defaultValue="Entities Count">

<TabItem value="Entities Count">

在这个示例中，我们有一个微服务蓝图，我们想计算与每个微服务相关的 Jira 开放问题的数量。

```json
{
  "aggregationProperties": {
    "numberOfOpenJiraIssues": {
      "title": "Number of open Jira issues",
      "target": "jiraIssue",
      "calculationSpec": {
        "calculationBy": "entities",
        "func": "count"
      },
      "query": {
        "combinator": "and",
        "rules": [
          {
            "property": "status",
            "operator": "=",
            "value": "OPEN"
          }
        ]
      }
    }
  }
}
```

aggregationProperties "包含一个名为 "numberOfOpenJiraIssues "的键，它是我们要定义的聚合属性的标识符。

* `title` - 聚合属性的 title(标题)。
* `target` - 我们要聚合数据的蓝图。
* `query` - **可选** - 将在目标蓝图上执行的查询。该查询基于筛选器，可根据 Port 的 "数据 "属性包含或排除特定数据。[Search Rules](../../../../../search-and-query/search-and-query.md#rules)
* `calculationSpec` - 计算规范。
    - calculationBy": "entities"` - 计算将在与查询匹配的 ** 实体上进行。
    - `"func": "count"` - 是我们要在计算中引用的函数。

</TabItem>

<TabItem value="Entities Average">

在这个例子中，我们有一个微服务蓝图，我们想计算每个微服务的平均部署频率。

```json
{
  "aggregationProperties": {
    "averageDeploymentFrequency": {
      "title": "Average deployment frequency",
      "target": "deployment",
      "calculationSpec": {
        "calculationBy": "entities",
        "func": "average",
        "averageOf": "week",
        "measureTimeBy": "$createdAt"
      },
      "query": {
        "combinator": "and",
        "rules": [
          {
            "property": "status",
            "operator": "=",
            "value": "SUCCESS"
          }
        ]
      }
    }
  }
}
```

aggregationProperties "包含一个名为 "averageDeploymentFrequency "的键，它是我们要定义的聚合属性的标识符。

* `title` - 聚合属性的 title(标题)。
* `target` - 我们要聚合数据的蓝图。
* `query` - **可选** - 将在目标蓝图上执行的查询。该查询基于筛选器，可根据 Port 的 "数据 "属性包含或排除特定数据。[Search Rules](../../../../../search-and-query/search-and-query.md#rules)
* `calculationSpec` - 计算规范。
    - calculationBy": "entities"` - 将对符合查询条件的实体进行计算(例如，计算符合查询条件的实体数量)。
    - `"func": "average"` - 是我们要用于计算的函数。
    - `"averageOf": "week"` - 我们要计算平均值的时间段。在本例中，我们要计算每周的平均部署频率。支持的选项有小时、天、周和月
    - `"measureTimeBy": "$createdAt"` - 我们要计算平均值的时间属性。默认情况下，您可以使用目标蓝图中的任何日期属性，我们会将 $createdAt 和 $updatedAt 作为[meta-properties](../meta-properties.md) 添加到每个实体中。

</TabItem>

</Tabs>

#### 按属性计算

Calculate by property(按属性计算)被用来根据与查询匹配的实体的属性计算指标。

属性类型必须是数字。

支持的方法

* `sum` - 将符合查询条件的实体的属性值相加。例如，与微服务相关的开放式 Jira 问题的故事点总和。
* `average` - 计算与查询匹配的实体属性的平均值。例如，微服务的平均 CPU 使用率。
* `min` - 计算与查询匹配的实体属性的最小值。例如，微服务上周的最低警报严重性。
* `max` - 计算与查询匹配的实体属性的最大值。例如，上周微服务的最高警报严重性。
* `median` - 计算与查询匹配的实体属性的中值。例如，微服务最近一周的 CPU 使用率中值。

<Tabs groupId="CalculateByProperty" defaultValue="Sum">

<TabItem value="Sum">

在这个示例中，我们有一个微服务蓝图，我们想计算与每个微服务相关的开放式 Jira 问题的故事点总和。

```json
{
  "aggregationProperties": {
    "sumOfStoryPoints": {
      "title": "Sum of story points",
      "target": "jiraIssue",
      "calculationSpec": {
        "calculationBy": "property",
        "func": "sum",
        "property": "storyPoints"
      },
      "query": {
        "combinator": "and",
        "rules": [
          {
            "property": "status",
            "operator": "=",
            "value": "OPEN"
          }
        ]
      }
    }
  }
}
```

聚合属性 "包含一个名为 "sumOfStoryPoints "的键，它是我们要定义的聚合属性的标识符。

* `title` - 聚合属性的 title(标题)。
* `target` - 我们要聚合数据的蓝图。
* `query` - **可选** - 将在目标蓝图上执行的查询。该查询基于筛选器，可根据 Port 的 "数据 "属性包含或排除特定数据。[Search Rules](../../../../../search-and-query/search-and-query.md#rules)
* `calculationSpec` - 计算规范。
    - calculationBy": "property"` - 将对符合查询条件的实体的属性进行计算(例如，对符合查询条件的实体的属性值求和)。
    - `"func": "sum"` 是我们要用于计算的函数。
    - `"property": "storyPoints"` - 我们要计算总和的属性。属性类型必须是数字。

</TabItem>

<TabItem value="Average">

在这个例子中，我们有一个微服务蓝图，想计算每个微服务的平均 CPU 使用率。

```json
{
  "aggregationProperties": {
    "averageCpuUsage": {
      "title": "Average CPU usage",
      "target": "deployment",
      "calculationSpec": {
        "calculationBy": "property",
        "func": "average",
        "property": "cpuUsage",
        "averageOf": "week",
        "measureTimeBy": "$createdAt"
      },
      "query": {
        "combinator": "and",
        "rules": [
          {
            "property": "status",
            "operator": "=",
            "value": "SUCCESS"
          }
        ]
      }
    }
  }
}
```

`aggregationProperties` 包含一个名为 `averageCpuUsage` 的键，它是我们要定义的聚合属性的标识符。

* `title` - 聚合属性的 title(标题)。
* `target` - 我们要聚合数据的蓝图。
* `query` - **可选** - 将在目标蓝图上执行的查询。该查询基于筛选器，可根据 Port 的 "数据 "属性包含或排除特定数据。[Search Rules](../../../../../search-and-query/search-and-query.md#rules)
* `calculationSpec` - 计算规范。
    - `calculationBy` - `property` 将对符合查询的实体的属性进行计算(例如，平均符合查询的实体的属性值)。
    - `"func": "average"` - 是我们要用于计算的函数。
    - `"property": "cpuUsage"` - 我们要计算平均值的属性。属性类型必须是数字。
    - `"averageOf": "week"` - 我们要计算平均值的时间段。在本例中，我们要计算每周的平均部署频率。支持的选项有小时、天、周、月和总计
    - `"measureTimeBy": "$createdAt"` - 我们要测量平均值的时间属性。默认情况下，您可以使用目标蓝图中的任何日期属性，我们会将 $createdAt 和 $updatedAt 作为[meta-properties](../meta-properties.md) 添加到每个实体中。

</TabItem>

<TabItem value="Min">

在这个示例中，我们有一个微服务蓝图，想计算上周每个微服务的警报严重性最小值。

```json
{
  "aggregationProperties": {
    "minAlertSeverity": {
      "title": "Minimum alert severity",
      "target": "alert",
      "calculationSpec": {
        "calculationBy": "property",
        "func": "min",
        "property": "severity"
      },
      "query": {
        "combinator": "and",
        "rules": [
          {
            "property": "status",
            "operator": "=",
            "value": "OPEN"
          },
          {
            "property": "$createdAt",
            "operator": "between",
            "value": {
              "preset": "lastWeek"
            }
          }
        ]
      }
    }
  }
}
```

`aggregationProperties` 包含一个名为 `minAlertSeverity` 的键，它是我们要定义的聚合属性的标识符。

* `title` - 聚合属性的 title(标题)。
* `target` - 我们要聚合数据的蓝图。
* `query` - **可选** - 将在目标蓝图上执行的查询。该查询基于筛选器，可根据 Port 的 "数据 "属性包含或排除特定数据。[Search Rules](../../../../../search-and-query/search-and-query.md#rules)
* `calculationSpec` - 计算规范。
    - `"calculationBy": "property"` 将对符合查询条件的实体的属性执行计算(例如，符合查询条件的实体的属性的最小值)。
    - `"func": "min"` - 是我们要用于计算的函数。
    - `"property": "severity"` - 我们要计算最小值的属性。属性类型必须是数字。

</TabItem>

<TabItem value="Max">

在这个示例中，我们有一个微服务蓝图，想计算上周每个微服务的警报严重性最大值。

```json
{
  "aggregationProperties": {
    "maxAlertSeverity": {
      "title": "Maximum alert severity",
      "target": "alert",
      "calculationSpec": {
        "calculationBy": "property",
        "func": "max",
        "property": "severity"
      },
      "query": {
        "combinator": "and",
        "rules": [
          {
            "property": "status",
            "operator": "=",
            "value": "OPEN"
          },
          {
            "property": "$createdAt",
            "operator": "between",
            "value": {
              "preset": "lastWeek"
            }
          }
        ]
      }
    }
  }
}
```

`aggregationProperties` 包含一个名为 `maxAlertSeverity` 的键，它是我们要定义的聚合属性的标识符。

* `title` - 聚合属性的 title(标题)。
* `target` - 我们要聚合数据的蓝图。
* `query` - **可选** - 将在目标蓝图上执行的查询。该查询基于筛选器，可根据 Port 的 "数据 "属性包含或排除特定数据。[Search Rules](../../../../../search-and-query/search-and-query.md#rules)
* `calculationSpec` - 计算规范。
    - `"calculationBy": "property"` 将对符合查询条件的实体的属性执行计算(例如，符合查询条件的实体的属性的最大值)。
    - `"func": "max"` - 是我们要用于计算的函数。
    - `"property": "severity"` - 我们要计算最大值的属性。属性类型必须是数字。

</TabItem>

<TabItem value="Median">

在这个例子中，我们有一个微服务蓝图，我们想计算每个微服务的所有相关 pod 的 cpu 使用率中值。

```json
{
  "aggregationProperties": {
    "medianCpuUsage": {
      "title": "Median CPU usage",
      "target": "pod",
      "calculationSpec": {
        "calculationBy": "property",
        "func": "median",
        "property": "cpuUsage"
      },
      "query": {
        "combinator": "and",
        "rules": [
          {
            "property": "status",
            "operator": "=",
            "value": "OPEN"
          },
          {
            "property": "$createdAt",
            "operator": "between",
            "value": {
              "preset": "lastWeek"
            }
          }
        ]
      }
    }
  }
}
```

`aggregationProperties` 包含一个名为 `medianCpuUsage` 的键，它是我们要定义的聚合属性的标识符。

* `title` - 聚合属性的 title(标题)。
* `target` - 我们要聚合数据的蓝图。
* `query` - **可选** - 将在目标蓝图上执行的查询。该查询基于筛选器，可根据 Port 的 "数据 "属性包含或排除特定数据。[Search Rules](../../../../../search-and-query/search-and-query.md#rules)
* `calculationSpec` - 计算规范。
    - `"calculationBy": "property"` 将对符合查询条件的实体的属性执行计算(例如，符合查询条件的实体的属性中值)。
    - `"func": "median"` 是我们要用于计算的函数。
    - `"property": "cpuUsage"` - 我们要计算中值的属性。属性类型必须是数字。

</TabItem>

</Tabs>

</TabItem>

<TabItem value="tf">

### 按实体计算

被引用的实体计算用于根据与查询匹配的实体计算指标。

支持的方法

* count_entities` - 计算与查询匹配的实体数量。例如，计算与微服务相关的打开 Jira 问题的数量。
* `average_entities` - 计算每个定义时间段内实体的平均数。例如，计算每周的平均部署频率。

<Tabs groupId="CalculateByEntities" queryString defaultValue="Entities Count">

<TabItem value="Entities Count">

在这个示例中，我们创建了一个父蓝图和一个子蓝图，并创建了一个聚合属性来计算父蓝图中的子蓝图: 

```hcl
resource "port_blueprint" "parent_blueprint" {
  title       = "Parent Blueprint"
  icon        = "Terraform"
  identifier  = "parent"
  description = ""
  properties = {
    number_props = {
      "age" = {
        title = "Age"
      }
    }
  }
}

resource "port_blueprint" "child_blueprint" {
  title       = "Child Blueprint"
  icon        = "Terraform"
  identifier  = "child"
  description = ""
  properties = {
    number_props = {
      "age" = {
        title = "Age"
      }
    }
  }
  relations = {
    "parent" = {
      title  = "Parent"
      target = port_blueprint.parent_blueprint.identifier
    }
  }
}

resource "port_aggregation_properties" "parent_aggregation_properties" {
  blueprint_identifier = port_blueprint.parent_blueprint.identifier
  properties           = {
    count_kids = {
      target_blueprint_identifier = port_blueprint.child_blueprint.identifier
      title                       = "Count Kids"
      icon                        = "Terraform"
      description                 = "Count Kids"
      method                      = {
        count_entities = true
      }
    }
  }
}
```

</TabItem>

<TabItem value="Entities Average">

在这个例子中，我们有一个微服务蓝图，我们想计算每个微服务的平均部署频率。

```hcl
resource "port_blueprint" "microservice_blueprint" {
  title       = "Microservice Blueprint"
  icon        = "Terraform"
  identifier  = "microservice"
  description = ""
}

resource "port_blueprint" "deployment_blueprint" {
  title       = "Deployment Blueprint"
  icon        = "Terraform"
  identifier  = "deployment"
  description = ""
  relations = {
    "microservice" = {
      title  = "Microservice"
      target = port_blueprint.microservice_blueprint.identifier
    }
  }
}

resource "port_aggregation_properties" "microservice_aggregation_properties" {
  blueprint_identifier = port_blueprint.microservice_blueprint.identifier
  properties           = {
    average_deployment_frequency = {
      target_blueprint_identifier = port_blueprint.deployment_blueprint.identifier
      title                       = "Average deployment frequency"
      icon                        = "Terraform"
      description                 = "Average deployment frequency"
      method                      = {
        average_entities = {
          average_of      = "week"
          measure_time_by = "$createdAt"
        }
      }
    }
  }
}
```

</TabItem>

</Tabs>

#### 按属性计算

Calculate by property(按属性计算)被用来根据与查询匹配的实体的属性计算指标。

属性类型必须是数字。

支持的方法

* `average_by_property` - 计算与查询匹配的实体的平均属性。例如，微服务的平均 CPU Usage。
* `aggregate_by_property` - 聚合与查询匹配的实体的属性值，支持的聚合函数有: sum、min、max、median`。例如，汇总与微服务相关的开放式 Jira 问题的故事点。

<Tabs groupId="CalculateByProperty" queryString defaultValue="Sum">

<TabItem value="Sum">

在这个示例中，我们有一个微服务蓝图，我们想计算与每个微服务相关的开放式 Jira 问题的故事点总和。

```hcl
resource "port_blueprint" "microservice_blueprint" {
  title       = "Microservice Blueprint"
  icon        = "Terraform"
  identifier  = "microservice"
  description = ""
}

resource "port_blueprint" "jira_issue_blueprint" {
  title       = "Jira Issue Blueprint"
  icon        = "Terraform"
  identifier  = "jira_issue"
  description = ""
  properties = {
    number_props = {
      "storyPoints" = {
        title = "Story Points"
      }
    }
  }
  relations = {
    "microservice" = {
      title  = "Microservice"
      target = port_blueprint.microservice_blueprint.identifier
    }
  }
}

resource "port_aggregation_properties" "microservice_aggregation_properties" {
  blueprint_identifier = port_blueprint.microservice_blueprint.identifier
  properties           = {
    sum_of_story_points = {
      target_blueprint_identifier = port_blueprint.jira_issue_blueprint.identifier
      title                       = "Sum of story points"
      icon                        = "Terraform"
      description                 = "Sum of story points"
      method                      = {
        aggregate_by_property = {
          func     = "sum"
          property = "storyPoints"
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="Average">

在这个例子中，我们有一个微服务蓝图，想计算每个微服务的平均 CPU 使用率。

```hcl
resource "port_blueprint" "microservice_blueprint" {
  title       = "Microservice Blueprint"
  icon        = "Terraform"
  identifier  = "microservice"
  description = ""
}

resource "port_blueprint" "deployment_blueprint" {
  title       = "Deployment Blueprint"
  icon        = "Terraform"
  identifier  = "deployment"
  description = ""
  properties = {
    number_props = {
      "cpuUsage" = {
        title = "CPU Usage"
      }
    }
  }
  relations = {
    "microservice" = {
      title  = "Microservice"
      target = port_blueprint.microservice_blueprint.identifier
    }
  }
}

resource "port_aggregation_properties" "microservice_aggregation_properties" {
  blueprint_identifier = port_blueprint.microservice_blueprint.identifier
  properties           = {
    average_cpu_usage = {
      target_blueprint_identifier = port_blueprint.deployment_blueprint.identifier
      title                       = "Average CPU usage"
      icon                        = "Terraform"
      description                 = "Average CPU usage"
      method                      = {
        average_by_property = {
          average_of      = "week"
          measure_time_by = "$createdAt"
          property        = "cpuUsage"
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="Min">

在这个示例中，我们有一个微服务蓝图，想计算上周每个微服务的警报严重性最小值。

```hcl
resource "port_blueprint" "microservice_blueprint" {
  title       = "Microservice Blueprint"
  icon        = "Terraform"
  identifier  = "microservice"
  description = ""
}

resource "port_blueprint" "alert_blueprint" {
  title       = "Alert Blueprint"
  icon        = "Terraform"
  identifier  = "alert"
  description = ""
  properties = {
    number_props = {
      "severity" = {
        title = "Severity"
      }
    }
  }
  relations = {
    "microservice" = {
      title  = "Microservice"
      target = port_blueprint.microservice_blueprint.identifier
    }
  }
}

resource "port_aggregation_properties" "microservice_aggregation_properties" {
  blueprint_identifier = port_blueprint.microservice_blueprint.identifier
  properties           = {
    min_alert_severity = {
      target_blueprint_identifier = port_blueprint.alert_blueprint.identifier
      title                       = "Minimum alert severity"
      icon                        = "Terraform"
      description                 = "Minimum alert severity"
      method                      = {
        aggregate_by_property = {
          func     = "min"
          property = "severity"
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="Max">

在这个示例中，我们有一个微服务蓝图，想计算上周每个微服务的警报严重性最大值。

```hcl
resource "port_blueprint" "microservice_blueprint" {
  title       = "Microservice Blueprint"
  icon        = "Terraform"
  identifier  = "microservice"
  description = ""
}

resource "port_blueprint" "alert_blueprint" {
  title       = "Alert Blueprint"
  icon        = "Terraform"
  identifier  = "alert"
  description = ""
  properties = {
    number_props = {
      "severity" = {
        title = "Severity"
      }
    }
  }
  relations = {
    "microservice" = {
      title  = "Microservice"
      target = port_blueprint.microservice_blueprint.identifier
    }
  }
}

resource "port_aggregation_properties" "microservice_aggregation_properties" {
  blueprint_identifier = port_blueprint.microservice_blueprint.identifier
  properties           = {
    max_alert_severity = {
      target_blueprint_identifier = port_blueprint.alert_blueprint.identifier
      title                       = "Maximum alert severity"
      icon                        = "Terraform"
      description                 = "Maximum alert severity"
      method                      = {
        aggregate_by_property = {
          func     = "max"
          property = "severity"
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="Median">

在这个例子中，我们有一个微服务蓝图，我们想计算每个微服务的所有相关 pod 的 cpu 使用率中值。

```hcl
resource "port_blueprint" "microservice_blueprint" {
  title       = "Microservice Blueprint"
  icon        = "Terraform"
  identifier  = "microservice"
  description = ""
}

resource "port_blueprint" "pod_blueprint" {
  title       = "Pod Blueprint"
  icon        = "Terraform"
  identifier  = "pod"
  description = ""
  properties = {
    number_props = {
      "cpuUsage" = {
        title = "CPU Usage"
      }
    }
  }
  relations = {
    "microservice" = {
      title  = "Microservice"
      target = port_blueprint.microservice_blueprint.identifier
    }
  }
}

resource "port_aggregation_properties" "microservice_aggregation_properties" {
  blueprint_identifier = port_blueprint.microservice_blueprint.identifier
  properties           = {
    median_cpu_usage = {
      target_blueprint_identifier = port_blueprint.pod_blueprint.identifier
      title                       = "Median CPU usage"
      icon                        = "Terraform"
      description                 = "Median CPU usage"
      method                      = {
        aggregate_by_property = {
          func     = "median"
          property = "cpuUsage"
        }
      }
    }
  }
}
```

</TabItem>

</Tabs>

#### 在聚合属性中使用查询

您可以使用查询来筛选要执行计算的实体。

该查询以过滤器为基础，根据 Port 的[Search Rules](../../../../../search-and-query/search-and-query.md#rules)

#### 查询示例

创建一个版本库蓝图和一个拉取请求蓝图，并创建一个聚合属性来计算平均每月的修复拉取请求: 

为此，我们将在聚合属性中添加一个查询，以便只过滤具有固定 title 的拉取请求: 

```hcl
resource "port_blueprint" "repository_blueprint" {
  title       = "Repository Blueprint"
  icon        = "Terraform"
  identifier  = "repository"
  description = ""
}

resource "port_blueprint" "pull_request_blueprint" {
  title       = "Pull Request Blueprint"
  icon        = "Terraform"
  identifier  = "pull_request"
  description = ""
  properties = {
    string_props = {
      "status" = {
        title = "Status"
      }
    }
  }
  relations = {
    "repository" = {
      title  = "Repository"
      target = port_blueprint.repository_blueprint.identifier
    }
  }
}

resource "port_aggregation_properties" "repository_aggregation_properties" {
  blueprint_identifier = port_blueprint.repository_blueprint.identifier
  properties           = {
    fix_pull_requests_count = {
      target_blueprint_identifier = port_blueprint.pull_request_blueprint.identifier
      title                       = "Pull Requests Per Day"
      icon                        = "Terraform"
      description                 = "Pull Requests Per Day"
      method                      = {
        average_entities = {
          average_of      = "month"
          measure_time_by = "$createdAt"
        }
      }
      query = jsonencode(
        {
          "combinator" : "and",
          "rules" : [
            {
              "property" : "$title",
              "operator" : "ContainsAny",
              "value" : ["fix", "fixed", "fixing", "Fix"]
            }
          ]
        }
      )
    }
  }
}
```

</TabItem>

<TabItem value="pulumi">

## 即将推出...

</TabItem>

</Tabs>

### 限制

蓝图中所有实体的聚合属性结果将每 15 分钟重新计算一次。