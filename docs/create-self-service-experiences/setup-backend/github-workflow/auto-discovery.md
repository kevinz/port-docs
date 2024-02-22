---
sidebar_position: 1
---

# 汽车发现

操作自动发现功能可自动发现 GitHub 工作流操作并将其与 Port 同步。

您可以运行该工具，它会自动将您的 GitHub 动作工作流导入 Port，并将其添加为指定蓝图中的自助服务动作。

有关自动发现工具的详细信息，请参阅[project's repository](https://github.com/port-labs/actions-auto-discovery) 中的 README 文件。

# Usage 示例

```bash showLineNumbers
#!/bin/bash
export PORT_CLIENT_ID="PORT_CLIENT_ID"
export PORT_CLIENT_SECRET="PORT_CLIENT_SECRET"
export GITHUB_TOKEN="GITHUB_TOKEN"
export GITHUB_ORG_NAME="GITHUB_ORG_NAME"
export BLUEPRINT_IDENTIFIER="BLUEPRINT_IDENTIFIER"
export ACTION_TRIGGER="TRIGGER" # optional, defaults to CREATE
export REPO_LIST="repo1,repo2,repo3" # optional, defaults to all repositories in the organization(*), comma separated list of repositories repo1,repo2,repo3

curl -s https://raw.githubusercontent.com/port-labs/actions-auto-discovery/main/github-actions/sync.sh | bash
```