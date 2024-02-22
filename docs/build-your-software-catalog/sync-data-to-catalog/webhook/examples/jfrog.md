---
sidebar_position: 18
description: 将 JFrog 工件、Docker 标记和构建摄入到您的目录中

---

# JFrog

在本示例中，您将在 JFrog 服务器和 Port 之间创建一个 webhook 集成，该集成将有助于将 JFrog 工件、Docker 标记和构建实体导入 Port。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Jfrog artifact blueprint</summary>

```json showLineNumbers
{
  "identifier": "jfrogArtifact",
  "description": "This blueprint represents an artifact in our JFrog catalog",
  "title": "JFrog Artifact",
  "icon": "JfrogXray",
  "schema": {
    "properties": {
      "name": {
        "type": "string",
        "title": "Name",
        "description": "Name of the artifact"
      },
      "path": {
        "type": "string",
        "title": "Path",
        "description": "Path to artifact"
      },
      "sha256": {
        "type": "string",
        "title": "SHA 256",
        "description": "SHA256 of the artifact"
      },
      "size": {
        "type": "number",
        "title": "Size",
        "description": "Size of the artifact"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {
    "repository": {
      "title": "Repository",
      "description": "Repository of the artifact",
      "target": "jfrogRepository",
      "required": false,
      "many": false
    }
  }
}
```

</details>

<details>
<summary>Jfrog Docker tag blueprint</summary>

```json showLineNumbers
{
  "identifier": "jfrogDockerTag",
  "description": "This blueprint represents a Docker tag in our Jfrog catalog",
  "title": "JFrog Docker Tag",
  "icon": "JfrogXray",
  "schema": {
    "properties": {
      "name": {
        "type": "string",
        "title": "Name",
        "description": "Name of the Docker tag"
      },
      "imageName": {
        "type": "string",
        "title": "Image Name",
        "description": "Name of the Docker image"
      },
      "path": {
        "type": "string",
        "title": "Path",
        "description": "Path to Docker tag"
      },
      "sha256": {
        "type": "string",
        "title": "SHA 256",
        "description": "SHA256 of the Docker tag"
      },
      "size": {
        "type": "number",
        "title": "Size",
        "description": "Size of the Docker tag"
      },
      "tag": {
        "type": "string",
        "title": "Docker tag",
        "description": "Docker tag"
      },
      "platforms": {
        "type": "array",
        "title": "Platforms",
        "description": "Platforms supported by image"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {
    "repository": {
      "title": "Repository",
      "description": "Repository of the artifact",
      "target": "jfrogRepository",
      "required": false,
      "many": false
    }
  }
}
```

</details>

<details>
<summary>Jfrog repository blueprint</summary>

```json showLineNumbers
{
  "identifier": "jfrogRepository",
  "description": "This blueprint represents a repository on Jfrog",
  "title": "JFrog Repository",
  "icon": "JfrogXray",
  "schema": {
    "properties": {
      "key": {
        "type": "string",
        "title": "Key",
        "description": "Name of the repository"
      },
      "description": {
        "type": "string",
        "title": "Description",
        "description": "Description of the repository"
      },
      "type": {
        "type": "string",
        "title": "Repository Type",
        "description": "Type of the repository",
        "enum": ["LOCAL", "REMOTE", "VIRTUAL", "FEDERATED", "DISTRIBUTION"],
        "enumColors": {
          "LOCAL": "blue",
          "REMOTE": "bronze",
          "VIRTUAL": "darkGray",
          "FEDERATED": "green",
          "DISTRIBUTION": "lightGray"
        }
      },
      "url": {
        "type": "string",
        "title": "Repository URL",
        "description": "URL to the repository",
        "format": "url"
      },
      "packageType": {
        "type": "string",
        "title": "Package type",
        "description": "Type of the package"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {}
}
```

</details>

<details>
<summary>Jfrog build blueprint</summary>

```json showLineNumbers
{
  "identifier": "jfrogBuild",
  "description": "This blueprint represents a build from JFrog",
  "title": "JFrog Build",
  "icon": "JfrogXray",
  "schema": {
    "properties": {
      "name": {
        "type": "string",
        "title": "Build name",
        "description": "Name of the build"
      },
      "uri": {
        "type": "string",
        "title": "Build URI",
        "description": "URI to the build"
      },
      "lastStarted": {
        "type": "string",
        "title": "Last build time",
        "description": "Last time the build ran",
        "format": "date-time"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "aggregationProperties": {},
  "relations": {}
}
```

</details>
:::tip  Blueprint Properties
You may modify the properties in your blueprints depending on what you want to track in your JFrog repositories and builds.

