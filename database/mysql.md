# mysql日常使用记录

##连接池查看语句

```sql
show full processlist;
```

> 碰到线程池卡住了，可以重启mysql，写代码时记得使用事务。

## 判断不为1，有可能为null

有时我们需要判断不为1，如果仅仅用`o.is_sys_var <> 1`，他同时会排出掉null的值，

```sql
select o.*
from mm_object o
where o.is_valid = 1
and ifnull(o.is_sys_var, 0) <> 1
```

> `ifnull(exp_1,exp_2)`函数接受2个参数，如果exp_1为null则返回exp_2，否则返回exp_1。

## docker下的mysql5.7的修改配置文件

``` sh
# 进入mysql容器
docker exec -it mysql /bin/bash
# 给配置文件追加配置内容
cat <<EOF >> /etc/mysql/mysql.conf.d/mysqld.cnf
# user-conf
group_concat_max_len=10240
EOF
# 退出
exit
# 重启mysql容器
docker restart mysql
```

上面以修改group_concat()最大值为例，docker版mysql5.7的配置文件目录：==/etc/mysql/mysql.conf.d/mysqld.cnf==

## group_concat()最大值

问题：因为mysql默认group_concat()最大值为1024，当需要合并的id比较多，则返回不全，需要修改这个最大值。

1. 当不能擅自重启MySQL服务时，可通过sql语句修改作用范围：

``` sql
-- 查看最大值
show variables like "group_concat_max_len";
-- 修改全局设置（需要root权限，mysql重启失效，需要在配置文件修改）
SET GLOBAL group_concat_max_len = 10240;
-- 修改此次访问设置（是否可用于程序每次启动时初始化？）
SET session group_concat_max_len = 10240;
```

2. 最终还是要修改配置文件来修改最大值

> 在MySQL配置文件添加配置：`group_concat_max_len=10240`，不能设置为-1（亲测5.7不支持），默认值为：`1024`，最小值可为：`4`，64位系统最大值可为：`18446744073709551615`，32位系统最大值可为：`4294967295`。

##  批量复制更新表字段

``` sql
-- 复制另一个表的字段
UPDATE ce_person_staff ps, cm_person p 
SET ps.dingding_id = p.code
WHERE ps.id = p.id
and ps.dingding_id is null;
-- 复制同一个表的字段
UPDATE ce_person_staff 
SET dingding_id = id
WHERE dingding_id is null;
```





``` sql
select j.id, j.code, j.name,j.org_id, o.name as org_id, dict_val(j.job_type_id) as job_type_id
from cm_job j
left join cm_org o on o.id = j.org_id and o.is_valid = 1
where j.is_valid = 1
and j.spec_id = 1007
and find_in_set(j.org_id, CONCAT_WS(',', org_children(1002), org_children(1005)))
```



``` sql
SELECT p.id, p.CODE, p.NAME, job.NAME AS job_name 
FROM cm_person p
LEFT JOIN (
	SELECT r.person_id, r.NAME 
	FROM (
		SELECT a.person_id, a.NAME, @rownum := @rownum + 1,
		IF (@pdept = a.person_id, @rank := @rank + 1, @rank := 1 ) AS rank, @pdept := a.person_id 
		FROM (
			SELECT j.id AS job_id, j.NAME, pj.person_id 
			FROM cm_job j, cr_person_job pj 
			WHERE j.id = pj.job_id AND j.spec_id = 1007 
			ORDER BY pj.person_id ASC, pj.start_date DESC 
			) a,
			(SELECT @rownum := 0, @pdept := NULL,@rank := 0 ) b 
		) r 
	WHERE r.rank = 1 
	) job ON p.id = job.person_id 
WHERE p.is_valid = 1 
	AND p.id = 1009
```



``` sql
INSERT INTO cm_org_history (org_id,code,name,creator_id,create_date,modifier_id,modify_date,is_valid,notes,version,tenant_id,spec_id,short_name,type,unique_id,magor_id,level,parent_id,is_own_lower,update_type_id,all_name)
SELECT id,code,name,creator_id,create_date,modifier_id,modify_date,is_valid,notes,version,tenant_id,spec_id,short_name,type,unique_id,magor_id,level,parent_id,is_own_lower,2248,all_name from cm_org where is_valid = 1
```

