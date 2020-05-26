# Kubernetes学习

## 安装 （使用阿里源，[参考](https://opsx.alibaba.com/mirror)）

```sh
sudo tee /etc/yum.repos.d/kubernetes.repo <<-'EOF'
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
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
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet
systemctl enable --now kubelet 
```

安装后操作（centos）

```sh
# 1:关闭系统的Swap
swapoff -a # 暂时关闭
sed -i.bak '/swap/s/^/#/' /etc/fstab # 永久禁用
# 2:开放Kubernetes各个组件所需要的端口（修改后需要重启docker）
sudo firewall-cmd --zone=public --add-port=6443/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10250/tcp --permanent
sudo firewall-cmd --reload
# 或者直接关闭firewalld
sudo systemctl stop firewalld
sudo systemctl disable firewalld
# 必须先安装 nfs-utils 才能挂载 nfs 网络存储
sudo yum install -y nfs-utils
# 查看要pull的镜像列表
kubeadm config images list
kubeadm config images list --kubernetes-version=$version|awk -F '/' '{print $2}'
# 可以提前下载需要的镜像文件
kubeadm config images pull

kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers


docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.17.3 k8s.gcr.io/kube-apiserver:v1.17.3
docker tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.17.3 k8s.gcr.io/kube-controller-manager:v1.17.3
docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.17.3 k8s.gcr.io/kube-scheduler:v1.17.3
docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.17.3 k8s.gcr.io/kube-proxy:v1.17.3
docker tag registry.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
docker tag registry.aliyuncs.com/google_containers/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0
docker tag registry.aliyuncs.com/google_containers/coredns:1.6.5 k8s.gcr.io/coredns:1.6.5
```

> kubernetes-1.13.1版本验证匹配的最高docker版本为18.06

```sh
kubectl config use-context docker-for-desktop
```
