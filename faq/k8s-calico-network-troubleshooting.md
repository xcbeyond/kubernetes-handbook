# k8s calico 网络排错

问题描述：

有一个 master 节点 k8s02 执行 kubectl 命令偶尔通过，基本上都是 timeout，kube-api log 显示 访问 metric timeout，ping metrics-server pod ip失败。

然后发现分布到该节点 pod dns 解析有问题，抓包发现未收到 coredns 的回应，怀疑网络问题。用 k8s02 上的 pod centos ping coredns pod ip，发现失败。

k8s01 ip: 10.2.7.200

k8s02 ip: 10.2.7.201

1. 怀疑 calico 路由问题，查看 k8s02 上的 路由，发现缺少到其它节点 pod 的路由

   ```shell
   [root@k8s02 ~]# route -n
   Kernel IP routing table
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   0.0.0.0         10.2.7.254      0.0.0.0         UG    100    0        0 eth0
   10.2.7.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
   10.253.65.64    0.0.0.0         255.255.255.192 U     0      0        0 *
   10.253.65.67    0.0.0.0         255.255.255.255 UH    0      0        0 calic6105d1b464
   172.31.255.0    0.0.0.0         255.255.255.0   U     0      0        0 docker0

   ```

2. 在 k8s02 上执行 calicoctl node status，发现 calico node 连接有问题

   ```shell
   [root@k8s02 ~]# calicoctl node status
   Calico process is running.

   IPv4 BGP status
   +--------------+-------------------+-------+----------+--------------------------------+
   | PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |              INFO              |
   +--------------+-------------------+-------+----------+--------------------------------+
   | 10.2.7.200   | node-to-node mesh | start | 02:04:22 | Idle Socket: Connection closed |
   | 10.2.7.202   | node-to-node mesh | start | 02:04:22 | Active Socket: Connection      |
   |              |                   |       |          | closed                         |
   | 10.2.7.203   | node-to-node mesh | start | 02:04:22 | Active Socket: Connection      |
   |              |                   |       |          | closed                         |
   | 10.2.7.204   | node-to-node mesh | start | 02:04:22 | Active Socket: Connection      |
   |              |                   |       |          | closed                         |
   | 10.2.7.205   | node-to-node mesh | start | 02:04:22 | Active Socket: Connection      |
   |              |                   |       |          | closed                         |
   | 10.2.7.206   | node-to-node mesh | start | 02:04:22 | Active Socket: Connection      |
   |              |                   |       |          | closed                         |
   | 10.2.7.207   | node-to-node mesh | start | 02:04:22 | Active Socket: Connection      |
   |              |                   |       |          | closed                         |
   | 10.2.7.208   | node-to-node mesh | start | 02:04:22 | Active Socket: Connection      |
   |              |                   |       |          | closed                         |
   | 10.2.7.209   | node-to-node mesh | start | 02:04:22 | Active Socket: Connection      |
   |              |                   |       |          | closed                         |
   +--------------+-------------------+-------+----------+--------------------------------+

   IPv6 BGP status
   No IPv6 peers found.

   ```

3. 在 k8s01 上执行 calicoctl node status，发现没有 k8s02 10.2.7.201，但是多了一个 192.188.1.1 的 ip，这个 ip 在 k8s02 上出现过，我直接用 ip link delete 删掉了那块网卡（不清楚这块网卡是什么时候加上去的，只知道现在没有地方用到这个网卡，所以直接删掉了）

   ```shell
   [root@k8s01 ~]# calicoctl node status
   Calico process is running.

   IPv4 BGP status
   +--------------+-------------------+-------+------------+-------------+
   | PEER ADDRESS |     PEER TYPE     | STATE |   SINCE    |    INFO     |
   +--------------+-------------------+-------+------------+-------------+
   | 192.188.1.1  | node-to-node mesh | start | 2020-04-08 | Connect     |
   | 10.2.7.202   | node-to-node mesh | up    | 2020-04-08 | Established |
   | 10.2.7.203   | node-to-node mesh | up    | 2020-04-08 | Established |
   | 10.2.7.204   | node-to-node mesh | up    | 2020-04-08 | Established |
   | 10.2.7.205   | node-to-node mesh | up    | 2020-04-08 | Established |
   | 10.2.7.206   | node-to-node mesh | up    | 2020-04-08 | Established |
   | 10.2.7.207   | node-to-node mesh | up    | 2020-04-08 | Established |
   | 10.2.7.208   | node-to-node mesh | up    | 2020-04-08 | Established |
   | 10.2.7.209   | node-to-node mesh | up    | 2020-04-08 | Established |
   +--------------+-------------------+-------+------------+-------------+

   IPv6 BGP status
   No IPv6 peers found.

   ```

