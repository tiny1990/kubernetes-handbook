# 创建TLS证书和秘钥
非重点内容，粗略笔记

## 安装CFSSL ( COPY && paste )
```
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
chmod +x cfssl_linux-amd64
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl

wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
chmod +x cfssljson_linux-amd64
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl-certinfo_linux-amd64
sudo mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
```

## 创建 CA (Certificate Authority)
### 创建 CA 配置文件
```
cat > ca-config.json << EOF
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "kubernetes": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOF
```

### 创建 CA 证书签名请求
```
cat > ca-csr.json << EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System",
            "ST": "BeiJing"
        }
    ]
}
EOF
```
### 生成 CA 证书和私钥
```
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
ls ca*
```
ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem

## 创建 kubernetes 证书

### 创建 kubernetes 证书签名请求
```
cat > kubernetes-csr.json << EOF
{
    "CN": "kubernetes",
    "hosts": [
        "127.0.0.1",
        "k8s-1",
        "k8s-2",
        "k8s-3",
        "k8s-1 的IP",
        "k8s-2 的IP",
        "k8s-3 的IP",
        "10.254.0.1",
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System",
            "ST": "BeiJing"
        }
    ]
}
EOF
```
### 生成 kubernetes 证书和私钥
```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
ls kubernetes*
```
kubernetes.csr  kubernetes-csr.json  kubernetes-key.pem  kubernetes.pem
## 创建 admin 证书
```
cat > admin-csr.json << EOF
{
    "CN": "admin",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "O": "system:masters",
            "OU": "System",
            "ST": "BeiJing"
        }
    ]
}
EOF
```
### 生成 admin 证书和私钥
```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
ls admin*
```
admin.csr  admin-csr.json  admin-key.pem  admin.pem

## 创建 kube-proxy 证书
### 创建 kube-proxy 证书签名请求
```
cat > kube-proxy-csr.json <<EOF
{
    "CN": "system:kube-proxy",
    "hosts": [],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "System",
            "ST": "BeiJing"
        }
    ]
}
EOF
```

### 生成 kube-proxy 客户端证书和私钥
```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes  kube-proxy-csr.json | cfssljson -bare kube-proxy
ls kube-proxy*
```
kube-proxy.csr  kube-proxy-csr.json  kube-proxy-key.pem  kube-proxy.pem

## 分发证书
```
mkdir -p /etc/kubernetes/ssl
cp *.pem /etc/kubernetes/ssl
ssh k8s-1 mkdir -p /etc/kubernetes/ssl
ssh k8s-2 mkdir -p /etc/kubernetes/ssl
ssh k8s-3 mkdir -p /etc/kubernetes/ssl
scp *.pem k8s-1:/etc/kubernetes/ssl
scp *.pem k8s-2:/etc/kubernetes/ssl
scp *.pem k8s-3:/etc/kubernetes/ssl
```
