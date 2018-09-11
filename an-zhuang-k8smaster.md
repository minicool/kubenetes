# 安装k8s-master

## yum 准备

```bash
yum install -y etcd kubernetes-master 
```

## etcd 安装

{% code-tabs %}
{% code-tabs-item title="/etc/etcd/etcd.conf" %}
```bash
#配置etcd服务
echo "update /etc/etcd/etcd.conf"
sed -i 's/localhost:2379/&\,http:\/\/192.168.0.201:2379/' /etc/etcd/etcd.conf
systemctl start etcd;systemctl enable etcd
# grep -v '^#' /etc/etcd/etcd.conf
# ETCD_NAME=default
# ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
# ETCD_LISTEN_CLIENT_URLS="http://localhost:2379,http://192.168.0.201:2379"
# ETCD_ADVERTISE_CLIENT_URLS="http://192.168.0.201:2379"
#测试etcd服务
etcdctl cluster-health
#正常
# member 8e9e05c52164694d is healthy: got healthy result from http://192.168.0.201:2379
# cluster is healthy
#查询etcd列表
etcdctl member list
#8e9e05c52164694d: name=default peerURLs=http://localhost:2380 clientURLs=http://192.168.0.201:2379,http://localhost:2379 isLeader=true
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 配置master

#### 配置kube-conf

{% code-tabs %}
{% code-tabs-item title="/etc/kubernetes/config" %}
```bash
# KUBE_MASTER：指定apiserver的url地址。我的master服务器ip为192.168.0.201
echo "update /etc/kubernetes/config"
sed -i 's/127.0.0.1/192.168.0.201/g' /etc/kubernetes/config

grep -v '^#' /etc/kubernetes/config
# KUBE_LOGTOSTDERR="--logtostderr=true"
# KUBE_LOG_LEVEL="--v=0"
# KUBE_ALLOW_PRIV="--allow-privileged=false"
# KUBE_MASTER="--master=http://192.168.0.201:8080"

```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### 配置kube-apiserver

{% code-tabs %}
{% code-tabs-item title="/etc/kubernetes/apiserver" %}
```bash
# 修改-insecure-bind-address（apiserver绑定主机的非安全IP地址，改为masterip表示绑定master地址）


grep -v '^#' /etc/kubernetes/apiserver
# KUBE_API_ADDRESS="--insecure-bind-address=127.0.0.1"
# KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"
# KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
# KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
# KUBE_API_ARGS=""

```
{% endcode-tabs-item %}
{% endcode-tabs %}

