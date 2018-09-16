# centos环境

## network 设置

设置网络处于同一局域网内

`vi /etc/sysconfgi/network-script/ifconf-eth0`

设置hostname

```bash
hostnamectl set-hostname k8s-master1-1
vi /etc/hostname
```

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
yum -y install epel-release
yum update
#使用阿里云的源予以替换
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
yum makecache
```

### 防火墙 设置

```bash
#K8S集群安装前进行防火墙关闭
#安全的做法是在防火墙上配置各组件需要相互通信的端口号，这里选择直接禁用防火墙。
systemctl stop firewalld & systemctl disable firewalld
```

### Swap 设置

```bash
#关闭swap防止颠簸，注释# /dev/mapper/centos-swap swap 
sed -i '/ swap / s/^/#/' /etc/fstab
swapoff -a
#通过top查看swap是否为0
```

## yum-k8s 环境

{% code-tabs %}
{% code-tabs-item title="/etc/yum.repos.d/kubernetes.repo" %}
```text
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 网络内核设置

{% code-tabs %}
{% code-tabs-item title="/etc/sysctl.conf" %}
```text
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
#如果net.bridge.bridge-nf-call-iptables＝1，也就意味着二层的网桥在转发包时也会被iptables的FORWARD规则所过滤，这样就会出现L3层的iptables rules去过滤L2的帧的问题
sysctl -p
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 时间校对

```bash
#统一时区和时间信息
timedatectl set-local-rtc 1 
timedatectl set-timezone Asia/Shanghai 
timedatectl status
#时间进行校对
yum install -y ntp
systemctl start ntpd;systemctl enable ntpd
ntpdate ntp1.aliyun.com
hwclock -w
```