:::

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>JFrog webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `JFrog mapper`；
    2.标识符 : `jfrogMapper`；
    3.Description : `将 JFrog 仓库和构建映射到 Port` 的 webhook 配置；
    4.图标 : `JfrogXray`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 

```json
[
  {
    "blueprint": "jfrogBuild",
    "filter": ".body.event_type == 'uploaded'",
    "entity": {
      "identifier": ".body.build_name",
      "title": ".body.build_name",
      "properties": {
        "name": ".body.build_name",
        "uri": "'/' + .body.build_name",
        "lastStarted": ".body.build_started"
      }
    }
  },
  {
    "blueprint": "jfrogDockerTag",
    "filter": ".body.event_type == 'pushed'",
    "entity": {
      "identifier": ".body.name",
      "title": ".body.name",
      "properties": {
        "name": ".body.name",
        "imageName": ".body.image_name",
        "path": ".body.path",
        "sha256": ".body.sha256",
        "size": ".body.size",
        "tag": ".body.tag",
        "platforms": ".body.platforms[] | \"(.os):(.architecture)\""
      },
      "relations": {
        "repository": ".body.repo_key"
      }
    }
  },
  {
    "blueprint": "jfrogArtifact",
    "filter": ".body.event_type == 'deployed'",
    "entity": {
      "identifier": ".body.data.name",
      "title": ".body.data.name",
      "properties": {
        "name": ".body.data.name",
        "path": ".body.data.path",
        "sha256": ".body.data.sha256",
        "size": ".body.data.size"
      },
      "relations": {
        "repository": ".body.data.repo_key"
      }
    }
  }
]
```

:::note 记下并复制该选项卡中提供的 Webhook URL

:::

3.点击页面底部的**保存**。

</details>

## 在 JFrog 中创建 webhook

1. 以具有管理员权限的用户身份登录 JFrog
2. 点击用户图标左侧右上角的齿轮图标；
3. 选择**平台管理**；
4. 在左侧边栏底部，即**常规**下方，选择**Webhooks**；
5. 点击**创建 Webhook**
6. 输入以下详细信息: 
    1. 名称"- 被用于一个有意义的名称，如**Port-Artifact**；
    2. `Description` - 输入 Webhook 的描述；
    3. `URL` - 输入在 Port 中创建 webhook 配置后收到的 `url` 键的值；
    4. `Events` - 在**Artifacts**下，选择**Artifact was deployed**，然后选择所有相应的存储库；
7.单击页面底部的**创建**。
8.使用详细信息再创建两个 webhook: 
    1.用于构建: 
        + 名称:  **Port-Build**；
        + 事件: 在**构建**下，选择**构建已上传**，然后选择所有适用的构建；
        + `Description` - 输入 webhook 的描述；
        + `URL` - 输入您在 Port 中创建 webhook 配置后收到的 `url` 键的值；
    2.对于 Docker 标记: 
        + 名称:  **Port-Docker-Tag**；
        + 事件: 在 **Docker**下，选择 **Docker标签已推送**，然后选择所有适用的软件源

