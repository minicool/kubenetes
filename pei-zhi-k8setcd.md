# 配置k8s-etcd

## 配置单etcd

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
#刷新配置文件
systemctl daemon-reload
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

## 配置多etcd

{% code-tabs %}
{% code-tabs-item title="/etc/etcd/etcd.conf" %}
```bash
#example 
k8s-etcd1 192.168.0.201
k8s-etcd2 192.168.0.211

export HOST_IP=192.168.0.211

sed -i 's/default.etcd/k8s-etcd1.etcd/' /etc/etcd/etcd.conf
sed -i 's/^ETCD_NAME=\"default\"/ETCD_NAME="k8s-etcd1"/' /etc/etcd/etcd.conf
sed -i 's/^#ETCD_LISTEN_PEER_URLS="http:\/\/localhost:2380"/ETCD_LISTEN_PEER_URLS="http:\/\/'$HOST_IP's:2380"/' /etc/etcd/etcd.conf

sed -i 's/^#ETCD_INITIAL_ADVERTISE_PEER_URLS/ETCD_INITIAL_ADVERTISE_PEER_URLS/' /etc/etcd/etcd.conf
sed -i 's/^#ETCD_INITIAL_CLUSTER/ETCD_INITIAL_CLUSTER/' /etc/etcd/etcd.conf
sed -i 's/^#ETCD_INITIAL_CLUSTER_STATE/ETCD_INITIAL_CLUSTER_STATE/' /etc/etcd/etcd.conf
sed -i 's/^#ETCD_INITIAL_CLUSTER_TOKEN/ETCD_INITIAL_CLUSTER_TOKEN/' /etc/etcd/etcd.conf


# k8s-sz-etcd1 192.168.0.201
vi /etc/etcd/etcd.conf
# [member]
ETCD_NAME=k8s-sz-etcd1
ETCD_DATA_DIR="/var/lib/etcd/k8s-sz-etcd1.etcd"
#监听URL，用于与其他节点通讯
ETCD_LISTEN_PEER_URLS="http://192.168.1.50:2380"
#告知客户端的URL, 也就是服务的URL
ETCD_LISTEN_CLIENT_URLS="http://192.168.1.50:2379,http://127.0.0.1:2379"

# [cluster]
#表示监听其他节点同步信号的地址
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.50:2380"
ETCD_INITIAL_CLUSTER="k8s-sz-etcd1=http://192.168.1.50:2380,k8s-sz-etcd2=http://192.168.1.51:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#–advertise-client-urls 告知客户端的URL, 也就是服务的URL，tcp2379端口用于监听客户端请求
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.50:2379"
```
{% endcode-tabs-item %}
{% endcode-tabs %}

