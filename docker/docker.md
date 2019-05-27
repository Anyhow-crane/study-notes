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
>
> 使用docker时不要登陆官方账号，登陆官方账号后默认从官方仓库地址获取镜像，不从配置的地址获取镜像了。

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
sudo systemctl enable docker
```

## 升级

```sh
sudo yum update docker-ce
sudo yum clean all
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
# 删除老版本docker
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```



``` sh
mkdir /etc/docker
cat <<EOF > /etc/docker/daemon.json
{
  "data-root": "/data/docker",
  "selinux-enabled": false,
  "registry-mirrors": ["https://pfs.wiseloong.com"]
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"] # 中科大
}
EOF
```

``` sh
sudo bash -c 'cat <<EOF > /etc/docker/daemon.json
{
  "data-root": "/data/docker",
  "selinux-enabled": false,
  "registry-mirrors": ["https://pfs.wiseloong.com"]
}
EOF'
```

``` sh
# --rm表示停止容器则自动删除容器，这个会在删除时一并删除volume，不能与-d同时使用
docker run --rm 镜像
# 删除容器时一并删除volume，只能删掉自动创建的volume，不会删掉你手动创建的volume
docker rm -v 容器
# 提交（-p 在commit时，将容器暂停。）
docker commit -p -a "feilong.li@wisdragon.com" -m "说明" 容器 新镜像名
```

```dockerfile
ENV TZ Asia/Shanghai
```



```sh
docker run -dit --name alpine1 --network alpine-net alpine ash
docker run -dit --name alpine alpine sh
```



# 数据卷-[官网](https://docs.docker.com/storage/volumes/)

## 常用命令

```sh
# 创建数据卷mysql-data
docker volume create mysql-data
# 查看数据卷列表
docker volume ls
# 查看数据卷信息，可以查看数据卷所在目录等
docker volume inspect mysql-data
# 删除数据卷
docker volume rm mysql-data
# 删除全部不用的数据卷
docker volume prune
```

## 数据卷备份和还原

1. 备份成压缩文件形式保存，以mysql为例：

```sh
# 创建数据卷
docker volume create mysql-data
# 运行mysql并挂载数据卷，使用的是制作后的镜像，原始官方镜像运行时需要设置root密码等参数
docker run -d --name mysql -p 3306:3306 -v mysql-data:/var/lib/mysql pfs.wiseloong.com/wise/mysql:5
# 连接数据库填加点数据（此步骤使用工具操作），再停止数据库
docker stop mysql
# 备份数据卷到当前目录，打包成压缩文件，使用的是alpine镜像，也可以使用ubuntu
docker run --rm --volumes-from mysql -v $(pwd):/backup alpine tar -zcf /backup/backup.tar.gz -C /var/lib/mysql ./
# 创建新数据卷2
docker volume create mysql-data2
# 运行新mysql2并挂载数据卷2
docker run -d --name mysql2 -p 13306:3306 -v mysql-data2:/var/lib/mysql pfs.wiseloong.com/wise/mysql:5
# 停止数据库2
docker stop mysql2
# 还原备份数据到新数据卷2
docker run --rm --volumes-from mysql2 -v $(pwd):/backup alpine sh -c 'rm -rf /var/lib/mysql/* && tar -zxf /backup/backup.tar.gz -C /var/lib/mysql'
# 启动数据库2
docker start mysql2
# 连接数据库2查看是否存在数据库1上增加的数据
```

> 备份还原时使用`--volumes-from mysql`，跟的是容器名称，不是数据卷名称；
>
> 使用`alpine`镜像，小巧，但是没用`bash`命令，所以使用`sh -c`，也可以换成官方使用的`ubuntu`，可以使用`bash -c`。

上面为官方方法改编，不太方便，还原时需要先启动容器，再停止容器，还原时还要清空数据卷，再解压；推荐下面更好的方法，可以在创建完新容器卷2后直接还原，再启动mysql容器2：

```sh
# 备份数据卷，不要丢掉最后的一个符号.，也可以写成./
docker run --rm -v mysql-data:/data -v $(pwd):/backup alpine tar -zcf /backup/backup.tar.gz -C /data .
# 还原数据卷，此步骤在创建完新数据卷mysql-data2后运行。执行完这步之后在运行新的mysql2容器
docker run --rm -v mysql-data2:/data -v $(pwd):/backup alpine tar -zxf /backup/backup.tar.gz -C /data
```

2. 备份成镜像文件，可以上传到镜像仓库

```sh
# 前面操作同上面，先创建容器卷，运行mysql，加点数据，停掉mysql（略）
# 备份，运行一个容器，备份数据卷数据到容器里，注意不要丢掉最后的一个符号.
docker run --name mysql-backup -v mysql-data:/data alpine tar -zcf /backup.tar.gz -C /data .
# 提交容器生成镜像
docker commit -a "feilong.li@wisdragon.com" -m "mysql备份数据" mysql-backup pfs.wiseloong.com/wise/mysql-backup:190321
# 上传镜像到仓库
docker push pfs.wiseloong.com/wise/mysql-backup:190321
# 删除容器
docker rm mysql-backup
# 换个服务器拉去数据镜像，在本机上模拟省略这步
docker pull pfs.wiseloong.com/wise/mysql-backup:190321
# 创建新数据卷2
docker volume create mysql-data2
# 恢复镜像中的备份数据到新数据卷2
docker run --rm -v mysql-data2:/data pfs.wiseloong.com/wise/mysql-backup:190321 tar -zxf /backup.tar.gz -C /data
# 运行新mysql2并挂载数据卷2
docker run -d --name mysql2 -p 13306:3306 -v mysql-data2:/var/lib/mysql pfs.wiseloong.com/wise/mysql:5
# 连接数据库2查看是否存在数据库1上增加的数据
```

上面恢复数据卷是在启动容器之前，比如使用docker-compose时是创建完容器卷和启动容器一起的，这时需要在使用`docker-compose up -d`命令后停掉容器`docker-compose stop`，再使用恢复数据命令，恢复后再启动`docker-compose start`；恢复命令如下：

```sh
# 恢复备份数据到新数据卷2，先删除新数据卷2里的数据，再解压数据
docker run --rm -v mysql-data2:/data pfs.wiseloong.com/wise/mysql-backup:190321 sh -c 'rm -rf /data/* && tar -zxf /backup.tar.gz -C /data'
```

# 通信

场景：一台服务器上2个运行的容器，一个数据库-p 3306:3306的mysql，一个后端服务-p 3000:3000的service；后端服务需要访问mysql数据库。

## link

运行后端服务时，通过选项--link连接

```sh
# 先运行mysql命令省略，再运行后台服务，类似如下命令：
docker run -d --name service --link mysql:mysql-link -p 3000:3000 images
```

> 后台服务数据库连接配置可设置为：jdbc:mysql://==mysql-link==:3306/db?useSSL=false&serverTimezone=PRC
>
> --link=mysql:mysql-link，前面的mysql为mysql容器名称，mysql-link为别名给service使用，也可以设置成--link=mysql:mysql，对应的配置为mysql:3306

## network

Docker安装的时候默认会创建三个不同的网络；none没用网络环境；host使用宿主机的网络，不安全，容器可以控制更改宿主机的网络；bridge默认的网络模式，每个容器会有它自己的虚拟网络接口连接到一个虚拟网桥上。

1. 常用命令

```sh
# 创建网络my-net，默认为bridge模式，使用-d overlay设置为overlay模式
docker network create mysql-data
# 查看网络列表
docker network ls
# 查看网络信息，可以查看数据卷所在目录等
docker network inspect mysql-data
# 删除网络
docker network rm mysql-data
# 删除全部不用的网络，默认创建的3个保留
docker network prune
```

2. run

```sh
# 创建网络，默认为bridge模式
docker network create my-net
# 运行mysql容器，加入my-net网络
docker run -d --name mysql --network my-net -p 3306:3306 images
# 运行后台服务容器，加入my-net网络
docker run -d --name service --network my-net -p 3000:3000 images
```

> 后台服务数据库连接配置可设置为：jdbc:mysql://==mysql==:3306/db?useSSL=false&serverTimezone=PRC

3. docker-compose

```yaml
version: '3'
# networks:
#   default:
services:
  hrms-db:
    image: mysql:5
    container_name: hrms-db-1
    ports:
      - "3306:3306"
    networks:
      - default
  hrms-s:
    image: hrms-s:1.0
    container_name: hrms-s-1
    ports:
      - "3000:3000"
    networks:
      - default
