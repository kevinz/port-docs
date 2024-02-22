---

sidebar_position: 5

---

# 定制 CRD

除了内置的 Kubernetes API 资源，还可以使用 Port 的 Kubernetes 导出器从 Kubernetes 集群导出和映射[CRDs](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) 。

我们创建了几个预建的 Port k8s[exporter templates](./templates/) ，使用 CRD 导出，以便在使用常见的第三方 k8s 工具时快速上手。

还可以为 Kubernetes 集群中可能使用的任何其他 CRD 编写自定义 CRD 导出模板。 要获得集群中可以查询的所有资源类型，可以运行

```bash showLineNumbers
$ kubectl api-resources
```

例如，要了解如何映射 ArgoCD 资源，可以运行

```bash showLineNumbers
$ kubectl api-resources | grep -i argo
applications app,apps argoproj.io/v1alpha1 true Application
applicationsets appset,appsets argoproj.io/v1alpha1 true ApplicationSet
appprojects appproj,appprojs argoproj.io/v1alpha1 true AppProject
```

然后，要映射 ArgoCD "应用程序"，请将匹配的资源种类添加到导出器配置中: 

```yaml showLineNumbers
- kind: argoproj.io/v1alpha1/applications
  identifier: .metadata.name
  blueprint: '"argoApp"'
  properties:
  ...
```