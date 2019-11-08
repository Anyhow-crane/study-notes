```clojure
{
  :user {
    :repositories ^:replace [["wiseloong" {:url "https://dc.demo.wiseloong.com/repository/maven-public/"
                                           :update :always}]]
    :plugin-repositories ^:replace [["wiseloong" "https://dc.demo.wiseloong.com/repository/maven-public/"]]
    :deploy-repositories [["snapshots" {:url   "https://dc.demo.wiseloong.com/repository/maven-snapshots/"
                                        :creds :gpg}]
                          ["releases" {:url   "https://dc.demo.wiseloong.com/repository/maven-releases/"
                                       :creds :gpg}]
                          ["wisdragon" {:url   "https://mvn.wisdragon.com/repository/maven-snapshots/"
                                       :creds :gpg}]]
    ; :signing {:gpg-key "loong"}
    }
}
```

- `^:replace` 表示只使用此仓库
- `:update :always` 只对snapshot有效，如果想要对release也生效需要加上`:releases {:checksum :fail :update :always}`，注意没有`:snapshots {:checksum :fail :update :always}`这样的写法。
- 如果仓库忽略snapshot，则配置`:snapshots false`