:::tip 为了查看 JFrog webhooks 中可用的不同有效载荷和事件、[look here](https://jfrog.com/help/r/jfrog-platform-administration-documentation/event-types)

:::

完成！您发布、触发或上传的任何工件都将触发 Webhook 事件，JFrog 将把该事件发送到 Provider 提供的 Webhook URL。 Port 将根据映射解析事件并相应地更新目录实体。

## 让我们来测试一下

本节包括 JFrog 在上传构建时发送的 webhook 事件示例。 此外，本节还包括根据上一节提供的 webhook 配置从事件中创建的实体。

### 有效载荷

下面是上传 JFrob 构建时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary>Webhook event payload</summary>

```json showLineNumbers
{
  "build_name": "sample_build_name",
  "event_type": "uploaded",
  "build_number": "1",
  "build_started": "2020-06-18T14:40:49.869+0300"
}
```

</details>

#### 映射结果

```json showLineNumbers
{
  "identifier": "sample_build_name",
  "title": "sample_build_name",
  "blueprint": "jfrogBuild",
  "properties": {
    "name": "sample_build_name",
    "uri": "/sample_build_name",
    "lastStarted": "21 hours ago"
  },
  "relations": {},
  "filter": true
}
```

## 导入 JFrog 历史版本和存储库

在本示例中，您将使用 Provider 提供的 Python 脚本从 JFrog Server API 获取数据并将其引用到 Port。

### 先决条件

本示例使用的是上一节中的[blueprint and webhook](#port-configuration) 定义。

此外，您还需要以下环境变量: 

* `PORT_CLIENT_ID` - 您的 Port 客户端 ID
* `PORT_CLIENT_SECRET` - 您的 Port 客户端secret
* `JFROG_ACCESS_TOKEN` - 您可以通过下面的说明获取[Jfrog documentation](https://jfrog.com/help/r/jfrog-platform-administration-documentation/access-tokens)
* `JFROG_HOST_URL` - 您的 Jfrog 实例的主机 URL

:::info 使用以下命令查找您的 Port 凭据[guide](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)

:::

使用以下 Python 脚本将历史 JFrog 版本和软件源引用到 Port 中: 

<details>
<summary>JFrog Python script for historical builds and repositories</summary>

```python showLineNumbers
# Dependencies to install
# pip install python-dotenv
# pip install requests

import logging
import os

import dotenv
import requests

dotenv.load_dotenv()

logger = logging.getLogger(__name__)

PORT_API_URL = "https://api.getport.io/v1"
PORT_CLIENT_ID = os.getenv("PORT_CLIENT_ID")
PORT_CLIENT_SECRET = os.getenv("PORT_CLIENT_SECRET")
JFROG_ACCESS_TOKEN = os.getenv("JFROG_ACCESS_TOKEN")
JFROG_HOST_URL = os.getenv("JFROG_HOST_URL")

class Blueprint:
    REPOSITORY = "jfrogRepository"
    BUILD = "jfrogBuild"

## Get Port Access Token
credentials = {"clientId": PORT_CLIENT_ID, "clientSecret": PORT_CLIENT_SECRET}
token_response = requests.post(f"{PORT_API_URL}/auth/access_token", json=credentials)
access_token = token_response.json()["accessToken"]

# You can now use the value in access_token when making further requests
headers = {"Authorization": f"Bearer {access_token}"}

def add_entity_to_port(blueprint_id, entity_object, transform_function):
    """A function to create the passed entity in Port

    Params
    --------------
    blueprint_id: str
        The blueprint id to create the entity in Port

    entity_object: dict
        The entity to add in your Port catalog

    transform_function: function
        A function to transform the entity object to the Port entity object

    Returns
    --------------
    response: dict
        The response object after calling the webhook
    """
    logger.info(f"Adding entity to Port: {entity_object}")
    entity_payload = transform_function(entity_object)
    response = requests.post(
        (
            f"{PORT_API_URL}/blueprints/"
            f"{blueprint_id}/entities?upsert=true&merge=true"
        ),
        json=entity_payload,
        headers=headers,
    )
    logger.info(response.json())

def get_all_builds():
    logger.info("Getting all builds")
    url = f"{JFROG_HOST_URL}/artifactory/api/build"
    response = requests.get(
        url, headers={"Authorization": "Bearer " + JFROG_ACCESS_TOKEN}
    )
    response.raise_for_status()
    builds = response.json()["builds"]
    return builds

def get_all_repositories():
    logger.info("Getting all repositories")
    url = f"{JFROG_HOST_URL}/artifactory/api/repositories"
    response = requests.get(
        url, headers={"Authorization": "Bearer " + JFROG_ACCESS_TOKEN}
    )
    response.raise_for_status()
    repositories = response.json()
    return repositories

if __name__ == "__main__":
    logger.info("Starting Port integration")
    for repository in get_all_repositories():
        repository_object = {
            "key": repository["key"],
            "description": repository.get("description", ""),
            "type": repository["type"].upper(),
            "url": repository["url"],
            "packageType": repository["packageType"].upper(),
        }
        transform_build_function = lambda x: {
            "identifier": repository_object["key"],
            "title": repository_object["key"],
            "properties": {
                **repository_object,
            },
        }
        logger.info(f"Added repository: {repository_object['key']}")
        add_entity_to_port(
            Blueprint.REPOSITORY, repository_object, transform_build_function
        )

    logger.info("Completed repositories, starting builds")
    for build in get_all_builds():
        build_object = {
            "name": build["uri"].split("/")[-1],
            "uri": build["uri"],
            "lastStarted": build["lastStarted"],
        }
        transform_build_function = lambda x: {
            "identifier": build_object["name"],
            "title": build_object["name"],
            "properties": {
                **build_object,
            },
        }
        logger.info(f"Added build: {build_object['name']}")
        add_entity_to_port(Blueprint.BUILD, build_object, transform_build_function)
```

</details>

完成！现在您可以将历史构建和软件源从 JFrog 导入 Port。 Port 将根据映射解析构建和软件源，并相应地更新目录实体。