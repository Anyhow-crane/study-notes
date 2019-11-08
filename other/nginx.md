# 安装

```sh
# 1 切换root账号
su -
# 2 添加nginx的yum源文件
cat <<EOF > /etc/yum.repos.d/nginx.repo
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/7/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
EOF
# 3 安装
yum install -y nginx
# 4 开启开机启动nginx
systemctl enable nginx
# 5 运行nginx
systemctl start nginx
/etc/nginx/conf.d/default.conf

nginx -c /etc/nginx/nginx.conf
nginx -s reload

sudo systemctl disable nginx
```

> yum源文件参考[官网](http://nginx.org/en/linux_packages.html#RHEL-CentOS)：baseurl路径中7为centos版本，x86_64为$basearch，可以使用`cat /proc/version`查看

1. 网站文件存放默认目录: `/usr/share/nginx/html`
2. 网站默认站点配置: `/etc/nginx/conf.d/default.conf`
3. 自定义Nginx站点配置文件存放目录: `/etc/nginx/conf.d/*.conf`
4. Nginx全局配置: `/etc/nginx/nginx.conf`
