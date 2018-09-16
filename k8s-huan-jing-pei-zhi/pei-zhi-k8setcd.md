# 配置k8s-etcd

## 防火墙开启

```bash
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 2379 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 2380 -j ACCEPT
```

## \(static\)静态服务配置启动

### 配置单etcd

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

### 配置多etcd

{% code-tabs %}
{% code-tabs-item title="/etc/etcd/etcd.conf" %}
```bash
# k8s-sz-etcd1 192.168.0.50
# k8s-sz-etcd2 192;168.0.51
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
ETCD_INITIAL_CLUSTER="k8s-sz-etcd1=http://192.168.1.50:2379,k8s-sz-etcd2=http://192.168.1.51:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
#–advertise-client-urls 告知客户端的URL, 也就是服务的URL，tcp2379端口用于监听客户端请求
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.50:2379"

# 编辑启动配置项
vi /usr/lib/systemd/system/etcd.service
##############################################
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd \
    --name ${ETCD_NAME} \
    --data-dir ${ETCD_DATA_DIR} \
    --listen-client-urls ${ETCD_LISTEN_CLIENT_URLS} \
    --listen-peer-urls ${ETCD_LISTEN_PEER_URLS} \
    --advertise-client-urls ${ETCD_ADVERTISE_CLIENT_URLS} \
    --initial-advertise-peer-urls ${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
    --initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN} \
    --initial-cluster-state ${ETCD_INITIAL_CLUSTER_STATE} \
    --initial-cluster ${ETCD_INITIAL_CLUSTER} "  
##############################################
```
{% endcode-tabs-item %}
{% endcode-tabs %}

#### 配置etcd-1

```bash
export ETCD_1_IP=192.168.0.201
export ETCD_1_NAME=k8s-etcd1
export ETCD_2_IP=192.168.0.211
export ETCD_2_NAME=k8s-etcd2
export ETCD_3_IP=192.168.0.212
export ETCD_3_NAME=k8s-etcd3
export ETCD_HOST_IP=$ETCD_1_IP
export ETCD_HOST_NAME=$ETCD_1_NAME

export ETCD_LISTEN_PEER_URLS=http://$ETCD_HOST_IP:2380
export ETCD_LISTEN_CLIENT_URLS=http://localhost:2379,http://$ETCD_HOST_IP:2379
export ETCD_ADVERTISE_CLIENT_URLS=http://localhost:2379,http://$ETCD_HOST_IP:2379
export ETCD_INITIAL_ADVERTISE_PEER_URLS=http://$ETCD_HOST_IP:2380
export ETCD_INITIAL_CLUSTER=$ETCD_1_NAME=http://$ETCD_1_IP:2380,$ETCD_2_NAME=http://$ETCD_2_IP:2380,$ETCD_3_NAME=http://$ETCD_3_IP:2380
export ETCD_INITIAL_CLUSTER_STATE=new
export ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
#部分环境变量带有/，导致sed出错，换成！作为表示
sed -i 's/default.etcd/'$ETCD_HOST_NAME'.etcd/' /etc/etcd/etcd.conf
sed -i 's/^ETCD_NAME=\"default\"/ETCD_NAME="'$ETCD_HOST_NAME'"/' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_LISTEN_PEER_URLS="http:\/\/localhost:2380"!ETCD_LISTEN_PEER_URLS="'$ETCD_LISTEN_PEER_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^ETCD_LISTEN_CLIENT_URLS="http:\/\/localhost:2379"!ETCD_LISTEN_CLIENT_URLS="'$ETCD_LISTEN_CLIENT_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^ETCD_ADVERTISE_CLIENT_URLS="http:\/\/localhost:2379"!ETCD_ADVERTISE_CLIENT_URLS="'$ETCD_ADVERTISE_CLIENT_URLS'"!' /etc/etcd/etcd.conf

sed -i 's!^#ETCD_INITIAL_ADVERTISE_PEER_URLS="http:\/\/localhost:2380"!ETCD_INITIAL_ADVERTISE_PEER_URLS="'$ETCD_INITIAL_ADVERTISE_PEER_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER="default=http:\/\/localhost:2380"!ETCD_INITIAL_CLUSTER="'$ETCD_INITIAL_CLUSTER'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER_STATE="new"!ETCD_INITIAL_CLUSTER_STATE="'$ETCD_INITIAL_CLUSTER_STATE'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"!ETCD_INITIAL_CLUSTER_TOKEN="'$ETCD_INITIAL_CLUSTER_TOKEN'"!' /etc/etcd/etcd.conf
```

