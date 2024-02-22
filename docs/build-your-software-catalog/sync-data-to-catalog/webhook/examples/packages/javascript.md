---
sidebar_position: 1
description: 将 Javascript 程序包收录到您的目录中

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import ServiceBlueprint from './resources/service/_example_global_service_blueprint.mdx'
import PackageBlueprint from './resources/javascript/_example_package_blueprint.mdx'
import PackageWebhookConfig from './resources/javascript/_example_package_webhook_config.mdx'

# JavaScript

在本示例中，您将创建一个 `package` 蓝图，该蓝图将使用 Port 的[API](../../../api/api.md) 和[webhook functionality](../../webhook.md) 组合，在 `package.json` 文件中引用所有第三方依赖和库。然后，您将把该蓝图与 `service` 蓝图关联起来，这样就可以映射服务所引用的所有软件包。

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

下面是一个示例片段，展示了如何使用 Python 和 Bash 将 Port 的 API 和 webhook 与现有管道集成: 

<Tabs groupId="usage" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "Bash", value: "bash"}
]}>

<TabItem value="python">

在版本库中创建以下 Python 脚本，以创建或更新 Port 实体，作为管道的一部分: 

<details>
  <summary> Python script example </summary>

```python showLineNumbers
import requests
import json

# Get environment variables using the config object or os.environ["KEY"]
WEBHOOK_URL = os.environ['WEBHOOK_URL'] ## the value of the URL you receive after creating the Port webhook
SERVICE_ID = os.environ['SERVICE_ID'] ## The identifier of your service in Port
PATH_TO_PACKAGE_JSON_FILE = os.environ['PATH_TO_PACKAGE_JSON_FILE']

def add_entity_to_port(entity_object):
    """A function to create the passed entity in Port using the webhook URL

    Params
    --------------
    entity_object: dict
        The entity to add in your Port catalog

    Returns
    --------------
    response: dict
        The response object after calling the webhook
    """
    headers = {"Accept": "application/json"}
    response = requests.post(WEBHOOK_URL, json=entity_object, headers=headers)
    return response.json()

def convert_package_json(package_json_path):
    """This function takes a package.json file path, converts the "dependencies" property into a
    JSON array using three keys (name, version, and id). It then sends the data to Port

    Params
    --------------
    package_json_path: str
        The path to the package.json file relative to the project's root folder

    Returns
    --------------
    response: dict
        The response object after calling the webhook
    """
    with open(package_json_path) as file:
        data = json.load(file)

    dependencies = data.get('dependencies', {})

    converted_dependencies = []
    for index, (name, version) in enumerate(dependencies.items(), start=1):
        pkg_id = f"pkg-{index}"
        converted_dependencies.append({
            'name': name,
            'version': version,
            'id': pkg_id
        })

    entity_object = {
        "service": SERVICE_ID,
        "dependencies": converted_dependencies
    }
    webhook_response = add_entity_to_port(entity_object)
    return webhook_response

converted_data = convert_package_json(PATH_TO_PACKAGE_JSON_FILE)
print(converted_data)
```

</details>

</TabItem>

<TabItem value="bash">

在版本库中创建以下 Bash 脚本，以创建或更新作为 Pipelines 一部分的 Port 实体: 

<details>
  <summary> Bash script example </summary>

```bash showLineNumbers
#!/bin/sh

# Get environment variables
WEBHOOK_URL="$WEBHOOK_URL"
SERVICE_ID="$SERVICE_ID"
PATH_TO_PACKAGE_JSON_FILE="$PATH_TO_PACKAGE_JSON_FILE"

add_entity_to_port() {
    local entity_object="$1"
    local headers="Accept: application/json"
    local response=$(curl -X POST -H "$headers" -H "Content-Type: application/json" -d "$entity_object" "$WEBHOOK_URL")
    echo "$response"
}

# This function takes a package.json file path, converts the "dependencies" property into a
# JSON array using three keys (name, version, and id). It then sends this data to Port
convert_package_json() {
    local package_json_path="$1"
    local data=$(cat "$package_json_path")
    local dependencies=$(echo "$data" | jq -r '.dependencies // {}')

    local converted_dependencies=""
    local index=1
    while IFS="=" read -r dep_name version; do
        pkg_id="pkg-$index"
        converted_dependencies="$converted_dependencies{\"name\":\"$dep_name\",\"version\":\"$version\",\"id\":\"$pkg_id\"},"
        index=$((index + 1))
    done <<EOF
$(echo "$dependencies" | jq -r 'to_entries[] | .key + "=" + .value')
EOF

    local entity_object="{\"service\":\"$SERVICE_ID\",\"dependencies\":[${converted_dependencies%,}]}"
    local webhook_response=$(add_entity_to_port "$entity_object")
    echo "$webhook_response"
}

converted_data=$(convert_package_json "$PATH_TO_PACKAGE_JSON_FILE")
echo "$converted_data"
```

</details>

</TabItem>
</Tabs>

有关如何将上述脚本与现有 Gitlab CI Pipelines 集成的示例，请访问: 

* * [Package.json example](https://github.com/port-labs/package-json-webhook-example)
