---
sidebar_position: 3
description: 将 Golang 软件包收录到你的目录中

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PackageBlueprint from './resources/golang/_example_package_blueprint.mdx'
import PackageWebhookConfig from './resources/golang/_example_package_webhook_config.mdx'
import ServiceBlueprint from './resources/service/_example_global_service_blueprint.mdx'

# Golang

在本例中，您将创建一个 "包 "蓝图，使用 Port's[API](../../../api/api.md) 和[webhook functionality](../../webhook.md) 的组合来引用 Go 模块、版本和依赖关系。然后，您将把该蓝图与 "服务 "蓝图关联起来，以便映射服务所引用的所有包。

为了将软件包引用到 Port，需要使用一个脚本，根据 webhook 配置发送软件包信息。

## 先决条件

创建以下蓝图定义和 webhook 配置: 

<details>
<summary>Service blueprint</summary>
<ServiceBlueprint/>
</details>

<details>
<summary>Package blueprint</summary>
<PackageBlueprint/>
</details>

<details>
<summary>Package webhook configuration</summary>

<PackageWebhookConfig/>

</details>

## 使用 Port 的 API 和 Bash 脚本

下面的示例片段展示了如何使用 Python 和 Bash 将 Port 的 API 和 Webhook 与现有管道集成: 

<Tabs groupId="usage" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "Bash", value: "bash"}
]}>

<TabItem value="bash">

在版本库中创建以下 Bash 脚本，以创建或更新作为 Pipelines 一部分的 Port 实体: 

<details>

<summary>Go Bash script</summary>

```bash showLineNumbers
#!/bin/bash

# Get environment variables
WEBHOOK_URL="$WEBHOOK_URL"
SERVICE_ID="$SERVICE_ID"

set -e
# Create or clear the output file
echo "[]" > output.json

# Extract require lines from go.mod excluding the first and the last lines
mapfile -t requires < <(sed -n '/require (/,/)/p' go.mod | tail -n +2 | head -n -1)

# Parse each require line into a package JSON
for require in "${requires[@]}"; do
    # Ignore if line is 'require (' or ')'
    if [[ "$require" == "require (" ]] || [[ "$require" == ")" ]]; then
        continue
    fi

    # Split line into an array
    IFS=' ' read -r -a parts <<< "$require"

    # Assign array items to variables
    package_url="${parts[0]}"
    version="${parts[1]}"
    indirect=false

    # Check if line is indirect
    if [[ "${parts[2]}" == "//" && "${parts[3]}" == "indirect" ]]; then
        indirect=true
    fi

    # Extract the package name from the URL
    package_name=$(basename "$package_url")

    # Prepend 'https://' to package URL if not already there and remove any white spaces
    package_url=$(echo "$package_url" | tr -d '[:space:]')
    if [[ "$package_url" != http* ]]; then
        package_url="https://$package_url"
    fi

    # Create the package JSON
    package_json=$(jq -n \
        --arg pn "$package_name" \
        --arg id "$package_name" \
        --arg pu "$package_url" \
        --arg v "$version" \
        --argjson i "$indirect" \
        '{
            identifier: $id,
            packageName: $pn,
            packageUrl: $pu,
            version: $v,
            indirect: $i,
            service: $SERVICE_ID
        }')

    # Add the package JSON to the output file
    jq --argjson p "$package_json" '. += [$p]' output.json > temp.json && mv temp.json output.json

    # Send the package JSON to the webhook
    curl --location '$WEBHOOK_URL' \
        --header 'Content-Type: application/json' \
        --data "$package_json"
done
```

:::note 

* 该脚本利用 Bash shell 中的内置命令 `mapfile` 来读取 `go.mod` 文件中的行并将其存储到数组中。请注意，并非所有 shell 默认都支持该命令。如果你被用于了不同的 shell，如 Dash 或 Zsh，你可能需要切换到 Bash 或修改脚本来实现类似的功能。
* 脚本依赖于 `jq` 命令来操作 JSON 数据。它被用来根据从 `go.mod` 文件中提取的 package 详细信息创建 JSON 对象，并将这些对象追加到输出 JSON 文件中。值得注意的是，"jq "是一个功能强大的命令行 JSON 处理器，但许多系统默认情况下并不包含它。您可能需要单独安装才能使用它。

:::

</details>
</TabItem>

<TabItem value="python">

在版本库中创建以下 Python 脚本，以创建或更新 Port 实体，作为管道的一部分: 

<details>

<summary>Go Python script</summary>

```python showLineNumbers
# Dependencies to install:
# pip install requests
# pip install tldextract

import json
import requests
import os
from urllib.parse import urlparse

output_filename = "output.json"
webhook_url = os.environ.get('WEBHOOK_URL')
SERVICE_ID = os.environ.get('SERVICE_ID')

# Prepare the headers for the requests
headers = {'Content-Type': 'application/json'}

# Initialize the output file
with open(output_filename, 'w') as f:
    json.dump([], f)

# Read the go.mod file
with open('go.mod', 'r') as f:
    lines = f.readlines()

# Find all require blocks
require_blocks = []
start = -1
for i, line in enumerate(lines):
    if line.strip() == 'require (':
        start = i
    elif line.strip() == ')' and start != -1:
        require_blocks.append(lines[start + 1:i])
        start = -1

# Process each require block
for requires in require_blocks:
    for require in requires:
        parts = require.split()  # Split the line into parts

        package_url = parts[0]
        version = parts[1]
        indirect = len(parts) > 3 and parts[2] == "//" and parts[3] == "indirect"  # Check if the package is indirect

        # Parse the package name from the URL
        package_name = os.path.basename(urlparse(package_url).path)

        # Ensure package_url is in URL format
        if not package_url.startswith('http://') and not package_url.startswith('https://'):
            package_url = 'https://' + package_url

        # Prepare the package dictionary
        package_dict = {
            "identifier": package_name,
            "package_name": package_name,
            "package_url": package_url,
            "version": version,
            "indirect": indirect,
            "service" SERVICE_ID
        }

        # Append to the output file
        with open(output_filename, 'r+') as f:
            data = json.load(f)
            data.append(package_dict)
            f.seek(0)
            json.dump(data, f, indent=4)

        # Send data to the webhook
        response = requests.post(webhook_url, headers=headers, data=json.dumps(package_dict))
        print(response.status_code)
```

</details>
</TabItem>
</Tabs>