#### 配置etcd-2

```bash
export ETCD_1_IP=192.168.0.201
export ETCD_1_NAME=k8s-etcd1
export ETCD_2_IP=192.168.0.211
export ETCD_2_NAME=k8s-etcd2
export ETCD_3_IP=192.168.0.212
export ETCD_3_NAME=k8s-etcd3
export ETCD_HOST_IP=$ETCD_2_IP
export ETCD_HOST_NAME=$ETCD_2_NAME

export ETCD_LISTEN_PEER_URLS=http://$ETCD_HOST_IP:2380
export ETCD_LISTEN_CLIENT_URLS=http://localhost:2379,http://$ETCD_HOST_IP:2379
export ETCD_ADVERTISE_CLIENT_URLS=http://localhost:2379,http://$ETCD_HOST_IP:2379
export ETCD_INITIAL_ADVERTISE_PEER_URLS=http://$ETCD_HOST_IP:2380
export ETCD_INITIAL_CLUSTER=$ETCD_1_NAME=http://$ETCD_1_IP:2380,$ETCD_2_NAME=http://$ETCD_2_IP:2380,$ETCD_3_NAME=http://$ETCD_3_IP:2380
export ETCD_INITIAL_CLUSTER_STATE=exist
export ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
#为加入k8s-etcd-cluster集群的实例，需要将其ETCD_INITIAL_CLUSTER_STATE设置为"exist"
#部分环境变量带有/，导致sed出错，换成！作为表示
sed -i 's/default.etcd/'$ETCD_HOST_NAME'.etcd/' /etc/etcd/etcd.conf
sed -i 's/^ETCD_NAME=\"default\"/ETCD_NAME="'$ETCD_HOST_NAME'"/' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_LISTEN_PEER_URLS="http:\/\/localhost:2380"!ETCD_LISTEN_PEER_URLS="'$ETCD_LISTEN_PEER_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^ETCD_LISTEN_CLIENT_URLS="http:\/\/localhost:2379"!ETCD_LISTEN_CLIENT_URLS="'$ETCD_LISTEN_CLIENT_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^ETCD_ADVERTISE_CLIENT_URLS="http:\/\/localhost:2379"!ETCD_ADVERTISE_CLIENT_URLS="'$ETCD_ADVERTISE_CLIENT_URLS'"!' /etc/etcd/etcd.conf

sed -i 's!^#ETCD_INITIAL_ADVERTISE_PEER_URLS="http:\/\/localhost:2380"!ETCD_INITIAL_ADVERTISE_PEER_URLS="'$ETCD_INITIAL_ADVERTISE_PEER_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER="default=http:\/\/localhost:2380"!ETCD_INITIAL_CLUSTER="'$ETCD_INITIAL_CLUSTER'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER_STATE="new"!ETCD_INITIAL_CLUSTER_STATE="'$ETCD_INITIAL_CLUSTER_STATE'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"!ETCD_INITIAL_CLUSTER_TOKEN="'$ETCD_INITIAL_CLUSTER_TOKEN'"!' /etc/etcd/etcd.conf
```

#### 配置etcd-3