4. 第 2 3 两步，初步可以判断是因为 calico 问题导致 k8s02 节点上的 node 路由出问题，所以在 k8s10 上重新启动所有的 calico pod 和 calico-kube-controllers pod

   ```shell
   [root@k8s01 ~]# kubectl get po -n kube-system | grep calico | awk '{print $1}' |xargs -I {} kubectl delete po {} -n kube-system
   ```

5. 检查 k8s02 上的 路由，发现恢复正常，ping k8s02 上面的 pod 也正常

   ```shell
   [root@k8s02 logs]# route -n
   Kernel IP routing table
   Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
   0.0.0.0         10.2.7.254      0.0.0.0         UG    100    0        0 eth0
   10.2.7.0        0.0.0.0         255.255.255.0   U     100    0        0 eth0
   10.253.13.192   10.2.7.207      255.255.255.192 UG    0      0        0 tunl0
   10.253.19.192   10.2.7.206      255.255.255.192 UG    0      0        0 tunl0
   10.253.65.64    0.0.0.0         255.255.255.192 U     0      0        0 *
   10.253.65.67    0.0.0.0         255.255.255.255 UH    0      0        0 calic6105d1b464
   10.253.71.192   10.2.7.209      255.255.255.192 UG    0      0        0 tunl0
   10.253.120.64   10.2.7.203      255.255.255.192 UG    0      0        0 tunl0
   10.253.123.0    10.2.7.204      255.255.255.192 UG    0      0        0 tunl0
   10.253.126.64   10.2.7.208      255.255.255.192 UG    0      0        0 tunl0
   10.253.140.128  10.2.7.202      255.255.255.192 UG    0      0        0 tunl0
   10.253.178.64   10.2.7.205      255.255.255.192 UG    0      0        0 tunl0
   10.253.181.0    10.2.7.200      255.255.255.192 UG    0      0        0 tunl0
   172.31.255.0    0.0.0.0         255.255.255.0   U     0      0        0 docker0

   ```

   ```shell
   [root@k8s02 logs]# calicoctl node status
   Calico process is running.

   IPv4 BGP status
   +--------------+-------------------+-------+----------+-------------+
   | PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |
   +--------------+-------------------+-------+----------+-------------+
   | 10.2.7.200   | node-to-node mesh | up    | 06:45:53 | Established |
   | 10.2.7.202   | node-to-node mesh | up    | 06:45:54 | Established |
   | 10.2.7.203   | node-to-node mesh | up    | 06:45:53 | Established |
   | 10.2.7.204   | node-to-node mesh | up    | 06:45:53 | Established |
   | 10.2.7.205   | node-to-node mesh | up    | 06:45:55 | Established |
   | 10.2.7.206   | node-to-node mesh | up    | 06:45:53 | Established |
   | 10.2.7.207   | node-to-node mesh | up    | 06:45:54 | Established |
   | 10.2.7.208   | node-to-node mesh | up    | 06:45:53 | Established |
   | 10.2.7.209   | node-to-node mesh | up    | 06:45:53 | Established |
   +--------------+-------------------+-------+----------+-------------+

   IPv6 BGP status
   No IPv6 peers found.

   ```

转载：[记一次 k8s calico 网络排错](https://blog.csdn.net/Man_In_The_Night/article/details/105557093)