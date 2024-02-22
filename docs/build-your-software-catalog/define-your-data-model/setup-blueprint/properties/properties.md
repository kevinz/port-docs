---

sidebar_position: 1

---

import DocCardList from '@theme/DocCardList';

# 属性

每个蓝图的 "模式 "下都有一个 "属性 "部分。 在这一部分，您可以定义描述资产的所有独特属性。

## 结构

```json showLineNumbers
{
  "myProp": {
    "title": "My property",
    "icon": "My icon",
    "description": "My property",
    "type": "property_type"
  }
}
```

下表列出了构成基本属性定义的不同组成部分: 


| Field         | Description                                                                                                                                                                        |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `title`       | Property title                                                                                                                                                                     |
| `type`        | **Mandatory field.** The data type of the property.                                                                                                                                |
| `icon`        | Icon for the property <br /><br />See the [full icon list](../setup-blueprint.md#full-icon-list).                                                                                  |
| `description` | Description of the property.<br /> This value is visible to users when hovering on the info icon in the UI. It provides detailed information about the use of a specific property. |
| `default`     | Default value for this property in case an entity is created without explicitly providing a value.                                                                                 |


:::tip 属性名称是属性对象的键。 例如，在上面的代码块中，属性名称是`myProp`。

:::

## 支持的属性

<DocCardList />

## 杂项

### 可用的枚举颜色

使用[enum](./string.md?api-definition=enum#api-definition) 定义的属性还可以为属性定义中可用的不同值包含特定的颜色，可用的枚举颜色有

```showLineNumbers text
blue
turquoise
orange
purple
pink
yellow
green
red
darkGray
lightGray
bronze
```