```

> 后台服务数据库连接配置可设置为：jdbc:mysql://==hrms-db==:3306/db?useSSL=false&serverTimezone=PRC
>
> 上面的yml文件为简化版，还故意把services名称和容器名称不同化，表明网络名称为services名称，不是容器名称。也可以单独给后台服务配置links:\n - "hrms-db:hrms-db"，感觉还是networks高大上点。
>
> docker-compose默认会创建一个`[projectname]_default`的网络，所以上面的注掉部分为默认创建的。

# Docker Compose

mac和windows安装完docker都自带Docker Compose。

## 在线安装-[官网文档](https://docs.docker.com/compose/install/)

``` sh
# 下载文件到目录，修改版本为最新的版本（真的慢,可使用下面）
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 给可运行权限
sudo chmod +x /usr/local/bin/docker-compose
# 查看版本
docker-compose --version
```

## 离线安装

下载最新版本的离线文件，[docker-compose-Linux-x86_64](https://github.com/docker/compose/releases/)，复制到服务器。

```sh
# 移动上面的文件到目录
sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
# 给可运行权限
sudo chmod +x /usr/local/bin/docker-compose
```

## 使用

```sh
# 后台启动
docker-compose up -d
```

## 数据卷

```yaml
version: '3'
services:
  hrms-db:
    image: pfs.wiseloong.com/wise/mysql:5
    container_name: hrms-db
    ports:
      - "3306:3306"
    volumes:
      - mysql:/var/lib/mysql
