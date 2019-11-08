# "parent" project

 使用"parent" projects来控制统一子项目依赖组件的版本。

## parent

```clojure
(defproject wiseloong/root "0.1.0-SNAPSHOT"
  :managed-dependencies [[ring/ring-core "1.7.1"]])
```

## children

```clojure
(defproject wiseloong/devtool "1.0.0"
  :parent-project {:coords  [wiseloong/root "0.1.0-SNAPSHOT"]
                   :inherit [:managed-dependencies]}
  :plugins [[lein-parent "0.3.5"]]
  :dependencies [[ring "1.6.1"]
                 ;[ring/ring-core]
                 ])
```

- 子项目不能缺少`[lein-parent "0.3.5"]`这个插件，只能放在这里，放到~/.lein/profiles.clj里不行。

- 子项目里的依赖组件版本统一按父级的来，除非你强制给版本：比如上面的例子里，ring包含ring-core，子项目的ring-core版本是1.7.1，不是1.6.1。

- 官方lein给的配置是`:parent-project [:coords xxx :inherit xxx]`，是不对的应该是`{}`。
