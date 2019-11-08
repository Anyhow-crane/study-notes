# cprop-clojure

获取配置信息组件，例如数据库配置等，[Github](https://github.com/tolitius/cprop)

```clojure
(ns config
  (:require [cprop.core :refer [load-config]]
            [cprop.source :refer [from-system-props from-env]]
            [mount.core :refer [args defstate]]))

(defstate env :start (load-config :merge [(args) (from-system-props) (from-env)]))
```

> 上面示例为结合mount，一起使用。

## load-config

1. 自动读取resources根路径下的config.edn文件，

2. 也读取系统配置里的conf参数：有如下两种使用场景

启动命令使用示例

```sh
java -Dconf="../somepath/app.edn" -jar app.jar
```

> 此模式可结合docker用于生产部署，dockerfile打包镜像时的启动命令可以使用此模式，方便修改数据库地址。只要复制相应的数据库配置文件到docker容器，重新启动容器。这种情况为一个项目打包一个镜像。

lein使用示例，此示例为开发模式示例，生产模式类似放到:prod里。

```clojure
:profiles {:dev {:jvm-opts ["-Dconf=resources/app.edn"]}}
```

## from-system-props

读取系统配置信息，覆盖上面的配置信息。其中`_`表示子元素，`.`表示`-`，例如：

```sh
-Dhttp_pool_socket.timeout=4242
```

```clojure
{:http {:pool {:socket-timeout 4242}}}
```

注意当配置的值比较复杂时要加上双引号。

```sh
-Dpool.spec_jdbc.url="jdbc:mysql://ip:3306/db?useSSL=false&serverTimezone=PRC"
```

```clojure
{:pool-spec {:jdbc-urll "jdbc:mysql://ip:3306/db?useSSL=false&serverTimezone=PRC"}}
```

> 此模式也可结合docker用于生产部署，使用不同脚本打包镜像时，改为对应的配置。这种情况为一个项目打包多个不同场景的镜像。当然也可在docker里使用环境参数，启动容器时设置参数。

## from-env

读取环境变量，覆盖上面的配置信息，下面2种情况：

1. 读取export设置的变量，其中`__`表示子元素，`_`表示`-`，例如：

```sh
export AWS__ACCESS_KEY=AKIAIOSFODNN7EXAMPLE
```

```clojure
{:aws {:access-key "AKIAIOSFODNN7EXAMPLE"}}
```

> 最适合docker模式，可以在容器启动时添加环境变量来改变配置信息，例如修改数据库连接信息

- 直接docker运行时，添加-e参数设置环境变量，如下：

```
docker run -d -e POOL_SPEC__JDBC_URL="jdbc:mysql://192.168.16.17:3306/hrms?useSSL=false&serverTimezone=PRC" image
```

- 使用**docker-compose.yml**添加`environment`，如下：

```yml
version: '3'
services:
   hrms-s:
     image: hrms:s
     ports:
       - "3000:3000"
     restart: always
     environment:
       POOL_SPEC__JDBC_URL: jdbc:mysql://ip:3306/hrms?useSSL=false&serverTimezone=PRC
```

2. lein配置（没测试不知道行不行，官网没介绍）

```clojure
{:dev  {:env {:database-url "jdbc:postgresql://localhost/dev"}}
 :test {:env {:database-url "jdbc:postgresql://localhost/test"}}}
```

# migratus

牛逼的自动数据库内容升级，回滚。

```sh
java -Dconf=conf.edn -jar app.jar migrate
```
