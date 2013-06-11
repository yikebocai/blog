def和defn的区别

def只评估一次，而defn每次执行都会评估，我的理解是def相当于java中的field声明，而defn相当于java中的method定义,看下面的例子： 
```clojure
user=> (def t1 (rand 10))
#'user/t1
user=> t1
7.032128762937022
user=> t1
7.032128762937022
user=> t1
7.032128762937022

user=> (defn t2 [] (rand 10))
#'user/t2
user=> (t2)
5.699600921636614
user=> (t2)
5.041377242233345
user=> (t2)
7.382964166904083
``` 

但是如果def后面跟上了fn匿名函数，那么结果和defn是一样的 
```clojure
user=> (def t3 (fn [] (rand 10)))
#'user/t3 
user=> (t3)
0.8802383384771573
user=> (t3)
9.385945393402316
user=> (t3)
6.777121776820955
user=> (t3)
3.9441466825577565
```