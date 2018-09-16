# 配置k8s-master

## yum 准备

```bash
yum install -y etcd kubernetes-master 
```

## 配置master

### 配置kube-conf

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

### 配置kube-apiserver

{% code-tabs %}
{% code-tabs-item title="/etc/kubernetes/apiserver" %}
```bash
# 修改-insecure-bind-address（apiserver绑定主机的非安全IP地址，改为masterip表示绑定master地址）
echo "update /etc/kubernetes/apiserver"
sed -i 's/"--insecure-bind-address=127.0.0.1"/"--insecure-bind-address=192.168.0.201"/g' /etc/kubernetes/apiserver 

grep -v '^#' /etc/kubernetes/apiserver
# KUBE_API_ADDRESS="--insecure-bind-address=192.168.0.201"
# KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"
# KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
# KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
# KUBE_API_ARGS=""

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 配置kube-controller-manager

{% code-tabs %}
{% code-tabs-item title="kube-controller-manager" %}
```bash
grep -v '^#' /etc/kubernetes/controller-manager
# KUBE_CONTROLLER_MANAGER_ARGS=""
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 配置kube-scheduler

{% code-tabs %}
{% code-tabs-item title=" /etc/kubernetes/scheduler" %}
```bash
echo "update /etc/kubernetes/scheduler"
sed -i 's/KUBE_SCHEDULER_ARGS=""/KUBE_SCHEDULER_ARGS="0.0.0.0"/g' /etc/kubernetes/scheduler 


grep -v '^#' /etc/kubernetes/scheduler
# KUBE_SCHEDULER_ARGS="--address=0.0.0.0"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 启动服务

```bash
for i in  kube-apiserver kube-controller-manager kube-scheduler;do systemctl restart $i; systemctl enable $i;done

# 检测状态
kubectl -s http://192.168.0.201:8080 get componentstatus
# NAME                 STATUS    MESSAGE              ERROR
# etcd-0               Healthy   {"health": "true"}
# scheduler            Healthy   ok
# controller-manager   Healthy   ok
```

