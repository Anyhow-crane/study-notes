### 领域模型

1. DO（Data Object）与数据库表结构一一对应，通过 DAO 层向上传输数据源对象。
2. DTO（Data Transfer Object）是远程调用对象，它是 RPC 服务提供的领域模型。
3. BO（Business Object），它是业务逻辑层封装业务逻辑的对象，一般情况下，它是聚合了多个数据源的复合对象。
4. VO（View Object） 通常是请求处理层传输的对象，它通过 Spring 框架的转换后，往往是一个 JSON 对象。

### RESTful API

1. 【GET】          /users                 # 查询用户信息列表
2. 【GET】          /users/1001            # 查看某个用户信息
3. 【POST】         /users                 # 新建用户信息
4. 【PUT】          /users/1001            # 更新用户信息(全部字段)
5. 【PATCH】        /users/1001            # 更新用户信息(部分字段)
6. 【DELETE】       /users/1001            # 删除用户信息