volumes:
  mysql:
networks:
  default:
```

遇到的问题：上面示例为数据库配置数据卷，使用`docker-compose up -d`，创建的数据卷名称不是mysql，而是hrms_mysql，可能在不同的环境hrms还不相同，使用`docker volume inspect hrms_mysql`，查看数据卷时发现hrms是`project name`项目名称，这个默认是`directory name`文件目录名称，也可以使用参数-p 指定；所以在数据卷数据备份恢复时，注意名称的变化；`docker-compose.yml`文件里统一名称就可以了。networks同样，创建的名字不叫default，为hrms_default。

```
environment:
        TZ: 'Asia/Shanghai'
```

```
docker run -d --name gitlab-runner-config \
    -v /etc/gitlab-runner \
    busybox:latest \
    /bin/true

docker run -d --name gitlab-runner --restart always \
    --volumes-from gitlab-runner-config \
    gitlab/gitlab-runner:latest
```

# docker-machine

## 安装

``` sh
curl -L https://github.com/docker/machine/releases/download/v0.16.0/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine && 
  chmod +x /usr/local/bin/docker-machine
```

> mac win 安装docker后自带docker-machine。

## 给远程服务器安装docker

1. 开启免密登陆

```sh
# 生成秘钥对，一直点回车，（一次就好）
ssh-keygen
# 将公钥传输给远程服务器，需要输入它的root密码
ssh-copy-id root@192.168.1.1
```

2. 开放2376端口

安装时使用--generic-engine-port参数默认为2376，如果不开放这个端口，安装后主机没法接收信息，会提示错误，实际上已经安装成功，只是主机无法监控。

```sh
# 登陆远程服务器
ssh root@192.168.1.1
firewall-cmd --zone=public --add-port=2376/tcp --permanent
firewall-cmd --reload
exit
```

3. 安装docker

``` sh
# 普通安装
docker-machine create -d generic --generic-ip-address=192.168.1.1 centos-1
# 使用配置参数
docker-machine create -d generic \
  --engine-opt data-root=/data/docker \
  --engine-opt selinux-enabled=false \
  --engine-registry-mirror https://pfs.wiseloong.com \
  --engine-insecure-registry registry.myco.com \
  centos-1
```

> Boot2Docker 有待研究

