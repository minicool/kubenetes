# k8s-etcd测试及节点操作

## k8s-etcd测试

### cli-api调用

```bash
#k8s-master-1
etcdctl put /test-dir/test-key "test"
#k8s-node-1
etcdctl get /test-dir/test-key
#查看数据是否同步获取
etcdctl update /test-dir/test-key "hello world"
etcdctl rm /test-dir/test-key
# 前缀查询 所有
etcdctl get /test-dir/test-key --prefix
etcdctl rm /test-dir/test-key --prefix
#申请租约
etcdctl lease grant 40
lease 051965d1ea57fa0c granted with TTL(40s)
#授权租约节点的生命伴随着租约到期将会被DELETE
etcdctl put --lease=051965d1ea57fa0c /test-dir/test-key/lease test
#撤销租约
etcdctl lease revoke 051965d1ea57fa0c
#租约续约
etcdctl lease keep-alive 051965d1ea57fa0c
```

### 网址调用

```text
#网址操作
#创建目录
curl http://192.168.0.201:2379/v2/keys/dir -XPUT -d dir=true
#创建键值
curl http://192.168.0.201:2379/v2/keys/hello -XPUT -d value="world"
#查看键值
curl http://192.168.0.201:2379/v2/keys
#创建带ttl键值
curl http://192.168.0.201:2379/v2/keys/ttlvar -XPUT -d value="ttl_value" -d ttl=10 
#创建有序键值
curl http://192.168.0.201:2379/v2/keys/seqvar -XPOST -d value="seq1"
curl http://192.168.0.201:2379/v2/keys/seqvar -XPOST -d value="seq2"
curl http://192.168.0.201:2379/v2/keys/seqvar -XPOST -d value="seq3"
curl http://192.168.0.201:2379/v2/keys/seqvar
#删除指定的键
curl http://192.168.0.201:2379/v2/keys/for_delete -XDELETE

```

### rpc远程调用

```text
#设定环境变量export ETCDCTL_API=3，设置成V3版本。
v3版本支持rpc的远程调用，
```

## k8s-etcd节点操作

### etcd集群操作

```text
# 查看节点
etcdctl member list
# 更新一个节点ip
etcdctl member update memberID http://ip:2380 
# 删除一个节点
etcdctl  member  remove  memberID
# 增加一个新节点
etcdctl member add k8s-etcd4 http://192.168.8.213:2380
# ETCD_NAME="k8s-etcd4"
# ETCD_INITIAL_CLUSTER="k8s-etcd1=http://192.168.0.201:2380,k8s-etcd4=http://192.168.8.213:2380,k8s-etcd3=http://192.168.0.212:2380,k8s-etcd2=http://192.168.0.211:2380"
# ETCD_INITIAL_CLUSTER_STATE="existing"
```

```bash
#临时新增节点
export ETCD_1_IP=192.168.0.201
export ETCD_1_NAME=k8s-etcd1
export ETCD_2_IP=192.168.0.211
export ETCD_2_NAME=k8s-etcd2
export ETCD_3_IP=192.168.0.212
export ETCD_3_NAME=k8s-etcd3
export ETCD_4_IP=192.168.0.213
export ETCD_4_NAME=k8s-etcd4
export ETCD_HOST_IP=$ETCD_4_IP
export ETCD_HOST_NAME=$ETCD_4_NAME

export ETCD_LISTEN_PEER_URLS=http://$ETCD_HOST_IP:2380
export ETCD_LISTEN_CLIENT_URLS=http://localhost:2379,http://$ETCD_HOST_IP:2379
export ETCD_ADVERTISE_CLIENT_URLS=http://localhost:2379,http://$ETCD_HOST_IP:2379
export ETCD_INITIAL_ADVERTISE_PEER_URLS=http://$ETCD_HOST_IP:2380
export ETCD_INITIAL_CLUSTER=$ETCD_1_NAME=http://$ETCD_1_IP:2380,$ETCD_2_NAME=http://$ETCD_2_IP:2380,$ETCD_3_NAME=http://$ETCD_3_IP:2380,$ETCD_4_NAME=http://$ETCD_4_IP:2380
export ETCD_INITIAL_CLUSTER_STATE=existing
export ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster

sed -i 's/default.etcd/'$ETCD_HOST_NAME'.etcd/' /etc/etcd/etcd.conf
sed -i 's/^ETCD_NAME=\"default\"/ETCD_NAME="'$ETCD_HOST_NAME'"/' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_LISTEN_PEER_URLS="http:\/\/localhost:2380"!ETCD_LISTEN_PEER_URLS="'$ETCD_LISTEN_PEER_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^ETCD_LISTEN_CLIENT_URLS="http:\/\/localhost:2379"!ETCD_LISTEN_CLIENT_URLS="'$ETCD_LISTEN_CLIENT_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^ETCD_ADVERTISE_CLIENT_URLS="http:\/\/localhost:2379"!ETCD_ADVERTISE_CLIENT_URLS="'$ETCD_ADVERTISE_CLIENT_URLS'"!' /etc/etcd/etcd.conf
​
sed -i 's!^#ETCD_INITIAL_ADVERTISE_PEER_URLS="http:\/\/localhost:2380"!ETCD_INITIAL_ADVERTISE_PEER_URLS="'$ETCD_INITIAL_ADVERTISE_PEER_URLS'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER="default=http:\/\/localhost:2380"!ETCD_INITIAL_CLUSTER="'$ETCD_INITIAL_CLUSTER'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER_STATE="new"!ETCD_INITIAL_CLUSTER_STATE="'$ETCD_INITIAL_CLUSTER_STATE'"!' /etc/etcd/etcd.conf
sed -i 's!^#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"!ETCD_INITIAL_CLUSTER_TOKEN="'$ETCD_INITIAL_CLUSTER_TOKEN'"!' /etc/etcd/etcd.conf
```

### etcd状态查询及备份

```bash
#数据查询
export ETCDCTL_API=3
etcdctl endpoint status --write-out=table
#数据备份及恢复
etcdctl snapshot save etcdback.db
etcdctl snapshot status etcdback.db --write-out=table
etcdctl snapshot restore etcdback.db --skip-hash-check=true
```

### etcd数据迁移

```text
scp -r /etcd/ 192.168.0.202:/etcd
```

