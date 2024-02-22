通过简单引用，可以将资源库中的文件内容作为实体属性的值。

下面的示例将读取 `~/module1/README.md` 中的字符串内容，并将其上传到指定实体的 `myStringProp` 中。

被引用到示例中的资源库文件夹结构: 

```
root
|
+- port.yml
|
+-+ module1
|   |
|   +- README.md
|   |
|   +-+ src
...
```

Port.yml` 文件: 

```yaml showLineNumbers
blueprint: code_module
title: Module 1
identifier: module_1_entity
properties:
  # highlight-next-line
  myStringProp: file://module1/README.md
```