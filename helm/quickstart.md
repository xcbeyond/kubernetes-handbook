# 快速入门

介绍如何快速上手使用 Helm。

## 1、前提条件

在使用 Helm 前，需具备以下前提条件：

- 一个 Kubernetes 环境

## 2、Helm 版本选择

Helm 是基于 Kubernetes 之上实现的，在选择 Helm 版本时需要考虑对应的 Kubernetes 版本。

当一个 Helm 的新版本发布时，它会针对 Kubernetes 的一个特定的次版本进行编译。比如，Helm 3.0.0 与 Kubernetes 的 1.16.2 的客户端版本交互，一次可以兼容 Kubernetes 1.16。

**对于最新 Helm 版本，建议使用当前 Kubernetes 的最新稳定版本。**

如果您选择了一个 Kubernetes 版本不支持的 Helm 版本，在使用过程中将面临未知的风险。

建议参考下表，以满足与 Kubernetes 的兼容性：

| Helm 版本 | 支持的 Kubernetes 版本 |
| --------- | ---------------------- |
| 3.7.x     | 1.22.x - 1.19.x        |
| 3.6.x     | 1.21.x - 1.18.x        |
| 3.5.x     | 1.20.x - 1.17.x        |
| 3.4.x     | 1.19.x - 1.16.x        |
| 3.3.x     | 1.18.x - 1.15.x        |
| 3.2.x     | 1.18.x - 1.15.x        |

