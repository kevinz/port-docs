---

sidebar_position: 6

---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";
import DeleteDependents from '/docs/generalTemplates/_delete_dependents_kubernetes_explanation_template.md'
import CreateMissingRelatedEntities from '/docs/generalTemplates/_create_missing_related_entities_kubernetes_explanation_template.md'

# 高级

k8s 输出程序支持额外的 flag 和提供额外配置源的选项，从而可以更方便地根据自己的喜好配置其行为。

## 所需配置

每次安装/升级 k8s 输出程序时都需要以下参数: 


| Parameter                         | Description                                                               |
| --------------------------------- | ------------------------------------------------------------------------- |
| `secret.secrets.portClientId`     | Port Client ID                                                            |
| `secret.secrets.portClientSecret` | Port Client Secret                                                        |


## 高级安装参数

可使用以下高级配置参数: 

<Tabs groupId="advanced" queryString="current-config-param" defaultValue="resyncInterval" values={[
{label: "Resync Interval", value: "resyncInterval"},
{label: "State Key", value: "stateKey"},
{label: "Verbosity (Log Level)", value: "verbosity"},
{label: "Event listener type", value: "eventListenerType"},
]} >

<TabItem value="resyncInterval">

resyncInterval "参数指定为所有已知现有对象(除新的集群事件外)发送重复同步事件的时间间隔(以分钟为单位)。

* 默认值:  `0`(禁用重新同步)
* 被引用: 每 X 分钟重新同步一次。在集群内报告有关系的实体时，如果在 Port 中创建了相关目标实体之前就报告了该实体，则此参数非常有用。初始同步将失败，但随后当目标实体可用时，实体创建将成功。

</TabItem>

<TabItem value="stateKey">

stateKey "参数指定了每个 k8s 输出程序安装的唯一状态密钥。 根据现有的输出程序应用程序配置，可以删除输出程序创建的、不应再同步(的陈旧 Port 实体。 输出程序将在 pod 初始化过程中检查待处理的删除，并对集群中的删除事件做出响应。

* 默认值: `""`。
    - 为空时，将自动生成一个 `UUID` 并保存在 configmaps 中。更改状态密钥会导致现有的输出程序失去对之前从集群中报告的实体的跟踪，因此不会从 Port 中删除这些实体。
* 被引用: 删除过时的 Port 实体。例如
    - 从出口程序配置中删除整个资源(如 `pods`)，也会从软件目录中删除它们。
    - 修改实体的标识符将导致删除过时的实体，并用正确的标识符重新创建。

</TabItem>

<TabItem value="eventListenerType">

k8s 输出程序为多个事件监听器提供支持。 事件监听器用于接收来自 Port 的事件和重同步请求，并将其转发给输出程序。

通过配置事件监听器，集成将监听Port发送的以下事件并作出反应: 

* **配置更新** - 集成将使用新配置的数据从 k8s 集群重新同步信息
* **重新同步请求** - 集成将根据现有配置从 k8s 集群到 Port 执行数据重新同步

支持以下事件监听器类型

* **POLLING** - 集成将自动查询 Port 中集成配置的更新，并在检测到更改时执行重新同步。
重新同步。
* **KAFKA** - 集成将从您的专用 Kafka 主题接收传入的重新同步请求。

