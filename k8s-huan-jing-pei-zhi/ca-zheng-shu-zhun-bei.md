# 配置ca及kubernetes证书

## 安装cfssl

### yum 下载安装

```bash
yum -y install cfssl
```

### wget 下载安装

```bash
#安装wget 
yum -y install wget
# 创建目录并下载文件
mkdir -p /opt/k8s/cfssl
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
# 移动修改文件名
mv cfssl_linux-amd64 /opt/k8s/cfssl/cfssl
mv cfssljson_linux-amd64 /opt/k8s/cfssl/cfssljson
mv cfssl-certinfo_linux-amd64 /opt/k8s/cfssl/cfssl-certinfo
# 设置可执行权限
chmod +x /opt/k8s/cfssl/*
# 添加到环境变量, 底部追加
vi /etc/profile
##############################################
export PATH=$PATH:/opt/k8s/cfssl
##############################################

# 使之生效
source /etc/profile
```

## 创建ca证书及私钥

### 创建配置文件

```bash
# 创建 CA 配置文件
mkdir -p /opt/k8s/ssl
cd /opt/k8s/ssl
​
cfssl print-defaults config > ca-config.json
cfssl print-defaults csr > ca-csr.json
```

### 创建ca证书

{% code-tabs %}
{% code-tabs-item title="cat ca-csr.json" %}
```bash
{
    "CN": "kubernetes",
    "key": {
        "algo": "ecdsa",
        "size": 256
    },
    "names": [
        {
            "C": "CN",
            "ST": "Shenzhen",
            "L": "Shenzhen",
            "O": "k8s",
            "OU": "System"
        }
    ]
}

# ca-config.json：可以定义多个 profiles，分别指定不同的过期时间、使用场景等参数；后续在签名证书时使用某个 profile；
# signing：表示该证书可用于签名其它证书；生成的 ca.pem 证书中 CA=TRUE；
# server auth：表示client可以用该 CA 对server提供的证书进行验证；
# client auth：表示server可以用该CA对client提供的证书进行验证；
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 创建CA签名请求

{% code-tabs %}
{% code-tabs-item title="ca-config.json" %}
```bash
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
            },
            "client": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "client auth"
                ]
            },
            "server": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth"
                ]
            }           
        }
    }
}

# CN：Common Name，kube-apiserver 从证书中提取该字段作为请求的用户名 (User Name)；浏览器使用该字段验证网站是否合法；
# O：Organization，kube-apiserver 从证书中提取该字段作为请求用户所属的组 (Group)；
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 创建etcd TLS证书与私钥

### 创建配置文件

```bash
cfssl print-defaults csr > etcd-csr.json
```

### 创建etcd证书签名请求

{% code-tabs %}
{% code-tabs-item title="etcd-csr.json" %}
```bash
{
    "CN": "etcd",
    "hosts": [
        "127.0.0.1",
        "192.168.0.201",
        "192.168.0.211",
        "192.168.0.212"
    ],
    "key": {
        "algo": "ecdsa",
        "size": 256
    },
    "names": [
        {
            "C": "CN",
            "ST": "ShenZhen",
            "L": "ShenZhen",
            "O": "k8s",
            "OU": "cloudteam"
        }
    ]
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### 生成etcd证书与私钥

```bash
cfssl gencert -ca=/etc/kubernetes/ssl/ca/ca.pem -ca-key=/etc/kubernetes/ssl/ca/ca-key.pem -config=/etc/kubernetes/ssl/ca/ca-config.json -profile=kubernetes etcd-csr.json | cfssljson -bare etcd
```



