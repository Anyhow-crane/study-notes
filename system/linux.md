# init-初始化

1. 关闭centos7-SElinux

```sh
# 把配置文件里的SELINUX=enforcing，改成permissive
sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
# 重启
reboot
# 如果暂时不想重启可以临时关闭命令
setenforce 0
# 查看SElinux状态命令
getenforce
```

2. 修改时区，如果时区不对，比如下载的docker镜像

```bash
# 查看时间（date -R 可查看时区）
date
# 创建软连接
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

3. 更新时间，时间不准

```bash
# 查看系统时间
date
# 查看硬件时间
hwclock
# 安装工具
yum -y install ntpdate
# 更新时间
ntpdate ntp.aliyun.com
# 将系统时间写入硬件时间
hwclock --systohc
```

4. 切换阿里yum源

```bash
# 备份原来的repo文件
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 下载阿里repo文件
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 修改非阿里云ECS用户错误
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
# 重建缓存
yum clean all
yum makecache
```

# 挂载目录（root）

```bash
# 查看磁盘信息（后面的/dev/sdb为磁盘位置）
fdisk -l
# 格式化磁盘（centos7使用xfs格式）
mkfs.xfs /dev/sdb
# 创建目录
mkdir /data
# 挂载磁盘
mount /dev/sdb /data
# 自动挂载（防止重启后挂载失效,追加配置信息到/etc/fstab）
echo '/dev/sdb /data xfs defaults 0 0' >> /etc/fstab
# 扩容（虚拟机管理中增加硬盘的容量后，使用下面命令扩容）
xfs_growfs /dev/sdb
```

> xfs文件系统只支持增加，不支持减少；如果需要减小，则需要重新格式化，数据丢失。

# 创建用户

```bash
# 新建用户wise，并给默认目录为/data/wise
useradd wise -d /data/wise
# 给用户wise设置密码
passwd wise
# 把wise添加到管理员组去
usermod -aG wheel wise
```

## 开放端口

```bash
# 查看防火墙状态
sudo firewall-cmd --state
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --reload
sudo systemctl stop firewalld.service #停止firewall
sudo systemctl disable firewalld.service #禁止firewall开机启动
```

## 压缩和解压

```bash
tar -zcf xx.tar.gz ./xx    #压缩（通过gzip指令处理备份文件）（真的压缩）
tar -zxf xx.tar.gz        #解压（通过gzip指令处理备份文件）
```

## 安装jdk

```bash
# Debian, Ubuntu, etc.
sudo apt-get install openjdk-8-jre
# Fedora, Oracle Linux, Red Hat Enterprise Linux, etc.
sudo yum install java-1.8.0-openjdk
# develop Java 使用 java-1.8.0-openjdk-devel
```

## 使用cat命令

```bash
# 1. 查看文件内容
cat xxx.txt
# 2. 修改文件内容，相当于替换全部内容
cat <<EOF > /xxx/xxx.txt
#
#
EOF
# 3. 追加内容到末尾
cat <<EOF >> /xxx/xxx.txt
#
#
EOF
# 4. 当修改的文件需要root权限时
sudo bash -c 'cat <<EOF >> /xxx/xxx.txt
#
#
EOF'
```

# 其他命令

```bash
# 关机
poweroff
# 重启
reboot
# 查看系统版本信息
cat /etc/os-release
cat /etc/redhat-release
rpm -q centos-release
# 查看host
hostname
# 查看系统信息，如cpu
lscpu
```

```bash
# 1.sh
#!/bin/bash
su - wisd
pwd
echo 'wisdragon' | sudo -S sh /root/2.sh

# 2.sh
#!/bin/bash
echo "wisdragon" | sudo -S mkdir 123


MYDATE=temp_`date +%Y%m%d`
```

## 判断一个命令是否存在

```bash
# 判断命令是否存在
command_exists() {
    command -v "$@" > /dev/null 2>&1
}

if command_exists docker; then
    ...
fi
```

## > 与 &>

liunx中“&>" 一般一个搜索命令有标准输出和标准错误输出，也就是”>"和“2>" 这两个的结合体就是 ”&>" 结果就是你输入命令，正确和错误信息都不输出在屏幕上直接输到你重定向的一个文件内。

## EOF和-EOF的区别

没有-的话，EOF作为结束符，前面不能有任何tab制表符，报个异常信息。有-的话，EOF作为结束符，前面可以有tab制表符,容错率更高一点。

## 2>&1

2>&1 的意思就是将标准错误重定向到标准输出。这里标准输出已经重定向到了 /dev/null。那么标准错误也会输出到/dev/null

```bash
index.php task testOne >/dev/null 2>&1
```

```bash
wise_add_dns() {
    dns=$(grep -c "nameserver 114.114.114.114" /etc/resolv.conf)
    if [ "$dns" -eq '0' ]; then
    echo "添加dns 114.114.114.114 "
        $wise_bash_c "echo 'nameserver 114.114.114.114' >> /etc/resolv.conf"
    fi
}

wise_ali_dns="nameserver 223.5.5.5
nameserver 223.6.6.6"

# 添加dns
wise_add_dns() {
    dns=$(grep -c "nameserver 223.5.5.5" /etc/resolv.conf)
    if [ "$dns" -eq '0' ]; then
    echo "添加dns"
        $wise_bash_c "cat >> /etc/resolv.conf <<-EOF
            $wise_ali_dns 
            EOF"
    fi
}
```

## ssh 免密登陆

```bash
# 生成公钥私钥，直接回车（默认目录文件），回车（不创建密码），回车
ssh-keygen
# 复制公钥到远程服务器的用户下
ssh-copy-id root@10.211.55.9
```

## centos系统升级

```bash
# 查看系统信息
cat /etc/redhat-release
# 清理
yum clean all
# 升级
yum update
# 重启
reboot
# 注意：升级的时候最好等升级结束再进行其他操作，不要再开一个窗口来升级另外一个服务器
```

## 修改 hostname

```bash
# 修改 hostname
hostnamectl set-hostname your-new-host-name
# 查看修改结果
hostnamectl status
# 设置 hostname 解析
echo "127.0.0.1   $(hostname)" >> /etc/hosts
```
