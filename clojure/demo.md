```clojure
(defn xxx [x & {:keys [a b] :as p}]
  (println x)
  (println a)
  (println b)
  (println p))
=> #'user/xxx
(xxx "d" :a 1 :b 2 :c 3)
d
1
2
{:c 3, :b 2, :a 1}
=> nil

(str "abc"
             \newline ;; 换行
             "Documentation:"
             \space ;; 空格
             "https://ant.design/components/")
```

```clojure
;判断是否是原子信息，satisfies?判断接口，instance?判断类型
(satisfies? IAtom options)
(instance? reagent.ratom/RAtom options) ;只能判断(r/atom x),不能判断 (r/cursor vdata [k])
```

```clojure

(doto (js/FormData.)
  (#(dotimes [i (count file)] (.append % "files" (get file i)))))
; = 
(doto (js/FormData.)
  (.append "files" (get file 1))
  (.append "files" (get file 2))...)
```

```clojure
(defn meta-list [obj-key d]
  (let [d (utils/page-size d)
        list-page (-> (str "personnel.db.metadata/" obj-key "-list-page") symbol resolve)
        m-count (-> (str "personnel.db.metadata/" obj-key "-count") symbol resolve)]
    (merge (m-count d)
           {:list (utils/map-time-string (list-page d))})))
```

可以用函数macroexpand看macro在被evaluated之前返回的data structure.

```clojure
(ns example.plot
   (:require [reagent.core :as reagent]))

 (defn- plot
   ([comp]
    (let [data [{:label "foo"
                 :data (-> comp reagent/props :data)}]  ; use props
          plot-options {:grid {:hoverable true
                               :clickable true}}]
      (.plot js/$ (reagent/dom-node comp) (clj->js data) 
                                          (clj->js plot-options)))))

 (defn plot-component []
   (reagent/create-class
     {:component-did-mount plot
      :component-did-update plot
      :display-name "plot-component"
      :reagent-render (fn []
                        [:div.plot-container 
                          {:style 
                            {:width "100%" :height "500px"}}])}))
```

```clojure
(defn temp-file-store
  ([]
   (let [k :temp-file
         now (atom (str (LocalDate/now)))]

     (println "f1-----------")
     (fn [item]
       (println "f2-----------" (str item))
       (let [temp-file (File/createTempFile "ring-multipart-" nil)]
         (io/copy (:stream item) temp-file)
         (-> (select-keys item [:filename :content-type])
             (assoc :tempfile temp-file
                    :size (.length temp-file)))
         )))))

(def redis-temp-file-store (delay (temp-file-store)))

:middleware [[wrap-multipart-params {:store @redis-temp-file-store}]]
```

```clojure
(try (println @f)
  (catch js/Error e)
  (catch :default e) ;; 这个必须放在最后一个抓去
  (finally ))
```
