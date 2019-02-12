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





``` sql
select j.id, j.code, j.name,j.org_id, o.name as org_id, dict_val(j.job_type_id) as job_type_id
from cm_job j
left join cm_org o on o.id = j.org_id and o.is_valid = 1
where j.is_valid = 1
and j.spec_id = 1007
and find_in_set(j.org_id, CONCAT_WS(',', org_children(1002), org_children(1005)))
```

