# swagger

[metosin/compojure-api "2.0.0-alpha25"] 这个依赖放到:profiles :dev :dependencies里，使用swagger时打开页面报错，找不到swagger.json；放到:dependencies里就正常了。

所以当只是测试api时，不要集成swagger了。

```clojure
(def app
  (api
    (context "/abc" []
      :tags ["abc"]
      (GET "/" []
        :summary "get"
        (ok (let [a 123]
              {:id "asd"})))
      (POST "/bcd" []
        :summary "post"
        (ok {:id "asd"})))))
```

