---
sidebar_position: 2
---

# Usage

在使用执行代理时，您需要在 `url` 字段中提供一个指向服务(例如 REST API)的 URL，该服务将接受调用事件。

* 服务可以是运行在专用网络内的专用服务；
* 或者，它可以是一个可从公共互联网访问的公共服务(**注** 在这种情况下，执行代理需要相应的出站网络规则，使其能够联系公共服务)。

:::note 
**重要**:  要被引用**Port 执行代理**，需要进行配置: 

<!-- TODO: add back the URLs here for changelog destination -->

* [Self-Service Action invocation method](/create-self-service-experiences/self-service-actions-deep-dive/self-service-actions-deep-dive.md#invocation-method-structure-fields) / 更改日志目的地 `type` 字段值应等于 `WEBhook`。
* [Self-Service Action invocation method](/create-self-service-experiences/self-service-actions-deep-dive/self-service-actions-deep-dive.md#invocation-method-structure-fields) / 更改日志`代理`字段值应等于`true`。

例如

```json showLineNumbers
{ "type": "WEBHOOK", "agent": true, "url": "URL_TO_API_INSIDE_YOUR_NETWORK" }
```

:::

干得好！**Port Agent** 现在已在您的环境中运行，并将触发您配置的任何 webhook(用于自助服务操作或软件目录中的更改)。

检测到新的调用时，代理将从 Kafka 主题中提取该调用，并将其转发到私人网络中的内部 API。

![Port Execution Agent Logs](/img/self-service-actions/port-execution-agent/portAgentLogs.png)

## 高级配置

使用 Port 代理时，某些环境需要特殊配置，包括使用自签名证书和/或代理。

Port 的代理程序使用 Python 的[requests](https://requests.readthedocs.io/en/latest/) 库。这允许使用环境变量传递高级配置。

要使用代理的 helm chart 引用环境变量，可以选择

1. 被引用 Helm 的 `--set` flag: 

```sh showLineNumbers
helm upgrade --install <MY_INSTALLATION_NAME> port-labs/port-ocean \
  # Standard installation flags
  # ...
  --set env.normal.VAR_NAME=VAR_VALUE
```

2.Helm `values.yaml` 文件: 

```yaml showLineNumbers
# The rest of the configuration
# ...
env:
  normal:
    VAR_NAME: VAR_VALUE
```

#### 代理配置

#### `http_proxy`、`https_proxy` 和 `all_proxy

HTTP_PROXY"、"HTTPS_PROXY "和 "ALL_PROXY "是环境变量，分别用于指定处理 HTTP、HTTPS 或所有类型请求的代理服务器。 分配给这些设置的值应该是代理服务器的 URL。

例如

```sh showLineNumbers
HTTP_PROXY=http://my-proxy.com:1111
HTTPS_PROXY=http://my-proxy.com:2222
ALL_PROXY=http://my-proxy.com:3333
```

#### `NO_PROXY`

NO_PROXY "允许将某些地址列入黑名单，使其无法通过代理处理。 该变量接受一个逗号分隔的主机名或 urls 列表。

例如

```sh showLineNumbers
NO_PROXY=http://127.0.0.1,google.com
```

欲了解更多信息，请访问请求[proxy configuration documentation](https://requests.readthedocs.io/en/latest/user/advanced/#proxies) 。

### SSL 环境配置

#### `requests_ca_bundle`

REQUESTS_CA_BUNDLE "是一个环境变量，用于指定一个自定义的证书颁发机构(CA)捆绑包，以验证 HTTPS 请求中的 SSL/TLS 证书。

将 `REQUESTS_CA_BUNDLE` 设为 CA 包的文件路径，其中应包含一个或多个 PEM 格式的 CA 证书。

例如

```sh
REQUESTS_CA_BUNDLE=/path/to/cacert.pem
```

此配置指示 `requests` 库使用指定的 CA 捆绑包进行 SSL/TLS 证书验证，覆盖系统默认设置。 它对于信任自签证书或来自私有 CA 的证书非常有用。

## 接下来的步骤

请按照以下指南之一进行操作: 

* [Self-Service Actions Deep Dive](/create-self-service-experiences/self-service-actions-deep-dive/self-service-actions-deep-dive.md) - 设置蓝图和自助操作。
* [Changelog Listener](/create-self-service-experiences/setup-backend/webhook/examples/changelog-listener.md) - 创建一个带有 `changelogDestination` 的蓝图，以监听软件目录中的变更并采取相应行动。
* [GitLab Pipeline Trigger](/create-self-service-experiences/setup-backend/gitlab-pipeline/gitlab-pipeline.md) - 创建一个可触发 GitLab Pipelines 执行的操作。
