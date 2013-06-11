【算法】使用Clojure进行快速排序

快速排序是数据结构中的经典算法，原理简单就是随便在列表中取一个值，不过一般是取中间值，然后把比它小的放左边，比它大的放右边，然后递归处理左右两边的部分。最常见的实现是使用递归，代码简单但有一个致命问题是如果列表很大递归的深度很深可能出现堆栈溢出，替代的方案是自己使用堆栈来记录每部分的中间结果，之前用java实现过，但因为java的Collections类里提供了现成的解决方案，很少人自己来实现了，不过原理了解一下还是有必要的。 

该算法中主要使用到如下几个函数： 
**count**：计算列表的长度，相当于java的List.size() 
**if**：条件表达式，语法格式为(if condition true-expression false-expression) 
**nth**:取列表中的第N个元素 
**concat**：和Mysql中的功能类似，将多个元素或集合合并成一个 

递归的实现方案如下： 
```clojure
(defn quick-sort [mylist]
  ;(println "mylist:" mylist)

  ;计算列表的长度
  (def cnt (count mylist))
  ;计算中间位置
  (def mid (if (> cnt 1) (int (/ cnt  2)) 0))
  ;(println (str "mid:" mid))

  ;如果中间位置不大于0直接返回列表
  (if (> mid 0)
      (let [mid-value (nth mylist mid)  ;取中间位置的值
            left (filter #(< % mid-value) mylist) ;获取比中间值小的集合
            right (filter #(> % mid-value) mylist)]  ;获取比中间值大的集合
      ;(println "left:" left )
      ;(println "mid-value:" mid-value)
      ;(println "right:" right)
      ;递归
      (concat (quick-sort left) [mid-value] (quick-sort right ) )
      )

      mylist
  )
)

(println "sorted:"(quick-sort  [2 5 9 10 4 22 1 8]) )
```

输出结果如下： 
```
sorted: (1 2 4 5 8 9 10 22)
```