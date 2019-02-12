# docker学习

## 安装（使用阿里源，[文档](https://yq.aliyun.com/articles/110806)，[参考](https://opsx.alibaba.com/mirror)）

### 使用官方安装脚本自动安装

``` sh
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# 或者下载下来运行
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh --mirror Aliyun
```

> 有时会提示如下错误：正在尝试其它镜像。。。。然后下载不了，失败后直接使用`sudo yum -y install docker-ce`再尝试安装。所以最好用下面的方法。

### 手动安装帮助

#### Ubuntu 14.04 16.04

``` sh
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)
# sudo apt-get -y install docker-ce=[VERSION]
```

#### CentOS 7

``` sh
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo systemctl start docker

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

> 安装yum-utils，它提供一个yum-config-manager单元，同时安装的device-mapper-persistent-data和lvm2用于储存设备映射（devicemapper）必须的两个软件包。（只要安装yum-utils就可以了，官方给的安装脚本就只用到这个。）

### 安装后操作（centos）

``` sh
# 1:普通用户your-user的授权
sudo usermod -aG docker your-user
sudo systemctl restart docker
# 执行完该命令之后，需将该用户退出重新登录
# 2: 设置随系统启动
sudo systemctl enable docker.service
```

## 删除docker

``` sh
# 1:列出docker包的具体的名字。
yum list installed | grep docker
# 2:删除docker。
sudo yum remove docker-ce
# 3:清除镜像和容器文件。
sudo rm -rf /var/lib/docker
# 4:手工查找并删除用户创建的配置文件。
```

adas

``` sh
mkdir /etc/docker
cat <<EOF > /etc/docker/daemon.json
{
  "data-root": "/data/docker",
  "selinux-enabled": false,
  "registry-mirrors": ["https://docker.wisdragon.com"],
  "insecure-registries": ["docker.wisdragon.com"]
}
EOF
```

