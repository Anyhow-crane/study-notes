# 安装

## 普通安装脚本

解压oraclexe的安装包里有下面的xe.rsp。

```sh
#!/bin/bash
 
rpm -ivh  /downloads/oracle-xe-11.2.0-1.0.x86_64 > /xe_logs/XEsilentinstall.log

/etc/init.d/oracle-xe configure responseFIle=<location of xe.rsp> >> /xe_logs/XEsilentinstall.log
```

## docker安装

安装文件：[官网](https://github.com/oracle/docker-images/tree/master/OracleDatabase/SingleInstance)

**SingleInstance**为单数据库模式，官方对oracle11g只支持Express Edition版本，所以[官网](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/index-092322.html)下载Oracle Database 11g Release 2 (11.2.0.2) Express Edition，下载linux-64位版本，放到`docker-images/OracleDatabase/SingleInstance/dockerfiles/11.2.0.2`目录下，ENV添加2个环境变量：

```
    TZ=Asia/Shanghai \
    ORACLE_PWD=wiseloong \
```

安装命令：

```sh
cd docker-images/OracleDatabase/SingleInstance/dockerfiles
./buildDockerImage.sh -x -v 11.2.0.2
```

安装完成就可以生成一个oracle/database:11.2.0.2-xe的镜像。

12c安装原理同上，可以安装Standard Edition 2版本，命令：`./buildDockerImage.sh -v 12.2.0.1 -s`

## docker使用

使用oracle/database:11.2.0.2-xe：

```sh
docker volume create oracle-data
docker run -d --name oracle -p 1521:1521 -e ORACLE_PWD=wiseloong -v oracle-data:/u01/app/oracle/oradata --shm-size=1g oracle/database:11.2.0.2-xe
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
docker run --rm -it --shm-size=1g --link oracle oracle/database:19.3.0-ee sqlplus pdbadmin/<yourpassword>@//<db-container-ip>:1521/ORCLPDB1

docker run --rm -it --shm-size=1g --link oracle -v $(pwd):/dmp --shm-size=1g pfs.wiseloong.com/wise/oracle:11-xe exp loong/loong@oracle/xe file=/dmp/1.dmp

docker run --rm -it --link oracle -v $(pwd):/dmp --shm-size=1g pfs.wiseloong.com/wise/oracle:11-xe imp test/test@oracle/xe file=/dmp/1.dmp ignore=y full=y
```

## 启动脚本启动docker

创建启动脚本`01_init.sql`：

```sql
alter profile DEFAULT limit password_life_time unlimited;
alter system set deferred_segment_creation=false;
shutdown immediate;
startup restrict;
ALTER DATABASE character set INTERNAL_USE ZHS16GBK;
shutdown immediate;
startup;
```

```sh
docker run -d --name oracle2 -p 1522:1521  -v $(pwd)/setup:/u01/app/oracle/scripts/setup --shm-size=1g oracle/database:11.2.0.2-xe
```

## 字符集修改

xe默认字符集为AL32UTF8，有时候需要转换成生产环境的ZHS16GBK，防止导入时长度不足，先创建执行文件`setCharacter.sh`，如下

```
sql> shutdown immediate;
sql> startup restrict;
sql> ALTER DATABASE character set INTERNAL_USE ZHS16GBK;
sql> shutdown immediate;
sql> startup;
sql> quit;
```

```sh
#!/bin/bash

echo 'export NLS_LANG=AMERICAN_AMERICA.ZHS16GBK' >> /etc/profile
source /etc/profile

su -p oracle -c "sqlplus / as sysdba << EOF
    shutdown immediate;
    startup mount;
    alter session set SQL_TRACE=TRUE;
    alter system enable restricted session; 
    alter system set JOB_QUEUE_PROCESSES=0; 
    alter system set AQ_TM_PROCESSES=0; 
    alter database open; 
    alter database character set internal_use ZHS16GBK; 
    alter session set SQL_TRACE=FALSE;
    shutdown immediate;
    startup;
    exit;
EOF"
```

```sh
# 复制上面文件到docker里
docker cp ./setCharacter.sh oracle:/
# 进入docker修改权限
docker exec -it oracle /bin/bash
chmod 777 /setCharacter.sh
exit
# 执行
docker exec oracle /setCharacter.sh
```

> 直接使用docker exec oracle sqlplus这种方式进入后connect / as sysdba;报错ORA-12547: TNS:lost contact，没找到解决办法，所以用上面的方法，当需要连接操作都可以使用上面的方法。

## 安装后设置

```sql
-- system登陆
-- 取消密码过期时间
alter profile DEFAULT limit password_life_time unlimited;
-- 导出空表，xe默认不导出空表
alter system set deferred_segment_creation=false;
```

## 新建用户

```sql
-- 新增用户
create user loong
identified by loong
default tablespace USERS
temporary tablespace TEMP;
-- 给管理员权限
grant dba to loong;
-- 授权一般用户
GRANT CONNECT,RESOURCE TO loong;
-- 删除用户
drop user loong cascade;	
```

```sql
-- 查询时间
select sysdate from dual;
-- 数据库编码
select userenv('language') from dual;
SELECT value$ FROM sys.props$ WHERE name = 'NLS_CHARACTERSET' ;
--查询oracle 相关参数
SELECT * FROM NLS_DATABASE_PARAMETERS ;
```

```sql
CREATE TABLESPACE iscu DATAFILE '/u01/app/oracle/oradata/XE/owndb.dbf' size 100M autoextend on next 20M maxsize unlimited;

CREATE USER iscu IDENTIFIED BY iscu DEFAULT TABLESPACE iscu;

--
GRANT CONNECT,RESOURCE,CREATE VIEW  TO iscu;

CONN iscu/iscu;


create table ISCU_ACCOUNT
(
  id           VARCHAR2(32) not null,
  username      VARCHAR2(32),
  isavaliable  INTEGER
)
;
alter table ISCU_ACCOUNT
  add constraint PK_ISC_ACCOUNT primary key (ID);

INSERT INTO ISCU_ACCOUNT values('12345678901234567890','一二三四五六七八九十一二三四五六',10);
```

```sql
-- 创建类型
CREATE OR REPLACE TYPE wm_concat_impl   
  AUTHID CURRENT_USER
AS OBJECT (
   curr_str   VARCHAR2 (32767),
   STATIC FUNCTION odciaggregateinitialize (sctx IN OUT wm_concat_impl)
      RETURN NUMBER,
   MEMBER FUNCTION odciaggregateiterate (
      SELF   IN OUT   wm_concat_impl,
      p1     IN       VARCHAR2
   )
      RETURN NUMBER,
   MEMBER FUNCTION odciaggregateterminate (
      SELF          IN       wm_concat_impl,
      returnvalue   OUT      VARCHAR2,
      flags         IN       NUMBER
   )
      RETURN NUMBER,
   MEMBER FUNCTION odciaggregatemerge (
      SELF    IN OUT   wm_concat_impl,
      sctx2   IN       wm_concat_impl
   )
      RETURN NUMBER
);
/

CREATE OR REPLACE TYPE BODY wm_concat_impl
IS
   STATIC FUNCTION odciaggregateinitialize (sctx IN OUT wm_concat_impl)
      RETURN NUMBER
   IS
   BEGIN
      sctx := wm_concat_impl (NULL);
      RETURN odciconst.success;
   END;
   MEMBER FUNCTION odciaggregateiterate (
      SELF   IN OUT   wm_concat_impl,
      p1     IN       VARCHAR2
   )
      RETURN NUMBER
   IS
   BEGIN
      IF (curr_str IS NOT NULL)
      THEN
         curr_str := curr_str || ',' || p1;
      ELSE
         curr_str := p1;
      END IF;

      RETURN odciconst.success;
   END;
   MEMBER FUNCTION odciaggregateterminate (
      SELF          IN       wm_concat_impl,
      returnvalue   OUT      VARCHAR2,
      flags         IN       NUMBER
   )
      RETURN NUMBER
   IS
   BEGIN
      returnvalue := curr_str;
      RETURN odciconst.success;
   END;
   MEMBER FUNCTION odciaggregatemerge (
      SELF    IN OUT   wm_concat_impl,
      sctx2   IN       wm_concat_impl
   )
      RETURN NUMBER
   IS
   BEGIN
      IF (sctx2.curr_str IS NOT NULL)
      THEN
         SELF.curr_str := SELF.curr_str || ',' || sctx2.curr_str;
      END IF;

      RETURN odciconst.success;
   END;
END;
/

CREATE OR REPLACE FUNCTION wm_concat (p1 VARCHAR2)
   RETURN VARCHAR2
   AGGREGATE USING wm_concat_impl;
/

--将wm_concat授权给所有人用
grant execute on wm_concat to public;
```

## 数据泵

```sql
-- 创建文件夹
CREATE DIRECTORY DUMP_DIR AS '/<dump_folder>';
GRANT read, write ON DIRECTORY DUMP_DIR TO system;
--导出
expdp system/system_password full=Y EXCLUDE=SCHEMA:\"LIKE \'APEX_%\'\",SCHEMA:\"LIKE \'FLOWS_%\'\" directory=DUMP_DIR dumpfile=DB10G.dmp logfile=expdpDB10G.log
--导出单表
expdp system/system_password TABLES=FLOWS_FILES.WWV_FLOW_FILE_OBJECTS$ directory=DUMP_DIR dumpfile=DB10G2.dmp logfile=expdpDB10G2.log
--导入
impdp  system/system_password full=Y directory=DUMP_DIR dumpfile=DB10G.dmp logfile=expdpDB10G1.log
--导入单表
impdp  system/system_password directory=DUMP_DIR TABLE_EXISTS_ACTION=APPEND TABLES=FLOWS_FILES.WWV_FLOW_FILE_OBJECTS$ dumpfile=DB10G2.dmp logfile=expdpDB10G1b.log
```

