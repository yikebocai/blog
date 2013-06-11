Clojure学习笔记之二：编程初探
> *原文写于 2012-8-12*

**1.和java的语法对比** 

![clojure_vs_java](https://f.cloud.github.com/assets/2130097/267007/6c4deb16-8e4f-11e2-8e73-69956f76f658.png)


**2.Numeric** 

![clojure_numeric1](https://f.cloud.github.com/assets/2130097/267008/7427364e-8e4f-11e2-84a8-201354eb6163.png)


**3.Collection** 
```clojure
;读取容器Vector中的数据
user=> (def v [42 "foo" 99.2 [5 12]])
#'user/v
user=> (first v)
42
user=> (second v)
"foo"
user=> (last v)
[5 12]
user=> (third v)
CompilerException java.lang.RuntimeException: Unable to resolve symbol: third in this context, compiling:(NO_SOURCE_PATH:128)
user=> (v 2)
99.2
user=> ((v 3) 0)
5

;读取Map中的数据
user=> (def m {:a 5 :b 6 
              :c [7 8 9] 
              :d {:e 10 :f 11} 
              "foo" 88 
              42 false})
#'user/m
;把map绑定到指定的表达式上
user=> (let [{a :a b :b} m] (+ a b))
11
user=> (let [{a :a b :b c :c} m] (+ a b (c 2)))
20
;取key为42的值，如果为真打印T，否则打印F
user=> (let[{f 42} m]  (if f "T" "F") )
"F"
;取嵌套map中的值
user=> (let [{{e :e} :d} m] (* 2 e))
20
user=> (def mv ["Bob" {:birthday (java.util.Date. 79 11 8)}])
#'user/mv
user=> mv
["Bob" {:birthday #inst "1979-12-07T16:00:00.000-00:00"}]
user=> (let [[name {bd :birthday}] mv]
   (str name " was born on " bd))
"Bob was born on Sat Dec 08 00:00:00 CST 1979"
```

**4.Comments** 有两种方式，注释掉一代用分号(;)，注释掉一个clojure的表式用#_，比如： 
```clojure
;注释掉其中的(* 2 2)
user=> (read-string "(+ 1 2 #_(* 2 2) 8)")
(+ 1 2 8)
```

**5.Namespace** 

默认的namespace是user，可以用\*ns\*来查看当前的namespace，也可自己指定使用哪个 
```clojure
user=> *ns*
#<Namespace user>
user=> (ns myNS)
nil
myNS=> *ns*
<Namespace myNS>
```

**6.do** 
```clojure
;do表示按次序执行clojure表达式
user=> (do (println "hi") (apply * [4 5 6]))
hi
120
```

**7.def** 
```clojure
user=> (def p "foo")
#'user/p
user=> p
"foo"
```

**8.let** 
```clojure
;定义一个本地变量，或者说绑定（* x x）到x2这个变量上
user=> (defn hypot
[x y]
(let [x2 (* x x)
      y2 (* y y )]
   (Math/sqrt (+ x2 y2))))
#'user/hypot
user=> (hypot 2 3)
3.605551275463989
user=> (+ (first v) (first (last v)))
47
```

**9.自定义函数fn** 
```clojure
;使用内嵌函数进行加法运算
user=> ((fn [x] (+ 10 x)) 5)
15
;使用内嵌表达式计算
user=> (let [x 8] (+ 10 x))
18
```

**10.loop & recur** 
```clojure
;循环递减打印每一个值，直到负数为止
user=> (loop [x 5] (if(neg? x) "STOP" (do (println x) (recur (dec x)))))
5
4
3
2
1
0
"STOP"
```

**11.和java进行交互** 

![clojure_java_interop](https://f.cloud.github.com/assets/2130097/267005/4aa7f9d4-8e4f-11e2-8811-6fa71aba59f5.png)
