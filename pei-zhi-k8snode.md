# 配置 k8s-node

## yum 准备

```bash
yum install -y kubernetes-node docker
```

## node配置

### k8s-node-1

#### kube-config

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

#### kubelet配置 {#修改kubelet配置文件}

{% code-tabs %}
{% code-tabs-item title="/etc/kubernetes/kubelet" %}
```bash
#KUBELET 指定
echo "update /etc/kubernetes/kubelet"
#设置为从本地访问为全网访问
sed -i 's/--address=127.0.0.1/--address=0.0.0.0/g' /etc/kubernetes/kubelet
#设置hostname
sed -i 's/--hostname-override=127.0.0.1/--hostname-override='$ETCD_HOST_NAME'/g' /etc/kubernetes/kubelet
#设置api-server
sed -i 's/-api-servers=http:\/\/127.0.0.1:8080/-api-servers=http:\/\/'$ETCD_1_IP':8080/g' /etc/kubernetes/kubelet
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### 启动配置

```bash
systemctl daemon-reload 
systemctl restart docker kubelet kube-proxy
systemctl enable docker kubelet kube-proxy
```

