# 搭建etcd cluster
## 下载etcd
```
wget https://github.com/coreos/etcd/releases/download/v3.2.4/etcd-v3.2.4-linux-arm64.tar.gz
tar -xvf etcd-v3.1.5-linux-amd64.tar.gz
mv etcd-v3.1.5-linux-amd64/etcd* /usr/local/bin
```

## 部署
### systemd
```
cat > /etc/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/local/bin/etcd \
  --name ${ETCD_NAME} \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  --peer-cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --peer-key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  --trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
  --initial-advertise-peer-urls ${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
  --listen-peer-urls ${ETCD_LISTEN_PEER_URLS} \
  --listen-client-urls ${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
  --advertise-client-urls ${ETCD_ADVERTISE_CLIENT_URLS} \
  --initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN} \
  --initial-cluster infra1=https://k8s-1:2380,infra2=https://k8s-2:2380,infra3=https://k8s-3:2380 \
  --initial-cluster-state new \
  --data-dir=${ETCD_DATA_DIR}
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

### etcd 配置
```
cat > /etc/etcd/etcd.conf << EOF
# [member]
ETCD_NAME=infra1
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="https://10.163.34.114:2380"
ETCD_LISTEN_CLIENT_URLS="https://10.163.34.114:2379"

#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://k8s-1:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://k8s-1:2379"
EOF
```

### 分发到其他机器
```
scp /usr/local/bin/etcd* k8s-n:/usr/local/bin/
scp /etc/etcd/etcd.conf  k8s-n:/etc/etcd/etcd.conf
```

### 修改分发的配置
修改  /etc/etcd/etcd.conf 中的 ``` ETCD_NAME ``` ``` ETCD_LISTEN_PEER_URLS ```  ``` ETCD_LISTEN_CLIENT_URLS ```



## 启动
```
ssh k8s-n systemctl daemon-reload
ssh k8s-n systemctl daemon-reload
ssh k8s-n systemctl start etcd
```

## 验证
```
etcdctl \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  cluster-health
```
```
member 5b464a2f75a38e65 is healthy: got healthy result from https://k8s-3:2379
member 69e952a5a6a6e798 is healthy: got healthy result from https://k8s-1:2379
member aee4be324f12ea53 is healthy: got healthy result from https://k8s-2:2379
```

## trouble-shouting
vim /var/log/messages 查看错误提示