更多参考 [Helm 版本支持策略](https://helm.sh/docs/topics/version_skew/)

## 2、安装

可以通过 [homebrew](https://brew.sh/) （macOS 下包管理工具）下载二进制 Helm 安装包，也可以通过 Github [下载](https://github.com/helm/helm/releases)。

### 2.1 macOS

```sh
$ brew install helm
```

### 2.2 二进制安装

每个 Helm 版本都提供了各种操作系统对应的[二进制版本包](https://github.com/helm/helm/releases)，这些版本可以手动下载和安装。

1. 下载所需的版本。

2. 解压(如：`tar -zxvf helm-v3.7.1-linux-amd64.tar.gz`)。

3. 在解压目录中找到 helm 程序，移动到需要的目录中(如：mv linux-amd64/helm /usr/local/bin/helm)。

更多安装方式参考：[安装 Helm](https://helm.sh/zh/docs/intro/install/)

## 3、初始化

当已经安装好 Helm 之后，可以添加一个 chart 仓库。可从 [Artifact Hub](https://artifacthub.io/packages/search?kind=0) 中查找有效的 Helm chart 仓库。

![Artifact Hub](artifact-hub.png)

> TIP：通过 Artifact Hub 几乎可以找到任何想要的 chart 包，并能找到对应的源码(github 地址)，然后就可以参考、修改为定制化的 chart 啦！

```sh
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

当添加完成，您将可以看到可以被您安装的 charts 列表：

```sh
$ helm search repo bitnami
NAME CHART VERSION APP VERSION DESCRIPTION
bitnami/bitnami-common 0.0.9 0.0.9 DEPRECATED Chart with custom templates used in ...
bitnami/airflow 8.0.2 2.0.0 Apache Airflow is a platform to programmaticall...
bitnami/apache 8.2.3 2.4.46 Chart for Apache HTTP Server
bitnami/aspnet-core 1.2.3 3.1.9 ASP.NET Core is an open-source framework create...
# ... and many more
```

## 4、安装 Chart 示例

通过 `helm install` 命令安装 chart。 Helm 可以通过多种途径查找和安装 chart，但最简单的是安装官方的 bitnami charts。

```sh
$ helm repo update # 确定我们可以拿到最新的 charts 列表
$ helm install bitnami/mysql --generate-name
NAME: mysql-1612624192
LAST DEPLOYED: Sat Feb 6 16:09:56 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES: ...
```

在上面的例子中，`bitnami/mysql` 这个 chart 被发布，名字是 `mysql-1612624192`

您可以通过执行 `helm show chart bitnami/mysql` 命令简单的了解到这个 chart 的基本信息。 或者您可以执行 `helm show all bitnami/mysql` 获取关于该 chart 的所有信息。

每当您执行 `helm install` 的时候，都会创建一个新的发布版本。 所以一个 chart 在同一个集群里面可以被安装多次，每一个都可以被独立的管理和升级。

## 5、卸载

可以使用 `helm uninstall` 命令卸载你的版本。

```sh
$ helm uninstall mysql-1612624192
release "mysql-1612624192" uninstalled
```

该命令会从 Kubernetes 卸载 mysql-1612624192， 它将删除和该版本相关的所有相关资源（service、deployment、 pod 等）甚至版本历史。

如果您在执行 `helm uninstall` 的时候提供 `--keep-history` 选项， Helm 将会保存版本历史。 您可以通过命令查看该版本的信息

```sh
$ helm status mysql-1612624192
Status: UNINSTALLED
...
```

因为 `--keep-history` 选项会让 helm 跟踪你的版本（即使你卸载了他们），所以你可以审计集群历史甚至使用 `helm rollback` 回滚版本。

## 6、Helm 命令

如果您想通过 Helm 命令查看更多的有用的信息，请使用 `helm -h` 命令，或者在任意命令后添加 -h 选项：

```sh
$ helm -h
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment variables:

| Name                               | Description                                                                       |
|------------------------------------|-----------------------------------------------------------------------------------|
| $HELM_CACHE_HOME                   | set an alternative location for storing cached files.                             |
| $HELM_CONFIG_HOME                  | set an alternative location for storing Helm configuration.                       |
| $HELM_DATA_HOME                    | set an alternative location for storing Helm data.                                |
| $HELM_DEBUG                        | indicate whether or not Helm is running in Debug mode                             |
| $HELM_DRIVER                       | set the backend storage driver. Values are: configmap, secret, memory, sql.       |
| $HELM_DRIVER_SQL_CONNECTION_STRING | set the connection string the SQL storage driver should use.                      |
| $HELM_MAX_HISTORY                  | set the maximum number of helm release history.                                   |
| $HELM_NAMESPACE                    | set the namespace used for the helm operations.                                   |
| $HELM_NO_PLUGINS                   | disable plugins. Set HELM_NO_PLUGINS=1 to disable plugins.                        |
| $HELM_PLUGINS                      | set the path to the plugins directory                                             |
| $HELM_REGISTRY_CONFIG              | set the path to the registry config file.                                         |
| $HELM_REPOSITORY_CACHE             | set the path to the repository cache directory                                    |
| $HELM_REPOSITORY_CONFIG            | set the path to the repositories file.                                            |
| $KUBECONFIG                        | set an alternative Kubernetes configuration file (default "~/.kube/config")       |
| $HELM_KUBEAPISERVER                | set the Kubernetes API Server Endpoint for authentication                         |
| $HELM_KUBECAFILE                   | set the Kubernetes certificate authority file.                                    |
| $HELM_KUBEASGROUPS                 | set the Groups to use for impersonation using a comma-separated list.             |
| $HELM_KUBEASUSER                   | set the Username to impersonate for the operation.                                |
| $HELM_KUBECONTEXT                  | set the name of the kubeconfig context.                                           |
| $HELM_KUBETOKEN                    | set the Bearer KubeToken used for authentication.                                 |

Helm stores cache, configuration, and data based on the following configuration order:

- If a HELM_*_HOME environment variable is set, it will be used
- Otherwise, on systems supporting the XDG base directory specification, the XDG variables will be used
- When no other location is set a default location will be used based on the operating system

By default, the default directories depend on the Operating System. The defaults are listed below:

| Operating System | Cache Path                | Configuration Path             | Data Path               |
|------------------|---------------------------|--------------------------------|-------------------------|
| Linux            | $HOME/.cache/helm         | $HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | $HOME/Library/Preferences/helm | $HOME/Library/helm      |
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
      --debug                       enable verbose output
  -h, --help                        help for helm
      --kube-apiserver string       the address and the port for the Kubernetes API server
      --kube-as-group stringArray   group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --kube-as-user string         username to impersonate for the operation
      --kube-ca-file string         the certificate authority file for the Kubernetes API server connection
      --kube-context string         name of the kubeconfig context to use
      --kube-token string           bearer token used for authentication
      --kubeconfig string           path to the kubeconfig file
  -n, --namespace string            namespace scope for this request
      --registry-config string      path to the registry config file (default "/Users/xcbeyond/Library/Preferences/helm/registry.json")
      --repository-cache string     path to the file containing cached repository indexes (default "/Users/xcbeyond/Library/Caches/helm/repository")
      --repository-config string    path to the file containing repository names and URLs (default "/Users/xcbeyond/Library/Preferences/helm/repositories.yaml")

Use "helm [command] --help" for more information about a command.
```
