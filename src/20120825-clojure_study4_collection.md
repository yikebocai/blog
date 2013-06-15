Clojure学习笔记之四：集合与数据结构

Tag:Clojure

> *原文写于 2012-8-25*

**1.不同集合的表示形式** 
```clojure
'(a b :name 12.5) 
;;list
['a 'b :name 12.5] ;;vector
{:name "Bob" :age:32} ;;map
#{1 2 3}  ;;set
{Math/PI "~3.14"
  [:composite "key"] 42
  nil "nothing"}  ;;another map
#{{:first-name "bob" :last-name "zhang"}
   {:first-name "tom" :last-name "wang"}} ;;a set of map
```

**2.对集合的操作,可以看到集合是不可变的** 
```clojure
user=> (def v [1 2 3])
#'user/v
user=> (conj v 4)
[1 2 3 4]
user=> (conj v 4 5)
[1 2 3 4 5]
user=> (seq v)
(1 2 3)
user=> (def m {:a 5 :b 6})
#'user/m
user=> (conj m [:c 7])
{:a 5, :c 7, :b 6}
user=> (conj m {:c 7})
{:a 5, :c 7, :b 6}
user=> (seq m)
([:a 5] [:b 6])
user=> (def s #{1 2 3})
#'user/s
user=> (conj s 4)
#{1 2 3 4}
user=> (conj s 4 3)
#{1 2 3 4}
user=> (seq s)
(1 2 3)
user=> (def lst '(1 2 3))
#'user/lst
user=> (conj lst 2 4)
(4 2 1 2 3)
user=> (seq lst)
(1 2 3)
user=> (seq (conj lst 2 4))
(4 2 1 2 3) 

;;使用into代替conj和seq,into操作符是建立在这个之上
user=> (seq (conj lst 2 4))
(4 2 1 2 3)
user=> (into v [4 5])
[1 2 3 4 5]
user=> (into m [[:c 7] [:d 8]])
{:a 5, :c 7, :b 6, :d 8}
user=> (into #{1 2} [2 3 4])
#{1 2 3 4}
user=> (into [1] {:a 1 :b 2})
[1 [:a 1] [:b 2]]
```

**3.clojure共7种数据结构** 
Collection Sequence Associative Indexed Stack Set Sorted 

**4.Collection** 
所有的集合都有如下几种操作: conj:把一个或多个元素添加到一个集合中 seq:获取一个集合的序列 count:获取集合中元素的数量 empty:定义一个同类型的空集合实例 =:比较两个集合中的值是否相同 

**5.Sequence** 
```clojure
;;创建一个序列
user=> (cons 1 (range 1 5))
(1 1 2 3 4)
user=> (cons :a [:b :c])
(:a :b :c)
;;cons和list*等同
user=> (list* :a [:b :c])
(:a :b :c)
```

**6.Associative** 
assoc:把一个key/value和一个集合合并 dissoc:从集合中移除 get:取指定key的value值 contains:判断集合中是否包含某个key/value 
```clojure
user=> (def m {:a 1,:b 2, :c 3})
#'user/m
user=> (get m :b)
2
user=> (get m :d)
nil
user=> (get m :d "not found")
"not found"
user=> (assoc m :d 4)
{:a 1, :c 3, :b 2, :d 4}
user=> (dissoc m :b)
{:a 1, :c 3}
user=> (contains? m :b)
true
```

**7.Indexed** 
```clojure
user=> (nth [:a :b :c] 2)
:c
user=> (nth [:a :b :c] 3)
IndexOutOfBoundsException   clojure.lang.PersistentVector.arrayFor (PersistentVector.java:106)
user=> (get [:a :b :c] 2)
:c
user=> (get [:a :b :c] 3)
nil
```

**8.Stack** 
conj:push进去一个值 pop：移除最顶端的一个值 peek：取最顶端的一个值 
```clojure
user=> (conj '() 1)
(1)
user=> (conj '(2) 1)
(1 2)
user=> (peek '(2 3 1))
2
user=> (pop '(2 3 1))
(3 1)
```

**9.Set** 
```clojure
user=> (get #{1 3 4} 2)
nil
user=> (get #{1 3 4} 3)
3
;;移除指定的值
user=> (disj #{1 2 3} 3 1)
#{2}
```

**10.Sorted** 
```clojure
user=> (def sm (sorted-map :z 5  :y 0 :b 2 :a 3 :c 4))
#'user/sm
user=> sm
{:a 3, :b 2, :c 4, :y 0, :z 5}
user=> (rseq sm)
([:z 5] [:y 0]  [:c 4] [:b 2] [:a 3])
user=> (subseq sm &lt;= :c)
([:a 3] [:b 2] [:c 4])
user=> (subseq sm &lt;= :c &gt;= :b)
([:c 4] [:x 9] [:y 0] [:z 5])
;;必须先写大于操作符再写小于操作符，否则会报错
user=> (subseq sm &gt;= :b &lt;= :c)
([:b 2] [:c 4])
user=> (rsubseq sm &gt;= :b &lt;= :c)
([:c 4] [:b 2])
;;相等返回0，左边大返回1，右边大返回-1
user=> (compare 2 2)
0
user=> (compare 2 3)
-1
user=> (compare 4 3)
1
user=> (compare "ab" "abc")
-1
user=> (compare "ab" "ac")
-1
user=> (compare "abc" "ac")
-1
user=> (compare "adbc" "ac")
1
```