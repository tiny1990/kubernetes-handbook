# kubernetes 从搭建到跑路

整理k8s-1.7.1 搭建过程

## Pre-Start
为了方便部署，做免密钥，非必需
### 修改hosts
10.163.34.114 k8s-1  
10.141.14.244 k8s-2  
10.141.63.163 k8s-3  

### 免密钥登陆
```
ssh-copy-id -i ~/.ssh/id_rsa
```

## QuickStart
### 基础服务搭建
1. 生成k8s需要的证书
2. 搭建etcd 高可用集群
3. docker + flannel
4. k8s-master
5. k8s-node

### 服务发现


###
