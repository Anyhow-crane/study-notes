# 前后端分离-project实例

## 后端完整文件

```clojure
(defproject xxx "0.1.0-SNAPSHOT"

  :dependencies [[org.clojure/clojure "1.9.0"]
                 [com.wiseloong.utils/soil "0.2.6-SNAPSHOT"]
                 [metosin/compojure-api "2.0.0-alpha25"]
                 [buddy "2.0.0"]
                 [ring-cors "0.1.12"]
                 [dk.ative/docjure "1.12.0"]
                 [mysql/mysql-connector-java "8.0.12"]]

  :plugins [[lein-ring "0.12.4"]
            [migratus-lein "0.6.0"]]

  :ring {:handler personnel.handler/app
         :init    personnel.handler/init
         :destroy personnel.handler/destroy
         :port    3000}

  :migratus {:store :database
             :db    "jdbc:mysql://localhost:3306/test?allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=PRC&user=test&password=test"}

  :profiles {:uberjar [:user :prod]
             :dev     {:dependencies   [[javax.servlet/javax.servlet-api "3.1.0"]
                                        [ring/ring-mock "0.3.2"]
                                        [ring/ring-jetty-adapter "1.7.1"]
                                        [ring/ring-devel "1.7.1"]]
                       :source-paths   ["env/dev/clj"]
                       :resource-paths ["env/dev/resources"]}
             :prod    {:uberjar-name   "xxx.jar"
                       :resource-paths ["env/prod/resources"]}})
```
