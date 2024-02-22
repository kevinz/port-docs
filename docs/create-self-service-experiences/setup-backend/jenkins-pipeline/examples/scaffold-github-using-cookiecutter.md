---
sidebar_position: 2
---

# 使用 Cookiecutter 为 GitHub 仓库搭建脚手架

本示例演示了如何使用[Cookiecutter Template](https://www.cookiecutter.io/templates) 通过 Port Actions 快速为 GitHub 仓库搭建脚手架。

此外，由于 cookiecutter 是一个开源项目，您可以制作自己的项目模板，了解更多信息请访问[here](https://cookiecutter.readthedocs.io/en/2.0.2/tutorials.html#create-your-very-own-cookiecutter-project-template) 。

## 示例 - golang 模板脚手架

请按照以下步骤开始使用 Golang 模板: 

1. 创建以下 Jenkins 认证: 
    1. `GITHUB_TOKEN` - 具有创建版本库权限的[fine-grained PAT](https://github.com/settings/tokens?type=beta) 。
    2. `PORT_CLIENT_ID` - Port客户端 ID[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    3. `PORT_CLIENT_SECRET` - Port客户端secret[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    ::注意
    使用 `Secret text` 作为证书类型。
    :::
2.创建具有以下属性的Port蓝图: 

:::note 请记住，这可以是你想要的任何蓝图，这只是一个例子。

:::

```json showLineNumbers
{
  "identifier": "microservice",
  "title": "Microservice",
  "icon": "Microservice",
  "schema": {
    "properties": {
      "description": {
        "title": "Description",
        "type": "string"
      },
      "url": {
        "title": "URL",
        "format": "url",
        "type": "string"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

3.使用以下 JSON 定义创建 Port 操作:  :::note 请确保替换 JENKINS_URL 和 JOB_TOKEN 的占位符。 :::: 

```json showLineNumbers
[
  {
    "identifier": "scaffold",
    "title": "Scaffold Golang Microservice",
    "description": "Scaffold a new Microservice from a Cookiecutter teplate",
    "icon": "Go",
    "userInputs": {
      "properties": {
        "repo_name": {
          "icon": "Microservice",
          "title": "Repo Name",
          "type": "string"
        },
        "github_org_name": {
          "icon": "Github",
          "title": "Github Org Name",
          "type": "string"
        }
      },
      "required": ["repo_name", "github_org_name"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "agent": false,
      "url": "https://<JENKINS_URL>/generic-webhook-trigger/invoke?token=<JOB_TOKEN>",
      "synchronized": false,
      "method": "POST"
    },
    "trigger": "CREATE"
  }
]
```

4.用以下配置创建一个 Jenkins Pipeline: 
    1.[Enable webhook trigger for a pipeline](../jenkins-pipeline.md#enabling-webhook-trigger-for-a-pipeline)
    2.[Define variables for a pipeline](../jenkins-pipeline.md#defining-variables) 定义 REPO_NAME、GITHUB_ORG_NAME 和 RUN_ID 变量。
    ![Define Vars](../../../../../static/img/self-service-actions/setup-backend/jenkins-pipeline/scaffold-jenkins-vars.png)
    3.[Token Setup](../jenkins-pipeline.md#token-setup): 定义令牌，使其与 Port Action 中配置的 `JOB_TOKEN` 匹配。
5.创建包含以下内容的 Jenkins Pipeline: 

<details>
<summary>Jenkins Pipeline Script</summary>

```yml showLineNumbers
import groovy.json.JsonSlurper

pipeline {
    agent any

    environment {
        COOKIECUTTER_TEMPLATE = 'https://github.com/lacion/cookiecutter-golang'
        REPO_NAME = "${REPO_NAME}"
        GITHUB_ORG_NAME = "${GITHUB_ORG_NAME}"
        SCAFFOLD_DIR = "scaffold_${REPO_NAME}"
        PORT_ACCESS_TOKEN = ""
        PORT_BLUEPRINT_ID = "microservice"
        PORT_RUN_ID = "${RUN_ID}"
    }

    stages {
        stage('Get access token') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'PORT_CLIENT_ID', variable: 'PORT_CLIENT_ID'),
                        string(credentialsId: 'PORT_CLIENT_SECRET', variable: 'PORT_CLIENT_SECRET')
                    ]) {
                        // Execute the curl command and capture the output
                        def result = sh(returnStdout: true, script: """
                            accessTokenPayload=\$(curl -X POST \
                                -H "Content-Type: application/json" \
                                -d '{"clientId": "${PORT_CLIENT_ID}", "clientSecret": "${PORT_CLIENT_SECRET}"}' \
                                -s "https://api.getport.io/v1/auth/access_token")
                            echo \$accessTokenPayload
                        """)

                        // Parse the JSON response using JsonSlurper
                        def jsonSlurper = new JsonSlurper()
                        def payloadJson = jsonSlurper.parseText(result.trim())

                        // Access the desired data from the payload
                        PORT_ACCESS_TOKEN = payloadJson.accessToken
                    }

                }
            }
        } // end of stage Get access token

        stage('Create Github Repository') {
            steps {
                script {
                    def logs_report_response = sh(script: """
                        curl -X POST \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"message": "Creating GitHub repository: ${REPO_NAME} in GitHub org: ${GITHUB_ORG_NAME}..."}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}/logs"
                    """, returnStdout: true)

                    println(logs_report_response)
                }
                script {
                    withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                        sh """

                            curl -i -H 'Authorization: token ${GITHUB_TOKEN}' \\
                            -d '{
                                "name": "${REPO_NAME}", "private": true
                                }' \\
                            https://api.github.com/orgs/${GITHUB_ORG_NAME}/repos
                        """
                    }
                }
            }
        } // end of stage Create Github Repository

        stage('Scaffold Cookiecutter Template') {
            steps {
                script {
                    def logs_report_response = sh(script: """
                        curl -X POST \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"message": "Scaffolding ${REPO_NAME}..."}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}/logs"
                    """, returnStdout: true)

                    println(logs_report_response)
                }
                script {
                    withCredentials([
                        string(credentialsId: 'GITHUB_USERNAME', variable: 'GITHUB_USERNAME'),
                        string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')
                    ]) {
                        def yamlContent = """
default_context:
  full_name: "Full Name"
  github_username: "githubuser"
  app_name: "${REPO_NAME}"
  project_short_description": "A Golang project."
  docker_hub_username: "dockerhubuser"
  docker_image: "dockerhubuser/alpine-base-image:latest"
  docker_build_image: "dockerhubuser/alpine-golang-buildimage"
"""
                    // Write the YAML content to a file
                    writeFile(file: 'cookiecutter.yaml', text: yamlContent)

                        sh("""
                            rm -rf ${SCAFFOLD_DIR} ${REPO_NAME}
                            git clone https://${GITHUB_USERNAME}:${GITHUB_TOKEN}@github.com/${GITHUB_ORG_NAME}/${REPO_NAME}

                            cookiecutter ${COOKIECUTTER_TEMPLATE} --output-dir ${SCAFFOLD_DIR} --no-input --config-file cookiecutter.yaml -f

                            rm -rf ${SCAFFOLD_DIR}/${REPO_NAME}/.git*
                            cp -r ${SCAFFOLD_DIR}/${REPO_NAME}/* "${REPO_NAME}/"

                            cd ${REPO_NAME}
                            git config user.name "Jenkins Pipeline Bot"
                            git config user.email "jenkins-pipeline[bot]@users.noreply.jenkins.com"
                            git add .
                            git commit -m "Scaffolded project ${REPO_NAME}"
                            git push -u origin main
                            cd ..

                            rm -rf ${SCAFFOLD_DIR} ${REPO_NAME}
                        """)
                    }

                }
            }
        } // end of stage Clone Cookiecutter Template

        stage('CREATE Microservice entity') {
            steps {
                script {
                    def logs_report_response = sh(script: """
                        curl -X POST \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"message": "Creating ${REPO_NAME} Microservice Port entity..."}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}/logs"
                    """, returnStdout: true)

                    println(logs_report_response)
                }
                script {
                    def status_report_response = sh(script: """
    					curl --location --request POST "https://api.getport.io/v1/blueprints/$PORT_BLUEPRINT_ID/entities?upsert=true&run_id=$PORT_RUN_ID&create_missing_related_entities=true" \
        --header "Authorization: Bearer $PORT_ACCESS_TOKEN" \
        --header "Content-Type: application/json" \
        --data-raw '{
    			"identifier": "${REPO_NAME}",
    			"title": "${REPO_NAME}",
    			"properties": {"description":"${REPO_NAME} golang project","url":"https://github.com/${GITHUB_ORG_NAME}/${REPO_NAME}"},
    			"relations": {}
    		}'

                    """, returnStdout: true)

                    println(status_report_response)
                }
            }
        } // end of stage CREATE Microservice entity

        stage('Update Port Run Status') {
            steps {
                script {
                    def status_report_response = sh(script: """
                        curl -X PATCH \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"status":"SUCCESS", "message": {"run_status": "Scaffold Jenkins Pipeline completed successfully!"}}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}"
                    """, returnStdout: true)

                    println(status_report_response)
                }
            }
        } // end of stage Update Port Run Status
    }

    post {

        failure {
            // Update Port Run failed.
            script {
                def status_report_response = sh(script: """
                    curl -X PATCH \
                        -H "Content-Type: application/json" \
                        -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                        -d '{"status":"FAILURE", "message": {"run_status": "Failed to Scaffold ${REPO_NAME}"}}' \
                            "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}"
                """, returnStdout: true)

                println(status_report_response)
            }
        }

        // Clean after build
        always {
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: false,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                               [pattern: '.propsfile', type: 'EXCLUDE']])
        }
    }
}
```

</details>

6.从 Port 应用程序的[Self-service](https://app.getport.io/self-serve) 标签触发操作。
