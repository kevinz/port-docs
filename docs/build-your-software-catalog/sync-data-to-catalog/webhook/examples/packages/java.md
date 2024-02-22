---
sidebar_position: 4
description: 将 Maven 依赖项纳入目录

---

import MavenBlueprint from "./resources/java/_example_maven_blueprint.mdx";
import MavenConfiguration from "./resources/java/_example_maven_webhook_configuration.mdx"
import MavenShell from "./resources/java/_example_maven_bash.mdx"

# Java

在本示例中，您将创建一个 `mavenDependencies` 蓝图，该蓝图将通过 Port's[API](../../../api/api.md) 和[webhook functionality](../../webhook.md) 的组合来引用 Maven 软件包。

要将 Maven 依赖包引用到 Port，需要使用一个脚本，该脚本会根据 webhook 配置发送有关软件包的信息。

## 先决条件

创建以下蓝图定义和 webhook 配置: 

<details>
<summary>Maven dependency blueprint</summary>
<MavenBlueprint/>
</details>

<details>
<summary>Maven webhook configuration</summary>
<MavenConfiguration/>
</details>

<details>
<summary>Maven Bash script</summary>
<MavenShell/>
</details>

## 解析 `pom.xml` 文件并向 Port 发送依赖关系数据

下文将概述如何使用映射器脚本将数据从 `pom.xml` 文件发送到 Port。

#### 脚本 Usage

1. 将脚本复制到 Java 项目根目录下的文件中。确保您的 `pom.xml` 文件也位于项目根目录下；
2. 使脚本可执行。例如，如果将脚本命名为 `parse_and_send.sh`，则应使用以下命令: 


   ```bash showLineNumbers
   chmod +x parse_and_send.sh
   ```


3.运行脚本: 


   ```bash showLineNumbers
   ./parse_and_send.sh
   ```


完成！脚本运行后，将自动通过 HTTP 请求将 Maven 依赖项注入 Port