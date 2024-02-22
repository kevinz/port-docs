---

title: 用户输入

---

import DocCardList from '@theme/DocCardList';

# 用户输入

每个操作的定义中都有一个 "userInputs "部分，在这部分中，您可以定义您希望开发人员和用户在调用操作时填写的所有用户输入信息。

## 结构

```json showLineNumbers
{
  "properties": {
    "myInput": {
      "title": "My input",
      "icon": "My icon",
      "description": "My input",
      "type": "input_type"
    }
  },
  "required": ["myInput"]
}
```

下表列出了构成基本用户输入定义的不同组件: 


| Field         | Description                                                                                                                                                                                             |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `title`       | Input title                                                                                                                                                                                             |
| `type`        | **Mandatory field.** The data type of the input.                                                                                                                                                        |
| `icon`        | Icon for the input <br /><br />See the [full icon list](../../../build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md#full-icon-list).                                 |
| `description` | Description of the input.<br /> This value is visible to users when hovering on the info icon in the UI. It provides detailed information about the use of a specific input or the way it will be used. |
| `default`     | Default value for this input in case the self-service action will be executed without explicitly providing a value.                                                                                     |


:::tip  属性结构 输入名称是输入对象的键。 例如，在上面的代码块中，输入名称是 `myInput` 。

请注意，所有可用于 Port 蓝图的[properties](../../../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/properties.md#supported-properties) 也可被用作用户输入，这就是为什么它们遵循相同的结构。

:::

## 支持的用户输入

<DocCardList />

## 排序用户输入

您可以使用 `order` 字段定义用户界面中显示用户输入的顺序。 该字段是一个包含用户输入名称的数组。

```json showLineNumbers
{
  "properties": {
    "myInput1": {
      "title": "My input 1",
      "icon": "My icon 1",
      "description": "My input 1",
      "type": "input_type"
    },
    "myInput2": {
      "title": "My input 2",
      "icon": "My icon 2",
      "description": "My input 2",
      "type": "input_type"
    }
  },
  "required": [],
  "order": ["myInput2", "myInput1"]
}
```