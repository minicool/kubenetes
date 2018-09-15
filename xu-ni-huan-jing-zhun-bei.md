---
description: 笔者在mac进行虚拟环境的搭建
---

# 环境规划

建议下载Parallers Deskop for mac 虚拟机进行环境准备

## 虚拟机数量要求

为建立集群，建立设置在同一局域网

k8s-master-1 192.168.0.201

k8s-slave-1 192.168.0.211

k8s-etcd-1 192.168.0.221

## hostname 配置

```bash
192.168.0.201 k8s-master-1 k8s-etcd-1 node-ip-201
192.168.0.211 k8s-slave-1 node-ip-211
192.168.0.221 k8s-slave-2 node-ip-212
```

## 虚拟机环境要求

* 操作系统 CentOS 7.4
* 内存 2G 【至少】
* CPU 2核【至少】
* 硬盘 20G 【至少】



