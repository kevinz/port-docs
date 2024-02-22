import ExampleImageBlueprint from "../_ci_example_image_blueprint.mdx";
import ExampleCiJobBlueprint from "../_ci_example_ci_job_blueprint.mdx";

# 示例

## 基本创建/更新示例

在此示例中，您将为 "ciJob "和 "镜像 "实体以及它们之间的关系创建蓝图。 然后，您将添加一些 Python 代码，以便在每次触发 Azure Pipeline 时在 Port 中创建新实体: 

<ExampleImageBlueprint />

<ExampleCiJobBlueprint />

创建蓝图后，您可以添加一个 Python 脚本，用于创建/更新 Port 实体: 

```python showLineNumber
import os
import requests
import json

# Env vars passed by the pipeline variables
# highlight-start
CLIENT_ID = os.environ['PORT_CLIENT_ID']
CLIENT_SECRET = os.environ['PORT_CLIENT_SECRET']
# highlight-end
API_URL = 'https://api.getport.io/v1'

credentials = {
    'clientId': CLIENT_ID,
    'clientSecret': CLIENT_SECRET
}
token_response = requests.post(f"{API_URL}/auth/access_token", json=credentials)
# use this access token + header for all http requests to Port
# highlight-start
access_token = token_response.json()['accessToken']
headers = {
    'Authorization': f'Bearer {access_token}'
}
# highlight-end

entity_json = {
  "identifier": "new-cijob-run",
  "properties": {
    "triggeredBy": os.environ['QUEUED_BY'],
    "commitHash": os.environ['GIT_SHA'],
    "actionJob": os.environ['JOB_NAME'],
    "jobLink": os.environ['JOB_URL']
  },
  "relations": {
      "image": ["example-image"]
  }
}

create_response = requests.post(f'{API_URL}/blueprints/{blueprint_id}/entities?upsert=true&create_missing_related_entities=true', json=entity_json, headers=headers)
```

:::note 请注意，您还创建了 `image` 关系，并添加了名为 `example-image` 的相关镜像实体。 这是 ciJob 的工件，您稍后会更新它。 创建时使用了 API url 中的 `create_missing_related_entities=true` flag，这样即使 `example-image` 实体还不存在，也能创建关系。

:::

将新的 Python 脚本添加到存储库后，在 Azure 管道 `yml` 文件中添加以下代码，以调用脚本并更新/创建新的 `ciJob` 实体: 

```yaml showLineNumbers
steps:
  # ... other tasks and steps
  - script: |
      pip install -r port_requirements.txt
  - task: PythonScript@0
    env:
      PORT_CLIENT_ID: $(PORT_CLIENT_ID)
      PORT_CLIENT_SECRET: $(PORT_CLIENT_SECRET)
      QUEUED_BY: $(Build.QueuedBy)
      GIT_SHA: $(Build.SourceVersion)
      JOB_NAME: $(Build.DefinitionName)
      JOB_URL: "$(System.TeamFoundationCollectionUri)$(System.TeamProject)/_build/results?buildId=$(Build.BuildId)"
    inputs:
      scriptSource: "filePath"
      scriptPath: "main.py"
```

## 基本获取示例

下面的示例从上一个示例中获取了 `new-cijob-run` 实体，如果您的 CI 流程创建了一个构建工件，然后引用了其中的某些数据(例如，最新的 `ciJob` 的运行链接)，这可能会很有用。

在 Python 代码中添加以下代码段: 

```python showLineNumbers
entity_id = "new-cijob-run"
blueprint_id = "ciJob"

get_response = requests.get(f"{API_URL}/blueprints/{blueprint_id}/entities/{entity_id}",
                        headers=headers)
entity = get_response.json()['entity']
print(f"Image tag is: {entity['properties']['runLink']}")
```

## 关系示例

在下面的示例中，您将更新在创建上一个示例中显示的 `ciJob` 实体时自动创建的 `example-image` 实体。

在 Python 代码中添加以下代码段: 

```python showLineNumbers
image_entity_json = {
  "identifier": "example-image",
  "team": [],
  "properties": {
    "imageTag": "v1",
    "synkHighVulnerabilities": "0",
    "synkMediumVulnerabilities": "0",
    "gitRepoUrl": "https://github.com/my-org/my-cool-repo",
    "imageRegistry": "docker.io/cool-image",
    "size": "0.71",
    "unitTestCoverage": "20",
    "unitTestCoverage": "50"
  },
  "relations": {}
}

create_image_response = requests.post(f'{API_URL}/blueprints/image/entities?upsert=true', json=image_entity_json, headers=headers)
```

就是这样！实体已创建或更新，并在用户界面中可见。