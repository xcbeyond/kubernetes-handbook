# kube-apiserver 6443 端口占用

## 现象

kube-apiserver 不断重启，查看日志提示：6443 端口被占用。

```sh
failed to create listener: failed to listen on 0.0.0.0:6443: listen tcp 0.0.0.0:6443: bind: address already in use
```

## 解决方法

kill 掉现有 kube-apiserver 进程，重启 kubelet 服务即可。

```sh
# 查询 kube-apiserver 进程ID
$ ps -ef | grep 6443

# kill 掉
$ kill -9 <pid>

# 重启 kubelet
$ service kubelet restart
```

## 参考

1. [kubernetes: api-server and controller-manager cant start](https://stackoverflow.com/questions/48734524/kubernetes-api-server-and-controller-manager-cant-start)