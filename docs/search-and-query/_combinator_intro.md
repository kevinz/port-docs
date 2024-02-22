有两个可用的组合器: `and`/`or`: 

* `and` - 将在所有规则之间应用逻辑 AND 运算，要求所有规则都满足给定资产的条件才能返回；
* 或"--将在所有规则之间应用逻辑 OR 运算，要求至少有一条规则满足给定资产的要求，才能返回该资产。

:::tip  单规则查询 如果查询中只有一条规则，则组合器没有任何作用。 但请记住，为了遵循查询结构，仍需将其包含在内。

<details>
<summary>Single rule query example</summary>

在下面的示例中，"rules "数组中只出现了一条规则，因此 "combinator "字段没有任何作用: 

```json showLineNumbers
{
  // highlight-next-line
  "combinator": "and",
  "rules": [
    // highlight-start
    {
      "property": "$blueprint",
      "operator": "=",
      "value": "myBlueprint"
    }
    // highlight-end
  ]
}
```

</details>

:::