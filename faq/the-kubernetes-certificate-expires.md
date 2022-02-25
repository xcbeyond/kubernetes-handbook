# kubernetes证书过期问题

## 现象

通过 kubeadm 安装 kubernetes 集群时会存在一个证书问题，默认证书的有效期为一年。随着 kubernetes 集群的使用，某一天证书过期了，此时 kubernetes 集群将无法正常使用，比如：kubectl 命令执行会产生错误（`You must be logged in to the server(unauthorized)`）、通过 k8s 接口访问资源时出现“证书过期”的错误等。

很明显，这是证书过期导致，只需更新证书即可。

## 解决方法

1. 查看证书过期时间。

   在 Master 节点上，执行 `sudo openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text |grep ' Not '` 或 `kubeadm alpha certs check-expiration` 命令，查看证书过期时间。

2. 备份证书。

   为防止更新证书等操作失败，关键操作前一定要进行备份。备份 `/etc/kubernetes` 目录：

   ```sh
   cp -r /etc/kubernetes /etc/kubernetes.old  # 当升级证书失败时， 可以将此文件夹复原， 即可恢复原有集群
   ```

3. 更新证书。

   保存现有集群配置文件：

   ```sh
   kubeadm config view > kubeadm.yaml
   ```

   生成新的证书：

   ```sh
   kubeadm alpha certs renew all --config kubeadm.yaml
   ```

   生成新的配置文件:

   ```sh
   kubeadm init phase kubeconfig all --config kubeadm.yaml
   ```

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
