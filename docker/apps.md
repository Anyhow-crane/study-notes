# Docker安装使用常用软件

## Mysql

### 5.7版本

``` sh
# 1. 下载mysql
docker pull mysql:5.7
# 2. 运行一个mysql容器，设置表名不区分大小写，配置utf-8编码，root密码为wisdragon
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=wisdragon mysql:5.7 --lower_case_table_names=1 --character-set-server=utf8 --collation-server=utf8_general_ci
# 3. 修改mysql容器时区为上海时区
docker exec -it mysql /bin/bash
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
exit
# 4. 打包新镜像（可以上传镜像到公司仓库）
docker commit -a “feilong.li@wisdragon.com” -m “mysql5.7官方版本重做，root密码为wisdragon” mysql  docker.wisdragon.com/library/mysql:5.7
docker stop mysql && docker rm mysql && docker rmi mysql:5.7
docker tag docker.wisdragon.com/library/mysql:5.7 mysql:5.7
# 5. 运行mysql
docker run -d --name mysql -p 3306:3306 -v /data/mysql/datadir:/var/lib/mysql mysql:5.7
```

### 8.0版本

``` sh
# 1. 下载mysql
docker pull mysql:8.0
# 2. 运行一个mysql容器，设置表名不区分大小写，root密码为wisdragon（8.0后默认为utf-8）
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=wisdragon mysql:8.0 --lower_case_table_names=1
# 3. 修改mysql容器时区为上海时区
docker exec -it mysql /bin/bash
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
exit
# 4. 打包新镜像（可以上传镜像到公司仓库）
docker commit -a “feilong.li@wisdragon.com” -m “mysql8.0官方版本重做，root密码为wisdragon” mysql  docker.wisdragon.com/library/mysql:8.0
docker stop mysql && docker rm mysql && docker rmi mysql:8.0
docker tag docker.wisdragon.com/library/mysql:8.0 mysql:8.0
# 5. 运行mysql
docker run -d --name mysql -p 3306:3306 -v /data/mysql/datadir:/var/lib/mysql mysql:8.0
```

## nginx

```sh
# 下载稳定alpine版本
docker pull nginx:1.14-alpine
```

> 如果需要修改时区，可以使用上面mysql的方法，重新打包个新镜像，注意进入这个容器使用`docker exec -it nginx sh`，不是/bin/bash。

``` sh
docker run -d --name nginx -p 8000:80 -p 8100:443 nginx:1.14-alpine
```

> 基本运行，如果使用1000以下端口需要root权限。这种方式需要进入容器内修改配置文件，`vi /etc/nginx/conf.d/default.conf`

``` sh
docker run -d --name nginx -p 8000:80 -p 8100:443 -v /data/nginx:/etc/nginx/conf.d --link tomcat:tomcatname nginx:1.14-alpine
```

> 如果同一个机器上运行docker的代理服务，比如tomcat容器，可以使用--link 来链接，配置里可以直接使用tomcatname:8080来代理，tomcat容器都可以不用对外暴露端口了。