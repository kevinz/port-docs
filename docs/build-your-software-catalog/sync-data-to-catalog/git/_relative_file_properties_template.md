也可以被用于为相对于 `port.yml` 规格文件位置的路径。

例如: `file://./` 用于引用与 `port.yml` 文件在同一目录下的文件。`file://./` 用于引用高一个目录的文件，依此类推。

以下示例使用相对于 `port.yml` 的路径读取 `README.md` 和 `module1/requirements.txt

被用于到示例中的资源库文件夹结构: 

```
root
|
+-+ meta
|   |
|   +-- port.yml
|   |
|   +-+ README.md
|
+-+ module1
|   |
|   +- requirements.txt
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
  readme: file://./README.md
  module1Requirements: file://../module1/requirements.txt
```