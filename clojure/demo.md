``` clojure
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
```