可找到可用的事件侦听器配置参数[here](https://github.com/port-labs/helm-charts/blob/main/charts/port-k8s-exporter/README.md#chart)

:::caution  多个出口程序实例 目前可用的事件监听器不支持同一出口程序的多个实例

:::

:::danger  resync 如果集成在执行重新同步时收到重新同步事件，当前正在运行的重新同步将被中止，并启动新的重新同步进程。

如果新的重新同步触发器持续中止正在运行的重新同步，这意味着您的集成从未完成完整的重新同步过程(这意味着集群中的某些信息可能永远不会出现在 Port 中)。

:::

</TabItem>

<TabItem value="verbosity">

verbosity` 参数用于控制 k8s 输出程序 pod 中 info logging 的冗长程度。

* 默认:  `0`(显示所有信息和错误日志，包括成功更新的信息日志)
* 被引用: 如果想清除成功实体更新的信息日志，请将值设为 `-1`。错误日志和一些信息日志(初始化和拆卸日志)将被报告。

</TabItem>

</Tabs>

## 安全配置

可以修改以下安全参数，让 k8s 输出程序对集群有更细粒度的访问权限: 


| Parameter               | Description                                                                                                                                                                             | Default |
| ----------------------- |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| ------- |
| `clusterRole.apiGroups` | The API groups that the K8s Exporter can access. Make sure to grant access to the relevant API groups, with respect to the resources that you've configured in the resource mapping     | `{'*'}` |
| `clusterRole.resources` | The resources that the K8s Exporter can access. Make sure to grant access to the relevant resources, with respect to the resources that you've configured in the resource mapping | `{'*'}` |


## 覆盖配置

安装 k8s 输出程序时，可以覆盖 `helm upgrade` 命令中的默认值: 

通过被引用 `--set` flag，你可以在安装/升级出口程序时覆盖特定的出口程序配置参数: 

```bash showLineNumbers
helm upgrade --install k8s-exporter port-labs/port-k8s-exporter \
    --create-namespace --namespace port-k8s-exporter \
    --set secret.secrets.portClientId="YOUR_PORT_CLIENT_ID"  \
    --set secret.secrets.portClientSecret="YOUR_PORT_CLIENT_SECRET"  \
    --set stateKey="k8s-exporter"  \
    # highlight-next-line
    --set eventListenerType="KAFKA"  \
    --set extraEnv=[{"name":"CLUSTER_NAME","value":"my-cluster"}]
```

例如，从[security configuration](#security-configuration) 部分设置参数: 

```bash showLineNumbers
--set clusterRole.apiGroups="{argoproj.io,'',apps}" \
--set clusterRole.resources="{rollouts,pods,replicasets}"
```

## 所有配置参数

* 使用 helm chart 时可用配置参数的完整列表可查阅[here](https://github.com/port-labs/helm-charts/tree/main/charts/port-k8s-exporter#chart) ；
* `values.yml` 文件骨架示例见[here](https://github.com/port-labs/helm-charts/blob/main/charts/port-k8s-exporter/values.yaml) 。

## 额外的环境变量

要将额外的环境变量传递给出口程序的运行时，可以使用安装时提供的 helm chart。 可以通过以下两种方式之一来实现: 

1. 被引用 Helm 的 `--set` flag: 

```sh showLineNumbers
helm upgrade --install <MY_INSTALLATION_NAME> port-labs/port-k8s-exporter \
  # Standard installation flags
  # ...
  --set "extraEnv[0].name"=HTTP_PROXY \
  --set "extraEnv[0].value"=http://my-proxy.com:1111
```

2.Helm `values.yaml` 文件: 

```yaml showLineNumbers
# The rest of the configuration
# ...
extraEnvs:
  - name: HTTP_PROXY
    value: http://my-proxy.com:1111
```

#### 代理配置

#### `http_proxy` &amp; `https_proxy`

`HTTP_PROXY` 和 `HTTPS_PROXY` 是环境变量，分别用于指定处理 HTTP 或 HTTPS 的代理服务器。 分配给这些设置的值应为代理服务器的 URL。

例如

```sh showLineNumbers
HTTP_PROXY=http://my-proxy.com:1111
HTTPS_PROXY=http://my-proxy.com:2222
```

###`NO_PROXY`

NO_PROXY "允许将某些地址列入黑名单，使其无法通过代理处理。 该变量接受一个逗号分隔的主机名或 urls 列表。

例如

```sh showLineNumbers
NO_PROXY=http://127.0.0.1,google.com
```

## 高级资源映射配置

<Tabs groupId="advanced" queryString="current-resource-mapping-configuration" defaultValue="deleteDependents" values={[
{label: "Delete Dependents", value: "deleteDependents"},
{label: "Create Missing Related Entities", value: "createMissingRelatedEntities"},
]} >

<TabItem value="deleteDependents">
<DeleteDependents/>
- Default: `false` (disabled)
- Use case: Deletion of dependent Port entities. Must be enabled if you want to delete a target entity (and its source entities) when the entity's blueprint has required relations.
</TabItem>
<TabItem value="createMissingRelatedEntities">
<CreateMissingRelatedEntities/>
- Default: `false` (disabled)
- Use case: Creation of missing related Port entities. For example:
  - Creation of related entity that has no matching resource kind in K8s, like `cluster`;
  - Creation of an entity and its related entity, even though the related entity doesn't exist yet in Port.
</TabItem>
</Tabs>