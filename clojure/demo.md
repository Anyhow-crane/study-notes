```clojure
user=> {:a 1, #_#_ :b 2, :c 3}
{:a 1, :c 3}


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

下载

```clojure
[ring.util.http-response :refer :all]
[ring.util.codec :as codec]

(GET "/exp/:object" []
  :path-params [object :- s/Str]
  (-> (io/resource "tpl-xlsx/test.txt")
      io/file    ;; 或者用 io/input-stream
      ok
      (content-type "application/vnd.ms-excel;charset=utf-8")
      (header "content-disposition"
        (str "attachment;filename=" (codec/url-encode "中文") ".txt"))))
;; (content-type "application/octet-stream") 表示不知道什么类型文件
```

```clojure
;; 行转列
(def abc [{:id 1 :name "a" :x1 "a1" :x2 1 }
          {:id 1 :name "a" :x1 "a2" :x2 2 }
          {:id 1 :name "a" :x1 "a3" :x2 3 }
          {:id 2 :name "b" :x1 "a1" :x2 4 }
          {:id 2 :name "b" :x1 "a2" :x2 5 }])

(defn abcd []
  (clojure.walk/walk
    (fn [{:keys [x1 x2] :as a}]
      (assoc a (keyword x1) x2))
    (fn [coll]
      (reduce
        (fn [a {:keys [id] :as b}]
          (if (some #(= id (:id %)) a)
            (map #(if (= id (:id %)) (merge % b) %) a)
            (conj a b)))
        [] coll))
    abc))
;; ({:id 2, :name b, :x1 a2, :x2 5, :a1 4, :a2 5} {:id 1, :name a, :x1 a3, :x2 3, :a1 1, :a2 2, :a3 3})
```

```clojure
[garden "1.3.9"]
(ns loongyun.core
  (:require
   [reagent.core :as r]
   [goog.style]
   [garden.core :refer [css]]))

(goog.style/installStyles (css [:body {:background "#ff0000"}]))
```

```clojure
(def my-variadic-set #(set %&))

(my-variadic-set 1 2 2)
;; => #{1 2}

(let [{name :name :or {name "Anonymous"}} {:language "ClojureScript"}]
  name)
;; => "Anonymous"
(let [{:strs [name surname]} {"name" "Cirilla" "surname" "Fiona"}]
  [name surname])
;; => ["Cirilla" "Fiona"]
(let [{:syms [name surname]} {'name "Cirilla" 'surname "Fiona"}]
  [name surname])
;; => ["Cirilla" "Fiona"]
(let [{:keys [::name ::surname]} {::name "Cirilla" ::surname "Fiona"}]
  [name surname])
;; => ["Cirilla" "Fiona"]
(let [{[fst snd] :languages} {:languages ["ClojureScript" "Clojure"]}]
  [snd fst])
;; => ["Clojure" "ClojureScript"]
```

```clojure
(ns yourapp.core
  (:require [goog.dom :as dom]))

(def element (dom/getElement "body"))
```

```
cljsjs/moment {:mvn/version "2.24.0-0"}
```

Then, open the repl and try the following:

using the`js/`special namespace

```
(require '[cljsjs.moment]) ;; just require, no alias

(.format (js/moment) "LLLL")
;; => "Monday, July 15, 2019 5:32 PM"
```

using the alias

```
(require '[moment :as m])

(.format (m) "LLLL")
;; => "Monday, July 15, 2019 5:33 PM"
```
