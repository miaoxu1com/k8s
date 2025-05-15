

[TOC]

# Helm 概念

**在使用 Helm 的过程中，需要理解如下的几个核心的概念：**

| 概念           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| **Chart**      | 一个 Helm 包，其中包含了运行一个应用所需要的镜像、依赖和资源定义等，还可能包含 Kubernetes 集群中的服务定义，类似 Homebrew 中的 formula、APT 的 dpkg 或者 Yum 的 rpm 文件 |
| **Repository** | 存储 Helm Charts 的地方                                        |
| **Release**    | Chart 在 k8s 上运行的 Chart 的一个实例，例如，如果一个 MySQL Chart 想在服务器上运行两个数据库，可以将这个 Chart 安装两次，并在每次安装中生成自己的 Release 以及 Release 名称。 |
| **Value**      | Helm Chart 的参数，用于配置 Kubernetes 对象                     |
| **Template**   | 使用 Go 模板语言生成 Kubernetes 对象的定义文件                   |
| **Namespace**  | Kubernetes 中用于隔离资源的逻辑分区                           |

# Helm 的使用

## 安装 Helm

**首先需要在本地机器或 Kubernetes 集群上安装 Helm**。

**Helm 只能在 k8s 中使用，不能在 docker 中使用，Helm是模板，有模板语法，模板函数，支持逻辑判断，可以实现动态生成目标资源清单,现在很多公有云服务商，都建立自己的模板市场，不用用户初始化yaml文件，提供一个yaml模板，直接修改定制自己的资源**

**docker不支持helm，需要自己编写yaml模板替换脚本，实现动态渲染资源清单**

