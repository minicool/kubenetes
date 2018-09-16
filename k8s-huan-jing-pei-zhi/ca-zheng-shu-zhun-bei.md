# ca证书

## 安装cfssl

### yum 下载安装

### wget 下载安装

```text
#安装wget 
yum -y install wget
# 创建目录并下载文件
mkdir -p /opt/k8s/cfssl
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
# 移动修改文件名
mv cfssl_linux-amd64 /opt/k8s/cfssl/cfssl
mv cfssljson_linux-amd64 /opt/k8s/cfssl/cfssljson
mv cfssl-certinfo_linux-amd64 /opt/k8s/cfssl/cfssl-certinfo
# 设置可执行权限
chmod +x /opt/k8s/cfssl/*
# 添加到环境变量, 底部追加
vi /etc/profile
##############################################
export PATH=$PATH:/opt/k8s/cfssl
##############################################

# 使之生效
source /etc/profile
```

