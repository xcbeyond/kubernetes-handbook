# Helm Charts

Helm 是一个 Kubernetes 应用的包管理工具，常以 `helm` 命令来执行使用。而 Charts 则是预先配置好的安装包资源,常被称之为“chart”,可通过 `helm create <chart-name>` 命令创建一个空的 chart 文件结构。

本节将针对 chart 展开说明，讲述 chart 的格式，为使用 Helm 构建 chart 提供指导。

## 准备工作

chart 本质上是一系列的 Kubernetes 资源 YAML 文件（例如，Deployment、Service等），所以在正式编写chart之前，建议事先准备好符合


## 构建 chart
