# centos环境

## network 设置

设置网络处于同一局域网内

`vi /etc/sysconfgi/network-script/ifconf-eth0`

设置hostname

`vi /etc/hostname`

设置hosts 指向hostname

`vi /etc/hosts` 

刷新环境

`service network restart`



## iptables 设置

```bash
chmod +x /etc/rc.d/rc.local 
iptables -I INPUT -s 192.168.0.0/24 -j ACCEPT iptables -I FORWARD -j ACCEPT 
echo 'iptables -I INPUT -s 192.168.0.0/24 -j ACCEPT' >> /etc/rc.d/rc.local 
echo 'iptables -I FORWARD -j ACCEPT' >> /etc/rc.d/rc.local
```

## selinux设置

```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
setenforce 0
```

## yum源 设置

```bash
#使用阿里云的源予以替换
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
yum makecache
```

### 防火墙 设置

```bash
#K8S集群安装前进行防火墙关闭
systemctl stop firewalld & systemctl disable firewalld
```

### Swap 设置

```bash
#关闭swap防止颠簸，注释# /dev/mapper/centos-swap swap 
sed -i '/ swap / s/^/#/' /etc/fstab
swapoff -a
#通过top查看swap是否为0
```



