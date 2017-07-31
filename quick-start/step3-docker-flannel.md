# docker + flannel
在k8s-node搭建,根据场景需求

## 安装flannel
```
yum install -y flannel
```
可以用``` cat /usr/lib/systemd/system/flanneld.service | grep EnvironmentFile ``` 查看配置文件位置

修改``` /etc/sysconfig/flanneld ```
```
# Flanneld configuration options  

# etcd url location.  Point this to the server where etcd runs
ETCD_ENDPOINTS="https://k8s-1:2379,https://k8s-2:2379,https://k8s-3:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
ETCD_PREFIX="/kube-centos/network"

# Any additional options that you want to pass
FLANNEL_OPTIONS="-etcd-cafile=/etc/kubernetes/ssl/ca.pem -etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem -etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem"
```

## 在etcd中创建网络配置
```
etcdctl --endpoints=https://k8s-1:2379,https://k8s-2:2379,https://k8s-3:2379 \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  mkdir /kube-centos/network
etcdctl --endpoints=https://k8s-1:2379,https://k8s-2:2379,https://k8s-3:2379 \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  mk /kube-centos/network/config '{"Network":"172.30.0.0/16","SubnetLen":24,"Backend":{"Type":"vxlan"}}'
```

## 配置docker network

```ini
cat /usr/lib/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target

[Service]
Type=notify
EnvironmentFile=/run/flannel/subnet.env
ExecStart=/usr/bin/dockerd  --bip=${FLANNEL_SUBNET} --mtu=${FLANNEL_MTU}
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target

```
确保/run/flannel/subnet.env 存在

## 检查flannel docker0是否在一个网段
```
ifconfig
```


## 验证跨node之间网络通讯
两台node 运行 ``` docker run -it -d busybox ``` 跨机器ping