```bash
export ETCD_1_IP=192.168.0.201
export ETCD_1_NAME=k8s-etcd1
export ETCD_2_IP=192.168.0.211
export ETCD_2_NAME=k8s-etcd2
export ETCD_3_IP=192.168.0.212
export ETCD_3_NAME=k8s-etcd3
export ETCD_HOST_IP=$ETCD_3_IP
export ETCD_HOST_NAME=$ETCD_3_NAME

export ETCD_LISTEN_PEER_URLS=http://$ETCD_HOST_IP:2380
export ETCD_LISTEN_CLIENT_URLS=http://localhost:2379,http://$ETCD_HOST_IP:2379
export ETCD_ADVERTISE_CLIENT_URLS=http://localhost:2379,http://$ETCD_HOST_IP:2379
export ETCD_INITIAL_ADVERTISE_PEER_URLS=http://$ETCD_HOST_IP:2380
export ETCD_INITIAL_CLUSTER=$ETCD_1_NAME=http://$ETCD_1_IP:2380,$ETCD_2_NAME=http://$ETCD_2_IP:2380,$ETCD_3_NAME=http://$ETCD_3_IP:2380
export ETCD_INITIAL_CLUSTER_STATE=exist
export ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
#为加入k8s-etcd-cluster集群的实例，需要将其ETCD_INITIAL_CLUSTER_STATE设置为"exist"
#部分环境变量带有/，导致sed出错，换成！作为表示
sed -i 's/default.etcd/'$ETCD_HOST_NAME'.etcd/' /etc/etcd/etcd.conf
sed -i 's/^ETCD_NAME=\"default\"/ETCD_NAME="'$ETCD_HOST_NAME'"/' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_LISTEN_PEER_URLS="http:\/\/localhost:2380"!ETCD_LISTEN_PEER_URLS="'$ETCD_LISTEN_PEER_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^ETCD_LISTEN_CLIENT_URLS="http:\/\/localhost:2379"!ETCD_LISTEN_CLIENT_URLS="'$ETCD_LISTEN_CLIENT_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^ETCD_ADVERTISE_CLIENT_URLS="http:\/\/localhost:2379"!ETCD_ADVERTISE_CLIENT_URLS="'$ETCD_ADVERTISE_CLIENT_URLS'"!' /etc/etcd/etcd.conf

sed -i 's!^#ETCD_INITIAL_ADVERTISE_PEER_URLS="http:\/\/localhost:2380"!ETCD_INITIAL_ADVERTISE_PEER_URLS="'$ETCD_INITIAL_ADVERTISE_PEER_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER="default=http:\/\/localhost:2380"!ETCD_INITIAL_CLUSTER="'$ETCD_INITIAL_CLUSTER'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER_STATE="new"!ETCD_INITIAL_CLUSTER_STATE="'$ETCD_INITIAL_CLUSTER_STATE'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"!ETCD_INITIAL_CLUSTER_TOKEN="'$ETCD_INITIAL_CLUSTER_TOKEN'"!' /etc/etcd/etcd.conf
```

## \(etcd discovery\)服务发现配置

## \(dns discovery\)DNS发现配置

## etcd服务检验

### etcd 服务验证

```bash
# 服务器节点成员状态
curl http://192.168.0.201:2379/v2/members
# 服务器节点状态
curl http://192.168.0.201:2379/v2/stats/store
# 自身状态
curl http://192.168.0.201:2379/v2/stats/self
# 查看leader
curl http://192.168.0.201:2379/v2/stats/leader

#cli-api命令行
etcdctl endpoint status -w="table"

#查询etcd状态
netstat -ntlp|grep etcd
```

## etcd备份



```bash
date_time=`date +%Y%m%d`
etcdctl backup --data-dir /etcd/ --backup-dir /python/etcdbak/${date_time}
find /python/etcdbak/ -ctime +7 -exec rm -r {} \;
```

### etcd验证

```bash

```

### etcd服务启动配置方法

```text
[Unit]
Description=etcd2.service
[Service]
Type=notify
TimeoutStartSec=0
Restart=always
ExecStartPre=-/bin/mkdir -p /data/etcd2
ExecStart=/usr/bin/etcd \
  --data-dir /data/etcd2 \
  --name etcd1 \
  --advertise-client-urls http://172.18.6.102:2379,http://172.18.6.102:4001 \
  --listen-client-urls http://0.0.0.0:2379,http://0.0.0.0:4001 \
  --initial-advertise-peer-urls http://172.18.6.102:2380 \
  --listen-peer-urls http://0.0.0.0:2380 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster etcd0=http://172.18.6.101:2380,etcd1=http://172.18.6.102:2380,etcd2=http://172.18.6.103:2380
[Install]
WantedBy=multi-user.target
-----------------------------------------------------------------
# 设置服务自启动
systemctl enable /etc/systemd/system/etcd2.service
# 启动etcd2
systemctl restart etcd2.service
```

## etcd 常见错误

### 集群ID错误

```bash
#集群id错误
request cluster ID mismatch (got fa1282e3895996fc want cdf818194e3a8c32)
#删除了etcd集群所有节点中的--data_dir的内容
```

### 时间不同步

```bash
he clock difference against peer 5d951def1d1ebd99 is too high 
# 进行时间同步
```

