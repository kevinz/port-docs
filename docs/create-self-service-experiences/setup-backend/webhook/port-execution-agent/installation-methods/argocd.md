---
sidebar_position: 2
---

import FindCredentials from "@site/docs/build-your-software-catalog/sync-data-to-catalog/api/_template_docs/_find_credentials_collapsed.mdx"

# ArgoCD

本页面将引导您利用 ArgoCD 在 Kubernetes 集群中安装 Port 执行代理，并利用其[Helm Capabilities](https://argo-cd.readthedocs.io/en/stable/user-guide/helm/) 。

:::info 

* 您可以观察 Helm 图表和可用参数[here](https://github.com/port-labs/helm-charts/tree/main/charts/port-agent) 。
* 完整的图表版本列表请参阅[Releases](https://github.com/port-labs/helm-charts/releases?q=port-agent&amp;expanded=true) 页面。

:::

## 先决条件

* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) 必须安装才能应用您的配置清单。
* [Helm](https://helm.sh) 必须安装才能使用图表。有关安装的更多详情，请参阅 Helm[documentation](https://helm.sh/docs/intro/install/) 。
* [ArgoCD](https://argoproj.github.io/cd/) 必须安装在 Kubernetes 集群中。有关安装的更多详情，请参阅 ArgoCD 的[documentation](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd) 。
* 您将需要[Port credentials](/build-your-software-catalog/sync-data-to-catalog/api/api.md#find-your-port-credentials) 。
* Port 为您提供与 Kafka 的连接凭证。
* 如果要触发 GitLab Pipelines，您需要有一个[GitLab trigger token](https://docs.gitlab.com/ee/ci/triggers/)

:::tip 

<FindCredentials />

:::

## 安装

1. 在 git 仓库中创建名为 `argocd` 的目录。

```bash
mkdir argocd
```

2.在 `argocd` 目录中为当前安装创建另一个目录。在我们的例子中，我们被用于`my-port-agent`。

```bash
mkdir -p argocd/my-port-agent
```

3.在 `my-port-agent` 目录中创建一个 `values.yaml` 文件，你可以用它来覆盖 helm chart 的值。将更改提交到 git 仓库。
4.创建以下 `my-port-agent.yaml` 配置清单，安装 `my-port-agent` ArgoCD 应用程序: 

:::note 记住要替换 `YOUR_ORG_ID`、`YOUR_KAFKA_COMNSUMER_GROUP`、`YOUR_PORT_CLIENT_ID`、`YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL`。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-port-agent
  namespace: argocd
spec:
  destination:
    namespace: my-port-agent
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-agent
    targetRevision: 0.7.2
    helm:
      valueFiles:
        - $values/argocd/my-port-agent/values.yaml
      parameters:
        - name: env.normal.KAFKA_CONSUMER_GROUP_ID
          value: YOUR_KAFKA_CONSUMER_GROUP
        - name: env.normal.PORT_ORG_ID
          value: YOUR_ORG_ID
        - name: env.secret.PORT_CLIENT_ID
          value: YOUR_PORT_CLIENT_ID
        - name: env.secret.PORT_CLIENT_SECRET
          value: YOUR_PORT_CLIENT_SECRET
  - repoURL: YOUR_GIT_REPO_URL
    targetRevision: main
    ref: values
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

</details>
<br/>

6.使用 `kubectl` 配置应用程序清单: 

```bash
kubectl apply -f my-port-agent.yaml
```

完成！导出器将很快开始以 Port 实体的形式从 Kubernetes 集群创建和更新对象。

## 接下来的步骤

* 请参考[usage guide](/create-self-service-experiences/setup-backend/webhook/port-execution-agent/usage.md) 设置 webhook。