可以从 Helm 官方网站下载适合自己平台的二进制文件，或使用包管理器安装 Helm，安装教程参考 [https://helm.sh](https://helm.sh/)

## 创建 Chart

**使用 helm create 命令创建一个新的 Chart**，Chart 目录包含描述应用程序的文件和目录，包括 Chart.yaml、values.yaml、templates 目录等；

例如：在本地机器上使用 `helm create` 命令创建一个名为 `wordpress` 的 `Chart`：
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/7cc7867064d54192a074aae9e6c8c3a9.png?x-oss-process=image/resize,w_1400/format,webp)
在当前文件夹，可以看到创建了一个 wordpress 的目录，且里面的内容如下：
![在这里插入图片描述](https://ucc.alicdn.com/images/user-upload-01/44a1dcb4b3354b1c9aa3fc42a1fc41bc.png?x-oss-process=image/resize,w_1400/format,webp)

## 创建/编辑 Chart 配置

**使用编辑器编辑 Chart 配置文件，包括 Chart.yaml 和 values.yaml**。

### Chart.yaml

> `Chart.yaml` 包含 `Chart` 的元数据和依赖项

Chart.yaml 的模板及注释如下：

```yaml
apiVersion: chart API 版本 （必需）  #必须有
name: chart名称 （必需）     # 必须有 
version: 语义化2 版本（必需） # 必须有

kubeVersion: 兼容Kubernetes版本的语义化版本（可选）
description: 一句话对这个项目的描述（可选）
type: chart类型 （可选）
keywords:
  - 关于项目的一组关键字（可选）
home: 项目home页面的URL （可选）
sources:
  - 项目源码的URL列表（可选）
dependencies: # chart 必要条件列表 （可选）
  - name: chart名称 (nginx)
    version: chart版本 ("1.2.3")
    repository: （可选）仓库URL ("https://example.com/charts") 或别名 ("@repo-name")
    condition: （可选） 解析为布尔值的yaml路径，用于启用/禁用chart (e.g. subchart1.enabled )
    tags: # （可选）
      - 用于一次启用/禁用 一组chart的tag
    import-values: # （可选）
      - ImportValue 保存源值到导入父键的映射。每项可以是字符串或者一对子/父列表项
    alias: （可选） chart中使用的别名。当你要多次添加相同的chart时会很有用

maintainers: # （可选） # 可能用到
  - name: 维护者名字 （每个维护者都需要）
    email: 维护者邮箱 （每个维护者可选）
    url: 维护者URL （每个维护者可选）

icon: 用做icon的SVG或PNG图片URL （可选）
appVersion: 包含的应用版本（可选）。不需要是语义化，建议使用引号
deprecated: 不被推荐的chart （可选，布尔值）
annotations:
  example: 按名称输入的批注列表 （可选）.
```

举例：

```yaml
name: nginx-helm
apiVersion: v1
version: 1.0.0
```

###  values.yaml

`values.yaml` 包含应用程序的默认配置值，举例：

```yaml
image:
  repository: nginx
  tag: '1.19.8'
```

### templates

在模板中引入 values.yaml 里的配置，在模板文件中可以通过 .VAlues 对象访问到，例如：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-helm-{{ .Values.image.repository }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-helm
  template:
    metadata:
      labels:
        app: nginx-helm
    spec:
      containers:
      - name: nginx-helm
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        ports:
        - containerPort: 80
          protocol: TCP
```

## 打包 Chart

使用 helm package 命令将 Chart 打包为一个 tarball 文件，例如在 wordpress 目录中使用 helm package 命令将 Chart 打包为一个 tarball 文件：

```shell
helm package wordpress/
```

这将生成一个名为 `wordpress-0.1.0.tgz` 的 `tarball` 文件。

## 发布 Chart

将打包好的 Chart 发布到一个 Helm Repository 中。可以使用 helm repo add 命令添加一个 Repository，然后使用 helm push 命令将 Chart 推送到 Repository 中，例如：

```shell
helm repo add myrepo https://example.com/charts
helm push wordpress-0.1.0.tgz myrepo
```

##  安装 Release

使用 helm install 命令安装 Chart 的 Release，可以通过命令行选项或指定 values.yaml 文件来配置 Release，例如：

```shell
helm install mywordpress myrepo/wordpress
```

这将在 `Kubernetes` 集群中创建一个名为 `mywordpress` 的 `Release`，包含 `WordPress` 应用程序和 `MySQL` 数据库。

## 管理 Release

使用 **helm ls** 命令查看当前运行的 Release 列表，例如：

```shell
helm upgrade mywordpress myrepo/wordpress --set image.tag=5.7.3-php8.0-fpm-alpine
```

这将升级 `mywordpress` 的 `WordPress` 应用程序镜像版本为 `5.7.3-php8.0-fpm-alpine`。

------

可以使用 `helm rollback` 命令回滚到先前版本，例如：

```shell
helm rollback mywordpress 1
```

这将回滚 `mywordpress` 的版本到 1。

##  更新 Chart

在应用程序更新时，可以更新 Chart 配置文件和模板，并使用 helm package 命令重新打包 Chart。然后可以使用 helm upgrade 命令升级已安装的 Release，可以按照以下步骤更新 Chart：

1. **在本地编辑 Chart 配置或添加新的依赖项**；
2. **使用 helm package 命令打包新的 Chart 版本**；
3. **使用 helm push 命令将新的 Chart 版本推送到 Repository 中**；
4. **使用 helm repo update 命令更新本地或远程的 Helm Repository**；
5. **使用 helm upgrade 命令升级现有 Release 到新的 Chart 版本**。

例如，可以使用以下命令更新 WordPress 的 Chart 版本：

```shell
helm upgrade mywordpress myrepo/wordpress --version 0.2.0
```

这将升级 mywordpress 的 Chart 版本到 0.2.0，其中包括新的配置和依赖项。

------

如果需要删除一个 Release，可以使用 helm uninstall 命令。例如：

```shell
helm uninstall mywordpress
```

这将删除名为 mywordpress 的 Release，同时删除 WordPress 应用程序和 MySQL 数据库。

------

如果需要删除与 Release 相关的 PersistentVolumeClaim，可以使用 helm uninstall 命令的--delete-data 选项，例如：

```shell
helm uninstall mywordpress --delete-data
```

这将删除名为 mywordpress 的 Release，并删除与之相关的所有 PersistentVolumeClaim。

# 05 Helm 的执行安装顺序

Helm 按照以下顺序安装资源：

- Namespace
- NetworkPolicy
- ResourceQuota
- LimitRange
- PodSecurityPolicy
- PodDisruptionBudget
- ServiceAccount
- Secret
- SecretList
- ConfigMap
- StorageClass
- PersistentVolume
- PersistentVolumeClaim
- CustomResourceDefinition
- ClusterRole
- ClusterRoleList
- ClusterRoleBinding
- ClusterRoleBindingList
- Role
- RoleList
- RoleBinding
- RoleBindingList
- Service
- DaemonSet
- Pod
- ReplicationController
- ReplicaSet
- Deployment
- HorizontalPodAutoscaler
- StatefulSet
- Job
- CronJob
- Ingress
- APIService

Helm 客户端不会等到所有资源都运行才退出，可以使用 helm status 来追踪 release 的状态，或是重新读取配置信息：

```shell
# [root@k8s-master nginx-helm-v2]# helm status mynginx
NAME: mynginx
LAST DEPLOYED: Fri Oct 29 14:27:32 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

# 06 Helm命令汇总

在最后，附上所有helm的命令，直接控制台使用 `helm --help`即可查看：

``` shell
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment variables:

| Name                               | Description                                                                                       |
|------------------------------------|---------------------------------------------------------------------------------------------------|
| $HELM_CACHE_HOME                   | set an alternative location for storing cached files.                                             |
| $HELM_CONFIG_HOME                  | set an alternative location for storing Helm configuration.                                       |
| $HELM_DATA_HOME                    | set an alternative location for storing Helm data.                                                |
| $HELM_DEBUG                        | indicate whether or not Helm is running in Debug mode                                             |
| $HELM_DRIVER                       | set the backend storage driver. Values are: configmap, secret, memory, sql.                       |
| $HELM_DRIVER_SQL_CONNECTION_STRING | set the connection string the SQL storage driver should use.                                      |
| $HELM_MAX_HISTORY                  | set the maximum number of helm release history.                                                   |
| $HELM_NAMESPACE                    | set the namespace used for the helm operations.                                                   |
| $HELM_NO_PLUGINS                   | disable plugins. Set HELM_NO_PLUGINS = 1 to disable plugins.                                        |
| $HELM_PLUGINS                      | set the path to the plugins directory                                                             |
| $HELM_REGISTRY_CONFIG              | set the path to the registry config file.                                                         |
| $HELM_REPOSITORY_CACHE             | set the path to the repository cache directory                                                    |
| $HELM_REPOSITORY_CONFIG            | set the path to the repositories file.                                                            |
| $KUBECONFIG                        | set an alternative Kubernetes configuration file (default "~/.kube/config ")                       |
| $HELM_KUBEAPISERVER                | set the Kubernetes API Server Endpoint for authentication                                         |
| $HELM_KUBECAFILE                   | set the Kubernetes certificate authority file.                                                    |
| $HELM_KUBEASGROUPS                 | set the Groups to use for impersonation using a comma-separated list.                             |
| $HELM_KUBEASUSER                   | set the Username to impersonate for the operation.                                                |
| $HELM_KUBECONTEXT                  | set the name of the kubeconfig context.                                                           |
| $HELM_KUBETOKEN                    | set the Bearer KubeToken used for authentication.                                                 |
| $HELM_KUBEINSECURE_SKIP_TLS_VERIFY | indicate if the Kubernetes API server's certificate validation should be skipped (insecure)       |
| $HELM_KUBETLS_SERVER_NAME          | set the server name used to validate the Kubernetes API server certificate                        |
| $HELM_BURST_LIMIT                  | set the default burst limit in the case the server contains many CRDs (default 100, -1 to disable)|

Helm stores cache, configuration, and data based on the following configuration order:

- If a HELM_*_HOME environment variable is set, it will be used
- Otherwise, on systems supporting the XDG base directory specification, the XDG variables will be used
- When no other location is set a default location will be used based on the operating system

By default, the default directories depend on the Operating System. The defaults are listed below:

| Operating System | Cache Path                | Configuration Path             | Data Path               |
|------------------|---------------------------|--------------------------------|-------------------------|
| Linux            | $HOME/.cache/helm         | $ HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | $ HOME/Library/Preferences/helm | $HOME/Library/helm      |
| Windows          | %TEMP%\helm               | %APPDATA%\helm                 | %APPDATA%\helm          |

Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart's dependencies
  env         helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examine a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  push        push a chart to remote
  registry    login to or logout from a registry
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      display the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information

Flags:
      --burst-limit int                 client-side default throttling limit (default 100)
      --debug                           enable verbose output
  -h, --help                            help for helm
      --kube-apiserver string           the address and the port for the Kubernetes API server
      --kube-as-group stringArray       group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --kube-as-user string             username to impersonate for the operation
      --kube-ca-file string             the certificate authority file for the Kubernetes API server connection
      --kube-context string             name of the kubeconfig context to use
      --kube-insecure-skip-tls-verify   if true, the Kubernetes API server's certificate will not be checked for validity. This will make your HTTPS connections insecure
      --kube-tls-server-name string     server name to use for Kubernetes API server certificate validation. If it is not provided, the hostname used to contact the server is used
      --kube-token string               bearer token used for authentication
      --kubeconfig string               path to the kubeconfig file
  -n, --namespace string                namespace scope for this request
      --registry-config string          path to the registry config file (default "/Users/yanglinwei/Library/Preferences/helm/registry/config.json")
      --repository-cache string         path to the file containing cached repository indexes (default "/Users/yanglinwei/Library/Caches/helm/repository")
      --repository-config string        path to the file containing repository names and URLs (default "/Users/yanglinwei/Library/Preferences/helm/repositories.yaml")

Use "helm [command] --help" for more information about a command.
```
