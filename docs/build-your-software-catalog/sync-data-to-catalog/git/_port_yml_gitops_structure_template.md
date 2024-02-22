import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

下面是一个 `port.yml` 文件示例: 

```yaml showLineNumbers
identifier: myEntity
title: My Entity
blueprint: myBlueprint
properties:
  myStringProp: myValue
  myNumberProp: 5
  myUrlProp: https://example.com
relations:
  mySingleRelation: myTargetEntity
  myManyRelation:
    - myTargetEntity1
    - myTargetEntity2
```

* 标识符 "键被用来指定应用程序将创建并在发生变化时保持更新的实体的标识符: 

```yaml showLineNumbers
// highlight-next-line
identifier: myEntity
title: My Entity
  ...
```

* title "键被用来引用实体的 title: 

```yaml showLineNumbers
identifier: myEntity
// highlight-next-line
title: My Entity
  ...
```

* blueprint "键被引用来指定要创建此实体的蓝图的标识符: 

```yaml showLineNumbers
...
title: My Entity
// highlight-next-line
blueprint: myBlueprint
  ...
```

* `properties` 键被用来将值映射到实体的不同属性: 

```yaml showLineNumbers
...
title: My Entity
blueprint: myBlueprint
// highlight-start
properties:
  myStringProp: myValue
  myNumberProp: 5
  myUrlProp: https://example.com
// highlight-end
  ...
```

* 关系 "键被用来将目标实体映射到实体的不同关系上: 

<Tabs>

<TabItem value="single" label="Single relation">

```yaml showLineNumbers
...
properties:
  myStringProp: myValue
  myNumberProp: 5
  myUrlProp: https://example.com
// highlight-start
relations:
  mySingleRelation: myTargetEntity
// highlight-end
```

</TabItem>

<TabItem value="many" label="Many relation">

```yaml showLineNumbers
...
properties:
  myStringProp: myValue
  myNumberProp: 5
  myUrlProp: https://example.com
// highlight-start
relations:
  myManyRelation:
    - myTargetEntity1
    - myTargetEntity2
// highlight-end
```

</TabItem>

</Tabs>