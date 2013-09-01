Clojure和Java性能比较

今天看到庄晓丹之前的文章[《几行代码解决淘宝面试题之Clojure版》](http://www.blogjava.net/killme2008/archive/2010/07/15/326171.html)，于是拿来和Java版[《淘宝面试题：如何充分利用多核CPU，计算很大的List中所有整数的和》](http://www.iteye.com/topic/711162)做了个简单的对比测试，执行结果有点让我诧异。

100万个数字，100个一组，java实现版本在我4核CPU MBP笔记本测试结果稳定在155ms左右：

```
分配给线程：pool-1-thread-100那一部分List的整数和为：	SubSum:10047010149
分配给线程：pool-1-thread-22那一部分List的整数和为：	SubSum:2149575021
…….
List中所有整数的和为:499999500000
spend times:157
```

而Clojure版的不太稳定，并且并发版的并不比非并发的性能好甚至更长，这是为什么？结果如下：

```
user=> (defn mysum [coll n]
  #_=>      (let [sub-colls   (partition n n [0] coll)
  #_=>            result-coll (map #(reduce + 0 %) sub-colls) ]
  #_=>          (reduce + 0 result-coll)))
#'user/mysum
user=> (time (mysum (range 1000000) 100))
"Elapsed time: 522.086 msecs"
499999500000
user=> (time (mysum (range 1000000) 100))
"Elapsed time: 516.005 msecs"
499999500000
user=> (time (mysum (range 1000000) 100))
"Elapsed time: 496.007 msecs"
499999500000
user=> (time (mysum (range 1000000) 100))
"Elapsed time: 498.684 msecs"
499999500000
user=> (time (mysum (range 1000000) 100))
"Elapsed time: 480.915 msecs"
499999500000
user=> (defn mysum2 [coll  n]
  #_=>     (let [sub-colls   (partition n n [0] coll)
  #_=>           result-coll (map #(future (reduce + 0 %)) sub-colls)]
  #_=>          (reduce #(+ %1 @%2) 0 result-coll)))
#'user/mysum2
user=> (time (mysum2 (range 1000000) 100))
"Elapsed time: 734.095 msecs"
499999500000
user=> (time (mysum2 (range 1000000) 100))
"Elapsed time: 731.283 msecs"
499999500000
user=> (time (mysum2 (range 1000000) 100))
"Elapsed time: 726.19 msecs"
499999500000
```

虽然Clojure版的代码比Java版的代码简洁了N多倍，但性能如果上不去，还是不太敢在某关键应用上使用，并且它号称的SMT技术怎么没有发挥作用呢？有人说分的块太多，线程间切换的开销太大，然后把它分块数减少再做测试。并发版本分成10个子任务反而性能更差，另外并发版本相对非并发版本一点优势也体现不出来，很奇怪

```
user=> (time (mysum2 (range 1000000) 10000))
"Elapsed time: 676.284 msecs"
499999500000
user=> (time (mysum2 (range 1000000) 10000))
"Elapsed time: 561.026 msecs"
499999500000
user=> (time (mysum2 (range 1000000) 10000))
"Elapsed time: 561.596 msecs"
499999500000
user=> (time (mysum2 (range 1000000) 100000))
"Elapsed time: 1136.716 msecs"
499999500000
user=> (time (mysum2 (range 1000000) 100000))
"Elapsed time: 1101.662 msecs"
499999500000
user=> (time (mysum (range 1000000) 100000))
"Elapsed time: 1077.683 msecs"
499999500000
user=> (time (mysum (range 1000000) 10000))
"Elapsed time: 565.98 msecs"
499999500000
user=> (time (mysum (range 1000000) 1000))
"Elapsed time: 516.26 msecs"
499999500000
```

庄晓丹提到了[reducers](http://clojure.com/blog/2012/05/08/reducers-a-library-and-model-for-collection-processing.html)，是Clojure一个并发库。于是用reducers库重写了一下，性能依然没见有提升：

```
user=> (use '[clojure.core.reducers :as r])
nil
user=> (defn mysum [coll n]
  #_=>      (let [sub-colls   (partition n n [0] coll)
  #_=>            result-coll (r/map #(r/fold + %) sub-colls) ]
  #_=>          (r/fold + result-coll)))
#'user/mysum
user=> (def v (vec (range 1000000)))
#'user/v
user=> (time (mysum v 100))
"Elapsed time: 912.119 msecs"
499999500000
user=> (time (mysum v 100))
"Elapsed time: 497.629 msecs"
499999500000
user=> (time (mysum v 100))
"Elapsed time: 533.187 msecs"
499999500000
user=> (time (mysum v 100))
"Elapsed time: 935.193 msecs"
499999500000
```

这让我十分困惑，为何并发版本的性能并没有明显提升呢？仔细看这篇文章中提到reducers对vector或map比较能发挥并发的优势，而partition之后是一个lazy seq，再用r/fold也和普通的reduce一样，那我把它改成这样总可以了吧：

```
user=> (defn mysum [coll n]
  #_=>      (let [sub-colls   (partition n n [0] coll)
  #_=>            result-coll (r/map #(r/fold + (vec %)) sub-colls) ]
  #_=>          (r/fold + result-coll)))
```

经过测试还是没有明显变化，原因应该是每次把lazy seq转成vector的开销也是满大的，想要避免使用lazy seq和vector转换，直接使用r/fold求和，对比普通的reduce大约有一倍的提升：

```
user=> (def v (vec (range 1000000)))
#'user/v
user=> (dotimes [x 10] (time (reduce + v)))
"Elapsed time: 325.293 msecs"
"Elapsed time: 34.115 msecs"
"Elapsed time: 33.641 msecs"
"Elapsed time: 32.655 msecs"
"Elapsed time: 32.954 msecs"
"Elapsed time: 33.673 msecs"
"Elapsed time: 33.086 msecs"
"Elapsed time: 31.827 msecs"
"Elapsed time: 33.159 msecs"
"Elapsed time: 33.266 msecs"
nil
user=> (dotimes [x 10] (time (r/fold + v)))
"Elapsed time: 27.395 msecs"
"Elapsed time: 15.037 msecs"
"Elapsed time: 13.962 msecs"
"Elapsed time: 13.986 msecs"
"Elapsed time: 12.147 msecs"
"Elapsed time: 13.367 msecs"
"Elapsed time: 12.71 msecs"
"Elapsed time: 13.499 msecs"
"Elapsed time: 11.584 msecs"
"Elapsed time: 13.325 msecs"
nil
```

另外，把那个用并发处理的案例，直接在Java中进行求和，性能是最好的，100万只需要3ms，1000万也只需要6ms，并发时多线程的上下文切换开销是相当大的，这个例子可能不太恰当，因为求和计算的开销比上下文切换的开销要小的多，如果测试一个比较耗时的操作可能更能体现并发的优势。
