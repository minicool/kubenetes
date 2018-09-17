# 配置k8s-master

## yum 准备

```bash
yum install -y kubernetes-master 
```

## 配置master

```bash
#写入集群环境信息
export ETCD_1_IP=192.168.0.201
export ETCD_1_NAME=k8s-etcd1
export ETCD_2_IP=192.168.0.211
export ETCD_2_NAME=k8s-etcd2
export ETCD_3_IP=192.168.0.212
export ETCD_3_NAME=k8s-etcd3
```

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
# KUBE_API_ADDRESS: 注意绑定的IP地址 不要 写成127.0.0.1, 要写成具体的IP地址(如果是阿里云的经典网络, 最好选择内网地址, 提升网络传输)
# KUBE_ETCD_SERVERS: 注意ETCD集群地址写法
# KUBE_SERVICE_ADDRESSES: 这个统一固定就好, 要和前面生成证书所指定的地址段一致
echo "update /etc/kubernetes/apiserver"
sed -i 's/--insecure-bind-address=127.0.0.1/--insecure-bind-address=192.168.0.201/g' /etc/kubernetes/apiserver
sed -i 's/--etcd-servers=http:\/\/127.0.0.1:2379/--etcd-servers=http:\/\/'$ETCD_1_IP',http://'$ETCD_2_IP',http://'$ETCD_3_IP'/g'  /etc/kubernetes/apiserver
sed -i 's/KUBE_API_ARGS=""/KUBE_API_ARGS="--authorization-mode=RBAC --runtime-config=rbac.authorization.k8s.io/v1beta1 --kubelet-https=true --experimental-bootstrap-token-auth --token-auth-file=\/etc\/kubernetes\/token.csv --service-node-port-range=30000-32767 --tls-cert-file=\/etc\/kubernetes\/ssl\/kubernetes\/kubernetes.pem --tls-private-key-file=\/etc\/kubernetes\/ssl\/kubernetes\/kubernetes-key.pem --client-ca-file=\/etc\/kubernetes\/ssl\/ca\/ca.pem --service-account-key-file=\/etc\/kubernetes\/ssl\/ca\/ca-key.pem --enable-swagger-ui=true --apiserver-count=3 --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=\/var\/lib\/audit.log --event-ttl=1h"/g'  /etc/kubernetes/apiserver

grep -v '^#' /etc/kubernetes/apiserver
# KUBE_API_ADDRESS="--insecure-bind-address=192.168.0.201"
# KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.0.201:2379,http://192.168.0.211:2379,http://192.168.0.201:2379"
# KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
# KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"
# KUBE_API_ARGS="--authorization-mode=RBAC --runtime-config=rbac.authorization.k8s.io/v1beta1 --kubelet-https=true --experimental-bootstrap-token-auth --token-auth-file=/etc/kubernetes/token.csv --service-node-port-range=30000-32767 --tls-cert-file=/etc/kubernetes/ssl/kubernetes.pem --tls-private-key-file=/etc/kubernetes/ssl/kubernetes-key.pem --client-ca-file=/etc/kubernetes/ssl/ca.pem --service-account-key-file=/etc/kubernetes/ssl/ca-key.pem --enable-swagger-ui=true --apiserver-count=3 --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/var/lib/audit.log --event-ttl=1h"

# 保存配置文件, 并启动kube-apiserver
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver
systemctl status kube-apiserver -l
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 配置kube-controller-manager

{% code-tabs %}
{% code-tabs-item title="/etc/kubernetes/controller-manager" %}
```bash
# --cluster-signing-* 指定的证书和私钥文件用来签名为 TLS BootStrap 创建的证书和私钥；
# --root-ca-file 用来对 kube-apiserver 证书进行校验，指定该参数后，才会在Pod 容器的 ServiceAccount 中放置该 CA 证书文件；
# --address 值必须为 127.0.0.1，因为当前 kube-apiserver 期望 scheduler 和 controller-manager 在同一台机器
# --leader-elect=true 参数就可以同时启动，controller-manager和scheduler 只要加上 ,系统会自动选举leader。而apiserver本来就可以多节点同时运行，只要它们连接同一个etcd cluster
echo "update /etc/kubernetes/controller-manager"
sed -i 's/KUBE_CONTROLLER_MANAGER_ARGS=""/KUBE_CONTROLLER_MANAGER_ARGS="--address=127.0.0.1 --service-cluster-ip-range=10.254.0.0\/16 --cluster-name=kubernetes --cluster-signing-cert-file=\/etc\/kubernetes\/ssl\/ca\/ca.pem --cluster-signing-key-file=\/etc\/kubernetes\/ssl\/ca\/ca-key.pem  --service-account-private-key-file=\/etc\/kubernetes\/ssl\/ca\/ca-key.pem --root-ca-file=\/etc\/kubernetes\/ssl\/ca\/ca.pem --leader-elect=true"/g' /etc/kubernetes/controller-manager 

grep -v '^#' /etc/kubernetes/controller-manager
# KUBE_CONTROLLER_MANAGER_ARGS="--address=127.0.0.1 --service-cluster-ip-range=10.254.0.0/16 --cluster-name=kubernetes --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem --root-ca-file=/etc/kubernetes/ssl/ca.pem --leader-elect=true"

# 保存配置并启动
systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
systemctl status kube-controller-manager -l
```
{% endcode-tabs-item %}

{% code-tabs-item title="" %}
```

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 配置kube-scheduler

{% code-tabs %}
{% code-tabs-item title=" /etc/kubernetes/scheduler" %}
```bash
# --leader-elect=true 参数就可以同时启动，controller-manager和scheduler 只要加上 ,系统会自动选举leader。而apiserver本来就可以多节点同时运行，只要它们连接同一个etcd cluster
# --address 值必须为 127.0.0.1，因为当前 kube-apiserver 期望 scheduler 和 controller-manager 在同一台机器；
echo "update /etc/kubernetes/scheduler"
sed -i 's/KUBE_SCHEDULER_ARGS=""/KUBE_SCHEDULER_ARGS="--leader-elect=true --address=127.0.0.1"/g' /etc/kubernetes/scheduler 

grep -v '^#' /etc/kubernetes/scheduler
# KUBE_SCHEDULER_ARGS="--leader-elect=true --address=127.0.0.1"

# 保存配置并启动
systemctl daemon-reload
systemctl enable kube-scheduler
systemctl start kube-scheduler
systemctl status kube-scheduler -l
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### 启动服务

```bash
for i in  kube-apiserver kube-controller-manager kube-scheduler;do systemctl restart $i; systemctl enable $i;done

# 检测状态
# cli-api
kubectl -s http://192.168.0.201:8080 get componentstatus
# NAME                 STATUS    MESSAGE              ERROR
# etcd-0               Healthy   {"health": "true"}
# scheduler            Healthy   ok
# controller-manager   Healthy   ok

# 检测端口
netstat -nltp | grep "kube"
# tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      5978/kubelet
# tcp        0      0 127.0.0.1:10251         0.0.0.0:*               LISTEN      6269/kube-scheduler
# tcp        0      0 127.0.0.1:10252         0.0.0.0:*               LISTEN      6188/kube-controlle
# tcp        0      0 127.0.0.1:43376         0.0.0.0:*               LISTEN      5978/kubelet
# tcp6       0      0 :::10250                :::*                    LISTEN      5978/kubelet
```

## 常见问题处理

### 1、 Failed to start Kubernetes API Server. {#1-failed-to-start-kubernetes-api-server}



