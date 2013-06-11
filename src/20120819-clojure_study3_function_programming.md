Clojure学习笔记之三：函数编程
> *原文写于 2012-8-19*

1.<a href="http://en.wikipedia.org/wiki/Functional_programming" target="_blank">函数编程(Functional Pragramming)</a>的定义是指计算机的运算都像数学上的函数运算一样，没有状态，值是不可变的；函数能作为另一个函数的参数传入，一个函数的返回结果也可以是一个函数。比如著名函数编程语言有<a href="http://common-lisp.net/" target="_blank">Common Lisp</a>，<a href="http://clojure.org" target="_blank">Clojure</a>，<a href="http://www.erlang.org/" target="_blank">Erlang</a>，<a href="http://www.scala-lang.org/" target="_blank">Scala</a>，<a href="http://www.haskell.org/haskellwiki/Haskell" target="_blank">Haskell</a>等 

2.把一个函数作为参数传进去，可以传入任意多个参数，调用任意多个函数 
```clojure
user=> (defn call-twice [f x y] (f x y) (f x y))
#'user/call-twice
user=> (call-twice println 123 456)
123 456
123 456
``` 

3.map,接收一个函数，后面跟上若干个集合，并使用传入的函数来对集合中的值做运算，返回计算后的集合序列 

```clojure
user=> (map clojure.string/upper-case ["a" "b" "c"])
("A" "B" "C")
user=> (map * [1 2 3] [5 6 7])
(5 12 21)
``` 

4.reduce,用指定的某个函数来计算集合中的值 

```clojure
user=> (reduce max [0 -3 10 48])
48
user=> (reduce min [0 -3 10 48])
-3
;;还可以指定初始值
user=> (reduce + 50  [0 -3 10 48])
105
;使用内置函数
user=> (reduce
   (fn [m v]
   (assoc m v (* v v))) ;assoc相当于java中的put,表示把参数v和v的平方作为key/value放到map m中
{} ;初始化一个空的map
[1 2 3 4])
{4 16, 3 9, 2 4, 1 1}
``` 

5.apply 

```clojure
user=> (def args [2 -2 10])
#'user/args
user=> (apply * 0.5 3 args)
-60.0

;;和reduce的区别是，apply可以在集合前加任意多个单个元素，reduce仅仅对集合生效 
user=> (apply *  1 3 [4 5])
60
user=> (reduce *  1 3 [4 5])
ArityException Wrong number of args (4) passed to: core$reduce  clojure.lang.AFn.throwArity (AFn.java:437)
``` 

6.parital,过滤 

```clojure
user=> (def s (partial filter string?))
#'user/s
user=> (s ["a" 1 2 "c"])
("a" "c")
user=> (def s (partial filter number?))
#'user/s
user=> (s ["a" 1 2 "c"])
(1 2)
``` 

7.composition function 

```clojure
;;注意参数前面的&符号
user=> (defn s [& nums] (str (- (apply + nums)))) 
#'user/s
user=> (s 10 20 3.2)
"-33.2"
 ;把&符号去掉看看
user=> (defn s [nums] (str (- (apply + nums))))
#'user/s
;再直接这样写就会出错
user=> (s 10 20 3.2) 
ArityException Wrong number of args (3) passed to: user$s  clojure.lang.AFn.throwArity (AFn.java:437)
;需要把一个集合传进去
user=> (s [10 20 3.2]) 
"-33.2"
;使用合成函数,写法会简洁很多
user=> (def s (comp str - +)) 
#'user/s
user=> (s 10 20 3.22)
"-33.22"
;也可以使用自函数
user=> (defn f [x] (Math/abs x))
#'user/f
user=> (def s (comp f - +))
#'user/s
user=> (s 10 20)
30
```