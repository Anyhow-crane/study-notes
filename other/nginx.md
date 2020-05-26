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

```
 proxy_set_header X-Forwarded-Proto $scheme;
 proxy_set_header Host $host;
 proxy_set_header X-Forwarded-Port $server_port;
 proxy_set_header X-Real-IP $remote_addr;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

$scheme 为http/https,没有测试，一般感觉也用不到，nexus需要这个配置
$host 为最后一个nginx配置有效 显示为当前浏览器访问的host，最后一个nginx不配的话显示会有问题，所以最好每个nginx都加上这个配置
$host 改为$http_host 则显示host和port，但是需要每一级都要配置这个，才能传递
$server_port 为最后一个有这个配置的nginx有效 显示这个nginx配置的服务端口，最好第一个nginx配置
$proxy_add_x_forwarded_for 为每个配置加一个，没有配置还使用上级的，第一个必须配置这个，为服务器拿用户ip用，而且第一个nginx不能是docker容器，应为有端口映射，显示的ip是172.17.0.1容器主机ip
$remote_addr 为最后一个有这个配置的nginx有效 显示发到这个nginx服务上级ip，最好第一个nginx配置，docker容器里不要配这个
```

```
#upstream stdsvc {
#  server 192.168.18.201:8000;
#}
#upstream secsvc {
#  server 192.168.18.201:8100;
#}
#upstream extsvc {
#  server 192.168.18.201:8200;
#}
server {
  listen 80;
  server_name loongyun;
  index index.html;
  access_log /app/data/log/host.access.log main;
#  proxy_set_header Host $host;
  location / {
    root /app;
  }
#  location /stdsvc {
#    proxy_pass http://stdsvc;
#  }
#  location /secsvc {
#    proxy_pass http://secsvc;
#  }
#  location /extsvc {
#    proxy_pass http://extsvc;
#  }
}

```


