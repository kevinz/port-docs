---
sidebar_position: 5
description: 将 Swagger 路径纳入目录

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import SwaggerBlueprint from './resources/swagger/_example_swagger_blueprint.mdx'
import SwaggerWebhookConfig from './resources/swagger/_example_swagger_webhook_config.mdx'

# Swagger

在本示例中，您将创建一个 `swaggerPath` 蓝图，该蓝图将使用 Port's[API](../../../api/api.md) 和[webhook functionality](../../webhook.md) 的组合来引用 `swagger.json` 文件中的所有 API 路径。

要将 API 路径引用到 Port，需要使用一个脚本，根据 webhook 配置发送路径信息。

## 先决条件

创建以下蓝图定义和 webhook 配置: 

<details>
<summary>Swagger path blueprint</summary>
<SwaggerBlueprint/>
</details>

<details>
<summary>Swagger path webhook configuration</summary>

<SwaggerWebhookConfig/>

</details>

## 使用 Port 的 API 和 Bash 脚本

下面的示例片段展示了如何使用 Python 和 Bash 将 Port 的 API 和 Webhook 与现有管道集成: 

<Tabs groupId="usage" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "Bash", value: "bash"}
]}>

<TabItem value="python">

在版本库中创建以下 Python 脚本，以创建或更新 Port 实体，作为管道的一部分: 

<details>
  <summary> Python script example </summary>

```python showLineNumbers
## Import the needed libraries

import requests
import json
import os

# Get environment variables using the config object or os.environ["KEY"]
WEBHOOK_URL = os.environ['WEBHOOK_URL'] ## the value of the URL you receive after creating the Port webhook
PATH_TO_SWAGGER_JSON_FILE = os.environ["PATH_TO_SWAGGER_JSON_FILE"]

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

def read_swagger_file(swagger_json_path):
    """This function takes a swagger.json file path, converts the "paths" property into a
    JSON array and then sends this data to Port

    Params
    --------------
    swagger_json_path: str
        The path to the swagger.json file relative to the project's root folder

    Returns
    --------------
    response: dict
        The response object after calling the webhook
    """
    with open(swagger_json_path) as file:
        data = json.load(file)

    project_info = data.get("info")
    project_title = project_info.get("title")
    project_version = project_info.get("version")
    hosted_url = data.get("host")
    base_path = data.get("basePath")

    paths = data.get('paths', {})
    path_list = []
    index = 1
    for path, methods in paths.items():
        for method, method_info in methods.items():
            path_id = f"{project_title}-{index}"
            path_info = {
                "id": path_id,
                "path": path,
                "method": method,
                "summary": method_info.get('summary'),
                "description": method_info.get('description'),
                "parameters": method_info.get("parameters"),
                "responses": method_info.get("responses"),
                "project": project_title,
                "version": project_version,
                "host": "https://" + hosted_url + base_path
            }
            path_list.append(path_info)
            index+=1

    entity_object = {
        "paths": path_list
    }
    webhook_response = add_entity_to_port(entity_object)
    return webhook_response

response = read_swagger_file(PATH_TO_SWAGGER_JSON_FILE)
print(response)
```

</details>

</TabItem>

<TabItem value="bash">

在版本库中创建以下 Bash 脚本，以创建或更新作为 Pipelines 一部分的 Port 实体: 

<details>
  <summary> Bash script example </summary>

```bash showLineNumbers
#!/bin/bash

# Set the environment variables
WEBHOOK_URL="$WEBHOOK_URL"
PATH_TO_SWAGGER_JSON_FILE="$PATH_TO_SWAGGER_JSON_FILE"

add_entity_to_port() {
    local entity_object="$1"
    local headers="Accept: application/json"
    local response=$(curl -X POST -H "$headers" -H "Content-Type: application/json" -d "$entity_object" "$WEBHOOK_URL")
    echo "$response"
}

read_swagger_json() {
    local swagger_json_path="$1"
    local data=$(cat "$swagger_json_path")

    local project_info=$(echo "$data" | jq -r '.info')
    local project_title=$(echo "$project_info" | jq -r '.title')
    local project_version=$(echo "$project_info" | jq -r '.version')
    local hosted_url=$(echo "$data" | jq -r '.host')
    local base_path=$(echo "$data" | jq -r '.basePath')
    local paths=$(echo "$data" | jq -r '.paths')

    local path_list=""
    local index=1
    while IFS="=" read -r path methods; do
        while IFS="=" read -r method method_info; do
            local path_id="${project_title}-${index}"
            local summary=$(echo "$method_info" | jq -r '.summary')
            local description=$(echo "$method_info" | jq -r '.description')
            local parameters=$(echo "$method_info" | jq -r '.parameters')
            local responses=$(echo "$method_info" | jq -r '.responses')

            local path_info="{\"id\":\"$path_id\",\"path\":\"$path\",\"method\":\"$method\",\"summary\":\"$summary\",\"description\":\"$description\",\"parameters\":$parameters,\"responses\":$responses,\"project\":\"$project_title\",\"version\":\"$project_version\",\"host\":\"https://${hosted_url}${base_path}\"}"
            path_list="${path_list}${path_info},"
            index=$((index + 1))
        done <<EOF
$(echo "$methods" | jq -r 'to_entries[] | .key + "=" + (.value | @json)')
EOF
    done <<EOF
$(echo "$paths" | jq -r 'to_entries[] | .key + "=" + (.value | @json)')
EOF

    local entity_object="{\"paths\":[${path_list%,}]}"
    local webhook_response=$(add_entity_to_port "$entity_object")
    echo "$webhook_response"
}

response=$(read_swagger_json "$PATH_TO_SWAGGER_JSON_FILE")
echo "$response"
```

</details>

</TabItem>
</Tabs>