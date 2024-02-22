# 示例

下面是几个计算属性被用于的例子: 

## 基本计算属性

在下面的示例中，我们将创建一个名为 "MBMemory "的计算属性，其类型为 "数"，然后将其转换为 GB 单位: 

```json showLineNumbers
"properties":{
    "MBMemory":{
        "type": "number"
    },
},
"calculationProperties" : {
    "GBMemory": {
        "title": "GB Memory",
        "type": "number",
        "calculation": ".properties.MBMemory / 1024"
    }
}
```

在这种情况下，如果您有一个 `MBMemory` **string** 属性，其值为

```json showLineNumbers
{
  "MBMemory": 2048
}
```

那么 `GBMemory` 计算属性值将是

```json showLineNumbers
{
  "GBMemory": 2
}
```

下面举例说明如何利用计算属性对被用于到 Port 中的数据进行数学运算。

## 连接字符串

假设有两个 `string` 属性: 一个名为 `str1`，值为 `hello` ；另一个名为 `str2`，值为 `world`。 下面的计算结果将是 `hello world`: 

```json showLineNumbers
{
  "title": "Concatenate strings example",
  "type": "string",
  "calculation": ".properties.str1 + .properties.str2"
}
```

:::tip 如果您想提供自己的字符串模板来连接属性，请用单引号 (`'`)将模板字符串包起来，例如`'https://' + .properties.str1'`

:::

## 计算数组长度

假设您有一个名为 `myArr` 的 `array` 属性，其值为 `["this", "is", "port"]`。

下面的计算结果将是 `3`: 

```json showLineNumbers
{
  "title": "Calculate array length",
  "type": "number",
  "calculation": ".properties.myArr | length"
}
```

## 切片数组

假设你有一个名为 `array1` 的 `array` 属性，其值为 `[1,2,3,4]`，你可以使用下面的切分计算来得到结果 `[2,3,4]`: 

```json showLineNumbers
{
  "title": "Slice array example",
  "type": "string",
  "calculation": ".properties.array1[1:4]"
}
```

## 合并对象

假设有两个 "对象 "属性: 一个名为 `deployed_config`，其值为`{cpu: 200}`；另一个名为 `service_config`，其值为`{memory: 400}`。 可以通过以下计算方法合并这两个对象属性，得到一个统一的配置: 

```json showLineNumbers
"calculationProperties" : {
    "merge_config": {
        "title": "Merge config",
        "type": "object",
        "calculation": ".properties.deployed_config * .properties.service_config",
    }
}
```

结果将是 `{cpu: 200, memory: 400}`。

:::info  对象合并

* 对象合并执行深度合并，导致原始对象中的嵌套键出现在合并后的对象中。
* 如果合并后的一个或多个属性中出现了相同的 "键"，则最后出现的属性的 "键 "将优先于计算中较早出现的属性的 "键"。

例如，假设我们有 2 个 `object` 类型的属性，我们想在它们之间执行深度合并: 

```json showLineNumbers
{
  "obj1": {
    "cpu": 200
  },
  "obj2": {
    "cpu": 400
  }
}
```

如果计算结果为`".properties.obj1 * .properties.obj2"` ，则结果为`{cpu: 400}`、

如果计算结果为`".properties.obj2 * .properties.obj1"` ，则结果为`{cpu: 200}`、

对于合并 YAML 属性，合并行为是相同的，但如果指定 `type: "string` 和 `format: "yaml"` ，结果将是一个 YAML 对象。

:::

## If-else conditions

假设您的服务使用多个软件包，有些服务使用用 Python 编写的软件包，有些服务使用用 Node.js 编写的软件包。

通过被用于 if-else JQ 规则，可以根据语言为每个 packages 指定不同的 URL: 

```json showLineNumbers
"calculationProperties" : {
    "package_manager_url": {
        "title": "Package Link",
        "type": "string",
        "format": "url",
        "calculation": "if .properties.language == \"Python\" then \"https://pypi.org/project/\" + .identifier else \"https://www.npmjs.com/package/\" + .identifier end",
    }
}
```

针对以下实体

```json showLineNumbers
{
  "identifier": "requests",
  "properties": {
    "language": "Python"
  }
}
```

结果将是 `package_manager_url: "https://pypi.org/project/requests"`。

针对以下实体

```json showLineNumbers
{
  "identifier": "axios",
  "properties": {
    "language": "Nodejs"
  }
}
```

结果将是 `package_manager_url: "https://www.npmjs.com/package/axios"`。

## 计算 k8s 标签

您可以在蓝图中创建一个计算属性来显示特定标签。 在节点蓝图中，您可以找到以下属性: 

```json
"properties": {
      ...
      "labels": {
        "type": "object",
        "title": "Labels",
        "description": "Labels of the Node"
      },
```

标签对象看起来就像

```json
{
  "kubernetes.io/metadata.name": "port-k8s-exporter",
  "name": "port-k8s-exporter"
}
```

要显示 `name` 的值，请在同一蓝图中创建一个新的计算属性，然后使用下面的 JQ 计算: 

```json
.properties.labels."name"
```

结果将是显示 `port-k8s-exporter` 的属性。

## 计算云资源标签

假设您的 Blueprint 中有一个属性 `tags`, 您可以使用 JQ 来显示标签的值。

在 Port 的 AWS 输出程序中，您可以找到以下数组: 

```json
"tags": [
      {
        "Value": "authentication-service",
        "Key": "server-application"
      },
      {
        "Value": "1a23-4bc5d-67efg-89k10",
        "Key": "applyId"
      },
      {
        "Value": "0.0.11",
        "Key": "server-version"
      }
    ]
```

要显示 `applyId` 的 Values，请在实体的蓝图中创建一个新的计算属性，并使用下面的 JQ 计算: 

```json
.properties.tags.[] | select(.Key=="applyId") | .Value
```

结果将显示属性 `1a23-4bc5d-67efg-89pk10`。
