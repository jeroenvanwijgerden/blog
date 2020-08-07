```clojure
(defn roll-fn [n]
  #(inc (rand-int n)))

(defn perform
  "Returns the result of performing roll fn `f`"
  [f]
  (f))

(float (with-precision 2 :rounding FLOOR (/ 1 (bigdec 3))))

(perform (roll-fn 6))

(defn histogram
  [f n]
  (frequencies (repeatedly n #(perform f))))

(histogram (roll-fn 6) 1e2)

(defn print-histogram
  [h]
  (let [result->frequency h
        n (reduce + (vals result->frequency))
        highest (apply max (vals result->frequency))]
    (println (str "#rolls: " n))
    (doseq [[r f rel-f] (->> result->frequency
                       (sort-by first)
                       (map (fn [[r f]] [r f (float (with-precision 3 :rounding FLOOR (/ f (bigdec n))))])))]
           (println (apply str r \tab \tab rel-f \tab \tab (repeat (* 20 (/ f highest)) \*))))))

(print-histogram (histogram (roll-fn 6) 1e2))

(def prh (fn [f n] (print-histogram (histogram f n))))

(prh (roll-fn 6) 1)

;; I could rewrite all the roll-fn stuff so it's all data instead of fns, perform would hide this.
;; then my printing stuff could be richer, we would still know how certain rolls were made.
;; however, could also do that by having each roll-fn return a map of details, including the result.

(defn with-advantage
  "Returns a roll fn that performs roll fn `f` twice and returns the highest result."
  [f]
  #(max (perform f) (perform f)))

(perform (with-advantage (roll-fn 6)))

(prh(with-advantage (roll-fn 6)) 1e3)

(defn with-disadvantage
  "Returns a roll fn that performs roll fn `f` twice and returns the lowest result."
  [f]
  #(min (perform f) (perform f)))

(prh (with-disadvantage (roll-fn 6)) 1e3)

(prh (with-disadvantage (with-advantage (roll-fn 20))) 1e6)
(prh (with-advantage (with-disadvantage (roll-fn 20))) 1e6)

(defn multiple
  "Returns a roll fn that performs roll fns `fs` and returns the sum of the results."
  [fs]
  #(reduce + (map perform fs)))

(prh (multiple (repeat 2 (roll-fn 6))) 1e6)

(prh (multiple (repeat 4 (with-advantage (roll-fn 20)))) 1e6)
(prh (with-advantage (multiple (repeat 4 (roll-fn 20)))) 1e6)
(prh (with-advantage (multiple (repeat 2 (with-advantage (roll-fn 6))))) 1e6)
```
```
(defmulti value ffirst)

(defmethod value :roll
  [[_ v :as p]]
  [p v])

(defmethod value :with-advantage
  [[_ a b :as p]]
  [p (max (second (value a)) (second (value b)))])

(defmulti perform first)

(defmethod perform :roll
  [[_ n :as r]]
  [r (inc (rand-int n))])

(defmethod perform :with-advantage
  [[_ f :as r]]
  [r (perform f) (perform f)])

(defn roll [n]
  [:roll n])

(defn with-advantage [f]
  [:with-advantage f])

(defn multiple [fs]
  [:multiple fs])

(value (perform (roll 6)))
(value [[:roll 6] 5])

(value (perform (with-advantage (with-advantage (roll 10)))))
```
```
(defn roll-fn [n]
  [[:roll n]
   (fn [] [[:roll n]
           (inc (rand-int n))])])

(defn with-advantage [rfn]
  [[:with-advantage rfn]
   (fn [] (let [r1 ((second rfn))
                r2 ((second rfn))]
            [[:with-advantage r1 r2]
             (max (second r1) (second r2))]))])

(defn multiple [rfns]
  [[:multiple rfns]
   (fn [] (let [rs (map (comp #(%) second) rfns)]
            [[:multiple rs]
             (reduce + (map second rs))]))])

((second (roll-fn 6)))
((second (with-advantage (roll-fn 6))))
((second (multiple (repeat 2 (with-advantage (roll-fn 6))))))
```
