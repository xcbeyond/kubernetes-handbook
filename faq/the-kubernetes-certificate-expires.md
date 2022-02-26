# kubernetes证书过期问题

## 现象

通过 kubeadm 安装 kubernetes 集群时会存在一个证书问题：由 kubeadm 生成的客户端证书在 1 年后到期。

随着 kubernetes 集群的使用，某一天证书过期了，此时 kubernetes 集群将无法正常使用，比如：kubectl 命令执行会产生错误（`You must be logged in to the server(unauthorized)`）、通过 k8s 接口访问资源时出现“证书过期”的错误等。

很明显，这是证书过期导致，只需更新证书即可。

> k8s 为什么将客户端证书的有效期设置为 1 年呢？
>
> **kubeadm 会在控制面升级的时候更新所有证书。**
>
> 如果你对此类证书的更新没有特殊要求， 并且定期执行 Kubernetes 版本升级（每次升级之间的间隔时间少于 1 年）， 则 kubeadm 将确保你的集群保持最新状态并保持合理的安全性。
>
> **最佳的做法是经常升级集群以确保安全。**

## 解决方法

在实际中，如果遇到证书过期，则需手动更新证书，通过 `kubeadm certs renew` 命令手动更新你的证书。

1. 查看证书过期时间。

   在 Master 节点上，执行 `kubeadm certs check-expiration` 命令，查看证书过期时间。

   输出类似以下内容：

   ```sh
   CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
   admin.conf                 Dec 30, 2020 23:36 UTC   364d                                    no
   apiserver                  Dec 30, 2020 23:36 UTC   364d            ca                      no
   apiserver-etcd-client      Dec 30, 2020 23:36 UTC   364d            etcd-ca                 no
   apiserver-kubelet-client   Dec 30, 2020 23:36 UTC   364d            ca                      no
   controller-manager.conf    Dec 30, 2020 23:36 UTC   364d                                    no
   etcd-healthcheck-client    Dec 30, 2020 23:36 UTC   364d            etcd-ca                 no
   etcd-peer                  Dec 30, 2020 23:36 UTC   364d            etcd-ca                 no
   etcd-server                Dec 30, 2020 23:36 UTC   364d            etcd-ca                 no
   front-proxy-client         Dec 30, 2020 23:36 UTC   364d            front-proxy-ca          no
   scheduler.conf             Dec 30, 2020 23:36 UTC   364d                                    no

   CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
   ca                      Dec 28, 2029 23:36 UTC   9y              no
   etcd-ca                 Dec 28, 2029 23:36 UTC   9y              no
   front-proxy-ca          Dec 28, 2029 23:36 UTC   9y              no
   ```

2. 备份证书。

   为防止更新证书等操作失败，关键操作前一定要进行备份。备份 `/etc/kubernetes` 目录：

   ```sh
   cp -r /etc/kubernetes /etc/kubernetes.old  # 当升级证书失败时， 可以将此文件夹复原， 即可恢复原有集群
   ```

3. 更新证书。

   使用 `kubeadm certs renew` 命令来更新证书。

4. 更新 ~/.kube/config 文件。

   ```sh
   mv config config.old
   cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   chown $(id -u):$(id -g) $HOME/.kube/config
   sudo chmod 644 $HOME/.kube/config
   ```

5. 重启。

   重启 kube-apiserver,kube-controller,kube-scheduler,etcd 这4个容器：

   `docker ps | grep -v pause | grep -E "etcd|scheduler|controller|apiserver" | awk '{print $1}' | awk '{print "docker","restart",$1}' | bash`

---

参考：

[使用 kubeadm 进行证书管理](https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)