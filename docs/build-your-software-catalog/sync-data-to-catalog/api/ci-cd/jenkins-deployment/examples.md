import ExampleImageBlueprint from "../_ci_example_image_blueprint.mdx";
import ExampleCiJobBlueprint from "../_ci_example_ci_job_blueprint.mdx";

# 示例

## 基本创建/更新示例

:::note 在本例中，以下插件是一个依赖项: 

* 构建用户变量插件(>=v1.9)

:::

在本例中，您将为 `ciJob` 和 `镜像` 实体创建蓝图，并在它们之间建立关系。 然后，您将添加一些 Groovy 代码，以便在每次触发 Jenkins 管道时在 Port 中创建新实体: 

<ExampleImageBlueprint />

<ExampleCiJobBlueprint />

:::note 以下示例中使用了变量 `token` 和 `API_URL`。点击[here](./jenkins-deployment.md#fetching-your-api-token) 查看它们是如何定义的。

:::

创建蓝图后，可将以下代码段添加到 Jenkins 构建中，以创建新的构建实体: 

```js showLineNumbers
// Use the `build users` plugin to fetch the triggering user's ID
wrap([$class: 'BuildUser']) {
        def user = env.BUILD_USER_ID
    }

    def body = """
        {
            "identifier": "new-cijob-run",
            "properties": {
                "triggeredBy": "${user}",
                "runLink": "${env.BUILD_URL}",
            },
            "relations": {
                "image": ["example-image"]
            }
        }
    """

    response = httpRequest contentType: "APPLICATION_JSON", httpMode: "POST",
            url: "${API_URL}/v1/blueprints/ciJob/entities?upsert=true&merge=true&create_missing_related_entities=true",
            requestBody: body,
            customHeaders: [
                [name: "Authorization", value: "Bearer ${token}"],
            ]
```

:::note 请注意，您还创建了 `image` 关系，并添加了名为 `example-image` 的相关镜像实体。 这是 ciJob 的工件，您稍后会更新它。 创建时使用了 API url 中的 `create_missing_related_entities=true` flag，这样即使 `example-image` 实体还不存在，也能创建关系。

:::

## 基本获取示例

下面的示例从上一个示例中获取了 `new-cijob-run` 实体，如果您的 CI 流程创建了一个构建工件，然后引用了其中的某些数据(例如，最新的 `ciJob` 的运行链接)，这可能会很有用。

在 Jenkins 构建中添加以下代码: 

```js showLineNumbers
response = httpRequest contentType: "APPLICATION_JSON", httpMode: "GET",
            url: "${API_URL}/v1/blueprints/ciJob/entities/new-cijob-run",
            customHeaders: [
                [name: "Authorization", value: "Bearer ${token}"],
            ]
    println(response.content)
```

## 关系示例

在下面的示例中，您将更新在创建上一个示例中显示的 `ciJob` 实体时自动创建的 `example-image` 实体。

在 Jenkins Pipeline 中添加以下代码段: 

```js showLineNumbers
def body = """
        {
            "identifier": "example-image",
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
        """
response = httpRequest contentType: "APPLICATION_JSON", httpMode: "POST",
url: "${API_URL}/v1/blueprints/image/entities?upsert=true&merge=true",
    requestBody: body,
    customHeaders: [
        [name: "Authorization", value: "Bearer ${token}"],
    ]
```

就是这样！实体已创建或更新，并在用户界面中可见。