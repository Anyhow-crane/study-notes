# 安装

## docker安装

安装文件：[官网](https://github.com/oracle/docker-images/tree/master/OracleDatabase/SingleInstance)

**SingleInstance**为单数据库模式，官方对oracle11g只支持Express Edition版本，所以[官网](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index-092322.html)下载Oracle Database 11g Release 2 (11.2.0.2) Express Edition，下载linux-64位版本，放到`docker-images/OracleDatabase/SingleInstance/dockerfiles/11.2.0.2`目录下，安装命令：

```sh
cd docker-images/OracleDatabase/SingleInstance/dockerfiles
./buildDockerImage.sh -v 11.2.0.2 -x
```

安装完成就可以生成一个oracle/database:11.2.0.2-xe的镜像。

12c安装原理同上，可以安装Standard Edition 2版本，命令：`./buildDockerImage.sh -v 12.2.0.1 -s`

## docker使用

使用oracle/database:11.2.0.2-xe：

```sh
docker volume create oracle-data
docker run --name oracle -p 1521:1521 -e ORACLE_PWD=wiseloong -v oracle-data:/u01/app/oracle/oradata --shm-size=1g oracle/database:11.2.0.2-xe
```

> 注意要使用`--shm-size=1g`，不给不行；端口8080可以不用映射出来。

```sh
# 在容器中使用SQL*Plus
docker exec -it oracle sqlplus sys/<your password>@//localhost:1521/XE as sysdba
docker exec -it oracle sqlplus system/<your password>@//localhost:1521/XE
# 修改密码
docker exec oracle ./setPassword.sh <your password>
```

自制精简版：

```sh
# docker volume create oracle-data
docker run -d --name oracle -p 1521:1521 -v oracle-data:/u01/app/oracle/oradata --shm-size=1g pfs.wiseloong.com/wise/oracle:11-xe
```

> 默认密码为wiseloong。

``` 
docker run --link oracle --rm pfs.wiseloong.com/wise/oracle:11-xe sqlplus system/wiseloong@//oracle:1521/XE
```

```sh
docker run --rm -ti --link oracle oracle/database:19.3.0-ee sqlplus pdbadmin/<yourpassword>@//<db-container-ip>:1521/ORCLPDB1


docker run --rm -ti --link oracle -v $(pwd):/dmp --shm-size=1g pfs.wiseloong.com/wise/oracle:11-xe exp loong/loong@oracle/xe file=/dmp/1.dmp

docker run --rm -ti --link oracle -v $(pwd):/dmp --shm-size=1g pfs.wiseloong.com/wise/oracle:11-xe imp test/test@oracle/xe file=/dmp/1.dmp ignore=y full=y
```

