# Kubernetes学习

## 安装 （使用阿里源，[参考](https://opsx.alibaba.com/mirror)）

``` sh
# root用户
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
# 将 SELinux 设置为 permissive 模式(将其禁用)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

安装后操作（centos）

``` sh
# 1:关闭系统的Swap
swapoff -a
# 2:开放Kubernetes各个组件所需要的端口（修改后需要重启docker）
sudo firewall-cmd --zone=public --add-port=6443/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10250/tcp --permanent
sudo firewall-cmd --reload
# 或者直接关闭firewalld
sudo systemctl stop firewalld
sudo systemctl disable firewalld
# 可以提前下载需要的镜像文件
kubeadm config images pull

docker pull mirrorgooglecontainers/kube-apiserver:v1.13.1
docker pull mirrorgooglecontainers/kube-scheduler:v1.13.1
docker pull mirrorgooglecontainers/kube-controller-manager:v1.13.1
docker pull mirrorgooglecontainers/kube-proxy:v1.13.1
docker pull mirrorgooglecontainers/etcd:3.2.24
docker pull mirrorgooglecontainers/pause:3.1
docker pull coredns/coredns:1.2.6
docker tag mirrorgooglecontainers/kube-apiserver:v1.13.1 k8s.gcr.io/kube-apiserver:v1.13.1
docker tag mirrorgooglecontainers/kube-scheduler:v1.13.1 k8s.gcr.io/kube-scheduler:v1.13.1
docker tag mirrorgooglecontainers/kube-controller-manager:v1.13.1 k8s.gcr.io/kube-controller-manager:v1.13.1
docker tag mirrorgooglecontainers/kube-proxy:v1.13.1 k8s.gcr.io/kube-proxy:v1.13.1
docker tag mirrorgooglecontainers/etcd:3.2.24 k8s.gcr.io/etcd:3.2.24
docker tag mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag coredns/coredns:1.2.6 k8s.gcr.io/coredns:1.2.6
```

> kubernetes-1.13.1版本验证匹配的最高docker版本为18.06

