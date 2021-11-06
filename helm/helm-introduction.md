# Helm 介绍

Helm 是 Kubernetes 的包管理器，在 Kubernetes 下能够非常方便的完成应用的安装、卸载、升级等，是查看、分享和使用软件构建 Kubernetes 的最优方式，被广泛的使用。

![Helm](helm.svg)

## 1、背景

在 Kubernetes 环境中部署一个应用，需要 Kubernetes 原生资源 YAML 文件，如 Deployment、Service 或 Pod 等。对于一个应用而言，这些 Kubernetes 资源 YAML 文件都是十分分散的，不方便进行管理，通常直接通过 `kubectl` 命令来管理一个应用，你会发现这十分繁琐。

面对一个复杂的应用，会有很多类似的资源 YAML 文件，如果有更新或回滚应用的需求，可能要修改和维护所涉及的大量资源 YAML 文件，且由于缺少对发布过的应用版本管理和控制，使 Kubernetes 上的应用维护和更新等面临诸多的挑战，而 Helm 的出现可以帮助我们解决这些问题：

- 如何统一管理、配置和更新这些分散的 Kubernetes 的资源 YAML 文件
- 如何分发和复用一套应用模板
- 如何将应用的一系列资源 YAML 文件当做一个软件包管理

## 2、Helm 是什么？

Helm 是一个 Kubernetes 应用的包管理工具，用来管理 [chart](https://github.com/helm/charts) -- 预先配置好的安装包资源，有点类似于 Ubuntu 的 APT 和 CentOS 中的 YUM，使得对 Kubernetes 资源（Deployment、Service、ServiceAccount 等）的管理（存档、安装、卸载、升级等）变得更加简单、便捷。2019 年 11 月 13 日，Helm 3 发布，2020 年 4 月 30 日，从 CNCF 中[毕业](https://helm.sh/blog/celebrating-helms-cncf-graduation/)。

Helm chart 是用来将应用程序大量的 Kubernetes 资源 YAML 文件进行封装、模版化，在不同环境、场景下可根据配置快速部署你的应用，同时可将 chart 归档，并存储在 chart 仓库中，便于统一管理、分发。（将彻底释放繁琐、重复的 `kubectl apply -f` 动作，其中还伴随着对 YAML 文件中参数的调整。）

## 3、特点

Helm 有如下特点：

- **易管理**：通过定义的 charts，可以将大量复杂应用的 Kubernetes 资源配置进行封装、模版化。
- **可升级**：可就地升级，或自定义 hook 完成升级。
- **可分享**：chart 支持版本化，可将其共享，并存储在 chart 仓库中，供不同环境、项目使用。
- **可回滚**：使用 `helm rollback` 轻松回滚到旧版本。

## 4、Helm 架构

![Helm 架构图（来自 IBM Developer Blog）](helm-architecture.png)

通过 Helm Client 可以从本地或远端的 Chart Repository 安装。当 chart 安装到 Kubernetes 中后就会创建一个 release，每次更新该 chart 的配置并执行 `helm upgrade`， release 的版本数就会加 1。

### 4.1 组件

Helm 分为两部分，由两个不同的组件组成：

- **Helm Client**：提供终端用户的命令行客户端。负责以下功能：
  - 本地 chart 开发
  - 管理 Repositories
  - 管理 Releases
  - 对接 Helm library 仓库
    - 发送要安装的 charts
    - 请求升级或者卸载已存在的 Releases
- **Helm Library**：提供执行所有 Helm 操作的逻辑，（在 Helm 2 中，以 Helm Server(Tiller)体现 ）。与 Kubernetes API Server 交互并提供以下功能：
  - 结合 chart 和配置来构建一个 Release
  - 在 Kubernetes 中安装 charts，并提供后续的 Release 对象
  - 通过与 Kubernetes 交互来升级或者卸载 charts

独立的 Helm library 封装了 Helm 逻辑，以便可以由不同的客户端使用。

### 4.2 相关概念

- **Chart**：Helm 的软件包，采用 `tgz` 格式文件归档，类似 apt 的 deb 包或者 yum 的 rpm 包，chart 包含了一组定义 kubernetes 资源相关的 YAML 文件。
- **Release**：通过 `helm install` 命令安装后，生成在 kubernetes 集群中部署的 chart 就叫 Release。
- **Repository**：Helm 的存储仓库，Repository 本质上是一个 Web 服务器，该服务器保存了一系列的 Chart 包以供用户下载，并且提供了一个该 Repository 的 Chart 包的清单文件以供查询。Helm 支持同时管理多个不同的 Repository。

### 4.3 实现

Helm 是基于 Go 语言[实现](https://github.com/helm/helm)。

Helm 使用 Kubernetes [apimachinery](https://github.com/kubernetes/apimachinery) 和 [client-go](https://github.com/kubernetes/client-go) 连接 Kubernetes 和管理 Release，将 Release 信息以 Kubernetes 的 Secrets、ConfigMap，内存或 SQL 进行[存储](https://github.com/helm/helm/tree/main/pkg/storage)。

基于上述的实现，在实际项目中可进行集成或二次开发对接。

## 参考

- [Helm](https://docs.helm.sh/zh/)
- [使用 Helm 管理 kubernetes 应用](https://jimmysong.io/kubernetes-handbook/practice/helm.html)
- [Helm 介绍](https://bbs.huaweicloud.com/blogs/detail/280351)
