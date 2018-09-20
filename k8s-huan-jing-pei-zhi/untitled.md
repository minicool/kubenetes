# 配置kubectl

## **kubectl.config文件**

```bash
export KUBE_APISERVER="https://192.168.0.201:8080"

# 设置集群参数
kubectl config set-cluster kubernetes
  --certificate-authority=/etc/kubernetes/ssl/ca/ca.pem
  --embed-certs=true
  --server=${KUBE_APISERVER}

# 设置客户端认证参数
kubectl config set-credentials admin
  --client-certificate=/etc/kubernetes/ssl/admin/admin.pem
  --embed-certs=true
  --client-key=/etc/kubernetes/ssl/admin/admin-key.pem

# 设置上下文参数
kubectl config set-context kubernetes
  --cluster=kubernetes
  --user=admin

# 设置默认上下文
kubectl config use-context kubernetes

# KUBE_APISERVER 这里指定SLB的地址, 因为我们是通过SLB来请求master的kube-apiserver.
# admin.pem 证书 OU 字段值为 system:masters，kube-apiserver 预定义的 RoleBinding cluster-admin 将 Group system:masters 与 Role cluster-admin 绑定，该 Role 授予了调用kube-apiserver 相关 API 的权限；
# 生成的 kubeconfig 被保存到 ~/.kube/config 文件；
# 可以将~/.kube/config文件复制到其他用户目录下, 这样其他用户也可以使用kubectl命令了
```

##  **TLS Bootstrapping Token**

{% code-tabs %}
{% code-tabs-item title="token.csv" %}
```bash
# 随机生成token值
export BOOTSTRAP_TOKEN=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ')
# 写入文件 token.csv Token,用户名,UID,用户组
cd /etc/kubernetes
echo "${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap"" > token.svc
#尽量不要更新BOOTSTRAP_TOKEN值, 如果要更新, kube-apiserver使用的token.csv文件和kubelet使用的bootstrap.kubeconfig文件都需要更新, 更新后还需要重启kube-apiserver和kube-proxy服务
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## **创建bootstrapping.kubeconfig文件**

```bash
cd /etc/kubernetes
# 设置集群参数
kubectl config set-cluster kubernetes 
  --certificate-authority=/etc/kubernetes/ssl/ca.pem 
  --embed-certs=true 
  --server=${KUBE_APISERVER} 
  --kubeconfig=bootstrap.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials kubelet-bootstrap 
  --token=${BOOTSTRAP_TOKEN} 
  --kubeconfig=bootstrap.kubeconfig

# 设置上下文参数
kubectl config set-context default 
  --cluster=kubernetes 
  --user=kubelet-bootstrap 
  --kubeconfig=bootstrap.kubeconfig

# 设置默认上下文
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig

# --embed-certs 为true时, 表示将certificate-authority证书写入到生成的bootstrap.kubeconfig文件中；
```

## **创建kube-proxy.kubeconfig文件**

```bash
cd /etc/kubernetes
# 设置集群参数
kubectl config set-cluster kubernetes 
  --certificate-authority=/etc/kubernetes/ssl/ca/ca.pem 
  --embed-certs=true 
  --server=${KUBE_APISERVER} 
  --kubeconfig=kube-proxy.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials kube-proxy 
  --client-certificate=/etc/kubernetes/ssl/kube-proxy/kube-proxy.pem \
  --client-key=/etc/kubernetes/ssl/kube-proxy-key.pem 
  --embed-certs=true 
  --kubeconfig=kube-proxy.kubeconfig

# 设置上下文参数
kubectl config set-context default 
  --cluster=kubernetes 
  --user=kube-proxy 
  --kubeconfig=kube-proxy.kubeconfig

# 设置默认上下文
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

## **kubeconfig文件分发**

```bash
# 将token.csv复制到其他master相对应的/etc/kubernetes中, 供kube-apiserver使用;
scp /etc/kubernetes/token.csv root@192.168.0.202:/etc/kubernetes/
# 将bootstrap.kubeconfig和kube-proxy.kubeconfig复制到其他node节点的/etc/kubernetes中, 供kubele和kube-proxy使用
scp /etc/kubernetes/*.kubeconfig root@192.168.0.211:/etc/kubernetes/
# 将~/.kube/config复制到任意想使用kubectl命令行工具的服务器中
scp ~/.kube/config root@192.168.0.211:~/.kube/
```

