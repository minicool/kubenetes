# 脚本安装

### Master端的安装与配置 {#1master端的安装与配置}

{% code-tabs %}
{% code-tabs-item title="init-k8s-master.sh" %}
```bash
#!/bin/bash
#关闭防火墙，并关闭防火墙的开机自启动安全的做法是在防火墙上配置各组件需要相互通信的端口号，这里选择直接禁用防火墙。
systemctl stop firewalld
systemctl disable firewalld
#禁用selinux，修改/etc/selinux/config
#SELINUX=enforcing改为SELINUX=disabled
echo "update /etc/selinux/config"
sed -i 's/enforcing/disabled/g' /etc/selinux/config
#安装etcd和kubernetes-master
#etcd为kubernetes集群的主数据库，配置文件通常不需要特别的参数配置，默认将监听127.0.0.1:2379地址供客户端链接使用，shell脚本执行后，可以通过etcdctl cluster-health验证etcd是否正确启动
yum -y install etcd kubernetes-master
#修改apiserver的配置文件/etc/kubernetes/apiserver
#修改-insecure-bind-address（apiserver绑定主机的非安全IP地址，改为masterip表示绑定master地址）
echo "update /etc/kubernetes/apiserver"
sed -i 's!KUBE_API_ADDRESS="--insecure-bind-address=127.0.0.1"!KUBE_API_ADDRESS="--insecure-bind-address=192.168.121.143"!' /etc/kubernetes/apiserver 
#修改kubernetes的配置文件/etc/kubernetes/config
#KUBE_MASTER：指定apiserver的url地址。我的master服务器ip为192.168.121.143
echo "update /etc/kubernetes/config"
sed -i 's!127.0.0.1!192.168.121.143!' /etc/kubernetes/config
systemctl daemon-reload
#让etcd kube-apiserver kube-scheduler kube-controller-manager随开机启动
systemctl enable etcd kube-apiserver kube-scheduler kube-controller-manager
#启动或重启etcd kube-apiserver kube-scheduler kube-controller-manager
systemctl restart etcd kube-apiserver kube-scheduler kube-controller-manager

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Node端的安装与配置 {#2node端的安装与配置}

{% code-tabs %}
{% code-tabs-item title="init-k8s-node.sh" %}
```bash
#!/bin/bash
#关闭防火墙，并关闭防火墙的开机自启动
#安全的做法是在防火墙上配置各组件需要相互通信的端口号，这里选择直接禁用防火墙。
systemctl stop firewalld
systemctl disable firewalld
#禁用selinux，修改/etc/selinux/config
#SELINUX=enforcing改为SELINUX=disabled
echo "update /etc/selinux/config"
sed -i 's/enforcing/disabled/g' /etc/selinux/config
#安装kubernetes-node（会自动安装docker）
yum -y install kubernetes-node
#修改kubernetes的配置文件/etc/kubernetes/config
#KUBE_MASTER：指定apiserver的url地址。我的master服务器ip为192.168.121.143
echo "update /etc/kubernetes/config"
sed -i 's!127.0.0.1!192.168.121.143!' /etc/kubernetes/config
#改成网易蜂巢云的镜像源,加速下载docker镜像
echo "update /etc/docker/daemon.json"
sed -i 's!{}!{"registry-mirrors":["http://hub-mirror.c.163.com"],"insecure-registries":["192.168.121.140:5000"]}!' /etc/docker/daemon.json
#修改kubelet的配置文件/etc/kubernetes/kubelet
#修改KUBELET_HOSTNAME，在此我改成了node ip
#修改KUBELET_API_SERVER，我的Master地址为192.168.121.143
echo "update /etc/kubernetes/kubelet"
sed -i 's!KUBELET_HOSTNAME="--hostname-override=127.0.0.1"!KUBELET_HOSTNAME="--hostname-override=192.168.121.144"!' /etc/kubernetes/kubelet
sed -i 's!KUBELET_API_SERVER="--api-servers=http://127.0.0.1:8080"!KUBELET_API_SERVER="--api-servers=http://192.168.121.143:8080"!' /etc/kubernetes/kubelet
sed -i 's!KUBELET_ARGS=""!KUBELET_ARGS="--cluster_dns=10.254.10.2 --cluster_domain=cluster.local"!' /etc/kubernetes/kubelet
sed -i 's!KUBELET_ADDRESS="--address=127.0.0.1"!KUBELET_ADDRESS="--address=192.168.121.144"!' /etc/kubernetes/kubelet
systemctl daemon-reload 
systemctl restart docker kubelet kube-proxy
systemctl enable docker kubelet kube-proxy


```
{% endcode-tabs-item %}
{% endcode-tabs %}

