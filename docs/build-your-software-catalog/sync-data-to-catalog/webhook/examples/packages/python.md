---

sidebar_position: 2
description: 将 Python 软件包收录到您的目录中

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import ServiceBlueprint from './resources/service/_example_global_service_blueprint.mdx'
import PackageBlueprint from './resources/python/_example_package_blueprint.mdx'
import PackageWebhookConfig from './resources/python/_example_package_webhook_config.mdx'

# Python

在本示例中，您将创建一个 `package` 蓝图，该蓝图将使用 Port 的[API](../../../api/api.md) 和[webhook functionality](../../webhook.md) 组合来引用 `requirements.txt` 文件中的所有第三方依赖项和库。然后，您将把该蓝图与 `service` 蓝图关联起来，这样就可以映射服务所引用的所有软件包。

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
PATH_TO_REQUIREMENTS_TXT_FILE = os.environ['PATH_TO_REQUIREMENTS_TXT_FILE']

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
    headers = {"Content-Type": "application/json"}
    response = requests.post(WEBHOOK_URL, json=entity_object, headers=headers)
    return response.json()

def convert_requirements_txt(requirements_txt_path):
    """This function takes a requirements.txt file path, converts all the dependencies into a
    JSON array using three keys (name, version, and id). It then sends this data to Port

    Params
    --------------
    requirements_txt_path: str
        The path to the requirements.txt file relative to the project's root folder

    Returns
    --------------
    response: dict
        The response object after calling the webhook"""
    with open(requirements_txt_path, 'r') as file:
        requirements = file.readlines()

    dependencies = []
    for index, requirement in enumerate(requirements, start=1):
        requirement = requirement.strip()
        if requirement:
            name, version = requirement.split("==")
            pkg_id = f"pkg-{index}"
            dependencies.append({
                'name': name,
                'version': version,
                'id': pkg_id
            })

    converted_data = {
        "service": SERVICE_ID,
        'dependencies': dependencies
    }

    return converted_data

entity_object = convert_requirements_txt(PATH_TO_REQUIREMENTS_TXT_FILE)
webhook_response = add_entity_to_port(entity_object)
print(webhook_response)
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
PATH_TO_REQUIREMENTS_TXT_FILE="$PATH_TO_REQUIREMENTS_TXT_FILE"

add_entity_to_port() {
    local entity_object="$1"
    local headers="Accept: application/json"
    local response=$(curl -X POST -H "$headers" -H "Content-Type: application/json" -d "$entity_object" "$WEBHOOK_URL")
    echo "$response"
}

# This function takes a requirements.txt file path, converts all the dependencies into a
# JSON array using three keys (name, version, and id). It then sends this data to Port

#!/bin/sh

convert_requirements_txt() {
    requirements_txt_path="$1"

    # Initialize variables
    index=1
    dependencies=""

    # Read the requirements.txt file line by line
    while IFS= read -r line || [ -n "$line" ]; do
        # Trim leading and trailing whitespace
        line=$(echo "$line" | sed -e 's/^[[:space:]]*//' -e 's/[[:space:]]*$//')

        # Skip empty lines or lines starting with #
        if [ -z "$line" ] || [ "$(printf %.1s "$line")" = "#" ]; then
            continue
        fi

        # Extract the name and version using awk
        name=$(echo "$line" | awk -F'==' '{print $1}')
        version=$(echo "$line" | awk -F'==' '{print $2}')

        # Generate the ID with the format "pkg-<ID>"
        pkg_id="pkg-$index"

        # Add the dependency to the JSON array
        dependencies="$dependencies{\"name\":\"$name\",\"version\":\"$version\",\"id\":\"$pkg_id\"},"

        # Increment the index
        index=$((index + 1))
    done < "$requirements_txt_path"

    # Remove the trailing comma from the dependencies string
    dependencies=$(echo "$dependencies" | sed 's/,$//')

    # Generate the final JSON object and send it to Port
    local entity_object="{\"service\":\"$SERVICE_ID\",\"dependencies\":[${dependencies}]}"

    local webhook_response=$(add_entity_to_port "$entity_object")
    echo "$webhook_response"

}
# Example usage

converted_data=$(convert_requirements_txt "$PATH_TO_REQUIREMENTS_TXT_FILE")
echo "$converted_data"
```

</details>

</TabItem>
</Tabs>

有关如何将上述脚本与现有 Gitlab CI Pipelines 集成的示例，请访问: 

* * [Requirements.txt example](https://github.com/port-labs/requirements-file-webhook-example)
