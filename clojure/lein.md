

使用atom，需要在它变化时，变化 需要在外层加(fn []) 

``` clojure
(defn yyy []
    (fn []
        (let [abc (:handler @!location)]
            [:div [abc]])))
```

不能是

``` clojure
(defn yyy []
    (let [abc (:handler @!location)]
        (fn []
            [:div [abc]])))
```

``` clojure
 (:import [goog.net XhrIo EventType WebSocket]
           [goog.net.xpc CfgFields CrossPageChannel]
           [goog Uri])
(.getParameterValue
                        (Uri. (.-href (.-location js/window)))
                        "xpc")
```

